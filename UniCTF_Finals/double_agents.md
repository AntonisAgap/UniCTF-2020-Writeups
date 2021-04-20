# Double Agents  HTB x Uni CTF 2021 - Finals

# Introduction

***The creator provided us with a docker to connect a server and the server's source code: server.py.***

Let's check out the code of the *server.py* file:
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import socketserver
import signal
import time
import os

key = os.urandom(16)


def challenge(req):
	req.sendall(bytes('Welcome, agent! Request a document:\n', 'utf-8'))

	ct = req.recv(4096).decode().strip()
	ct = bytes.fromhex(ct)
	if len(ct) % 16 != 0:
		req.sendall(bytes('Invalid input.\n', 'utf-8'))
		return

	cipher = AES.new(key, AES.MODE_CBC, key)
	
	try:
		file = cipher.decrypt(ct)
		f = open(unpad(file, 16),'rb')
		dt = f.read()
		f.close()
		req.sendall(dt)
	except:
		req.sendall(bytes("File not found: " + file.hex()  + "\n", 'utf-8'))

class incoming(socketserver.BaseRequestHandler):
	def handle(self):
		signal.alarm(300)
		req = self.request
		challenge(req)

class ReusableTCPServer(socketserver.ForkingMixIn, socketserver.TCPServer):
	pass


socketserver.TCPServer.allow_reuse_address = True
server = ReusableTCPServer(("0.0.0.0", 23333), incoming)
server.serve_forever()
```
Let's what the *challenge()* function does:
1. Accepts a ciphertext.
2. Checks if the length of the ciphertext bytes is multiple to 16 bytes which is also the length of the key.
3. Initializes a cipher object, which in our case is an AES with a CBC mode of operation.
4. It deciphers the ciphertext and opens a file using the plaintext as a name.
5. If the file exists it sends us its data, if it doesnt exist it responds with the name of the file (plaintext)

According to the description of the challenge we must find the file: *double_agents.txt*. So what we have to do is send a ciphertext to the server, which decrypts to the plaintext: double_agents.py.

To be able to do that we must encrypt the plaintext *double_agents.txt* with the same key that was used in the server's cipher. Let's take a closer look at how the AES CBC is used.

```
cipher = AES.new(key, AES.MODE_CBC, key)
```
That's interesting... We can see that the IV (initialize vector) is the same as the key used to encrypt the plaintexts. We can exploit that choice to retrieve the key, so we can use it to find the correct ciphertext of *double_agents.txt*.

Let's see how and why we can use that.

# AES-CBC cipher

In the CBC mode of operation, each plaintext is XORed with the previous ciphertext block before being encypted. This way, each ciphertext block depends on all plaintext blocks processed up to that point. To make each message unique, an initialization vector must be used in the first block.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/8/80/CBC_encryption.svg/1920px-CBC_encryption.svg.png)
![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2a/CBC_decryption.svg/800px-CBC_decryption.svg.png)
If the first block has index 1, the mathematical formula for CBC encryption is
<a href="https://www.codecogs.com/eqnedit.php?latex=C_{i}&space;=&space;E_{k}(P_{i}&space;\oplus&space;C_{i-1})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C_{i}&space;=&space;E_{k}(P_{i}&space;\oplus&space;C_{i-1})" title="C_{i} = E_{k}(P_{i} \oplus C_{i-1})" /></a>,

<a href="https://www.codecogs.com/eqnedit.php?latex=C_{0}&space;=&space;IV" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C_{0}&space;=&space;IV" title="C_{0} = IV" /></a>

While the mathematical formula for CBC decryption is

<a href="https://www.codecogs.com/eqnedit.php?latex=P_{i}&space;=&space;D_{k}(C_{i})\oplus&space;C_{i-1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?P_{i}&space;=&space;D_{k}(C_{i})\oplus&space;C_{i-1}" title="P_{i} = D_{k}(C_{i})\oplus C_{i-1}" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=C_{0}&space;=&space;IV" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C_{0}&space;=&space;IV" title="C_{0} = IV" /></a>

But since the key <a href="https://www.codecogs.com/eqnedit.php?latex=K" target="_blank"><img src="https://latex.codecogs.com/gif.latex?K" title="K" /></a> is used as an <a href="https://www.codecogs.com/eqnedit.php?latex=IV" target="_blank"><img src="https://latex.codecogs.com/gif.latex?IV" title="IV" /></a> we actually have <a href="https://www.codecogs.com/eqnedit.php?latex=P_{1}&space;=&space;D_{K}(C_{1})\oplus&space;K" target="_blank"><img src="https://latex.codecogs.com/gif.latex?P_{1}&space;=&space;D_{K}(C_{1})\oplus&space;K" title="P_{1} = D_{K}(C_{1})\oplus K" /></a>

Now in order to find the key <a href="https://www.codecogs.com/eqnedit.php?latex=K" target="_blank"><img src="https://latex.codecogs.com/gif.latex?K" title="K" /></a> we need to get the value of <a href="https://www.codecogs.com/eqnedit.php?latex=D_{K}(C_{1})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{K}(C_{1})" title="D_{K}(C_{1})" /></a> . To find that value we can feed to the decryption oracle <a href="https://www.codecogs.com/eqnedit.php?latex=C_{i-1}&space;=&space;0" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C_{i-1}&space;=&space;0" title="C_{i-1} = 0" /></a> in which case we'll get <a href="https://www.codecogs.com/eqnedit.php?latex=P_{i}=D_{K}(C_{1})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?P_{i}=D_{K}(C_{1})" title="P_{i}=D_{K}(C_{1})" /></a> and then we can get the key by XORing <a href="https://www.codecogs.com/eqnedit.php?latex=P_{i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?P_{i}" title="P_{i}" /></a> with <a href="https://www.codecogs.com/eqnedit.php?latex=P_{1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?P_{1}" title="P_{1}" /></a>,

<a href="https://www.codecogs.com/eqnedit.php?latex=D_{K}(C_{1})\oplus&space;(D_{K}(C_{1})&space;\oplus&space;IV)&space;=&space;IV" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D_{K}(C_{1})\oplus&space;(D_{K}(C_{1})&space;\oplus&space;IV)&space;=&space;IV" title="D_{K}(C_{1})\oplus (D_{K}(C_{1}) \oplus IV) = IV" /></a>.

So the attack would be:
1. Pick a random <a href="https://www.codecogs.com/eqnedit.php?latex=C_{1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C_{1}" title="C_{1}" /></a>
2. Feed the decryption oracle with the ciphertext <a href="https://www.codecogs.com/eqnedit.php?latex=C_{1}\left&space;|&space;\right&space;|0\left&space;|&space;\right&space;|C_{3}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C_{1}\left&space;|&space;\right&space;|0\left&space;|&space;\right&space;|C_{3}" title="C_{1}\left | \right |0\left | \right |C_{3}" /></a> where <a href="https://www.codecogs.com/eqnedit.php?latex=C_{1}=C_{3}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C_{1}=C_{3}" title="C_{1}=C_{3}" /></a>
3. Compute <a href="https://www.codecogs.com/eqnedit.php?latex=P_{1}\oplus&space;P_{3}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?P_{1}\oplus&space;P_{3}" title="P_{1}\oplus P_{3}" /></a>

# Implementing the attack
```python
from pwn import *
from time import sleep
import socket
import os
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad,pad

HOST = "docker.hackthebox.eu"
PORT = 31502
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))
data = s.recv(1024)


# creating payload

c1 = b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA'
r = b'00000000000000000000000000000000'
payload = c1+r+c1


s.sendall(payload)
data = s.recv(1024)
data = data.decode("utf-8")
data = data.replace("File not found: ","").rstrip()

p1 = data[0:32]
p3 = data[64:96]

# xoring p1 and p3 to find the key

key = int(p1,16) ^ int(p3,16)
key = '{:x}'.format(key)
key = bytes.fromhex(key)

# finding ciphertext of flag.txt
cipher = AES.new(key, AES.MODE_CBC, key)
raw = b"double_agents.txt"
raw = pad(raw,16)
filec = cipher.encrypt(raw)


s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))
data = s.recv(1024)

# sending the ciphertext to the decryption oracle
ct = filec.hex().encode()
s.sendall(ct)
data = s.recv(1024)
print(repr(data))
```
*output*
```
b'Tracee Nick\nDestiny Ramsay\nHarlan Plantz\nHerta Lyles\nEarl Crunk\nScott Denker\nArgelia Virgil\nGale Boeck\nClare Jenning\nTamesha Light\nQuincy Emert\nVanna Blade\nHTB{1v_sh01d_b3_r4nd0m}\nRandell Grimsley\nDominga Hicklin\nElise Godby\nInga Knabe\nBoyd Hendriks\nHai Duhaime\nJoy Tarlton\nJanuary Knauf\n \n'
```
Nice! We got the flag!!
```
HTB{1v_sh01d_b3_r4nd0m}
```