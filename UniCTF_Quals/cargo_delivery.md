# Cargo Delivery  HTB x Uni CTF 2020 - Quals 

***The creator provided us with a docker to connect to and the code the server is running: 
server.py***

Lets check out the code
```python
def challenge(req):
	req.sendall(bytes('This crypto service is used for Chasa\'s delivery system!\n'
		'Not your average gangster.\n'
		'Options:\n'
		'1. Get encrypted message.\n'
		'2. Send your encrypted message.\n', 'utf-8'))
	try:
		choice = req.recv(4096).decode().strip()

		index = int(choice)

		if index == 1:
			req.sendall(bytes(encrypt(flag) + '\n','utf-8'))
		elif index == 2:
			req.sendall(bytes('Enter your  ciphertext:\n', 'utf-8'))
			ct = req.recv(4096).decode().strip()
			req.sendall(bytes(is_padding_ok(bytes.fromhex(ct)), 'utf-8'))
		else:
			req.sendall(bytes('Invalid option!\n', 'utf-8'))
			exit(1)
	except:
		exit(1)
```
We can see that we can communicate with the server in two ways. 
- Get the encrypted message, which is probably the flag, and 
- send an encrypted message

The first option returns us in bytes the encrypted message and the second option lets us send an encrypted message and all it does is to check its padding.

*Hmmm...*

Lets take a look at the encryption/decryption algorithm used.
```python
KEY_LENGTH = 16
BLOCK_SIZE = AES.block_size

key = os.urandom(KEY_LENGTH)

def add_padding(msg):
	pad_len = BLOCK_SIZE - (len(msg) % BLOCK_SIZE)
	padding = bytes([pad_len]) * pad_len
	return msg + padding

def remove_padding(data):
	pad_len = data[-1]
	if pad_len < 1 or pad_len > BLOCK_SIZE:
		return None
	for i in range(1, pad_len):
		if data[-i-1] != pad_len:
			return None
	return data[:-pad_len]

def encrypt(msg):
	iv = os.urandom(BLOCK_SIZE)
	cipher = AES.new(key, AES.MODE_CBC, iv)
	return (iv + cipher.encrypt(add_padding(msg))).hex()


def decrypt(data):
	iv = data[:BLOCK_SIZE]

	cipher = AES.new(key, AES.MODE_CBC, iv)
	return remove_padding(cipher.decrypt(data[BLOCK_SIZE:]))

def is_padding_ok(data):
	if decrypt(data) is not None:
		return 'This is a valid ciphertext!\n'
	else:
		return 'Invalid ciphertext\n'
```
We can see tat the encryption algorithm is a ***128bit AES on CBC mode***. That's interesting. What we can also see is that the server uses a function *is_padding_ok()* which checks if an encrypted message's padding is valid or not. That's very interesting. Because the server returns if the padding is valid or not instead of a *decryption failed* error, we (the attacker) can use an attack called padding oracle attack.

## Cipher block chaining (CBC)
![alt text](https://upload.wikimedia.org/wikipedia/commons/8/80/CBC_encryption.svg)
![alt text](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2a/CBC_decryption.svg/1920px-CBC_decryption.svg.png)

In CBC encryption mode of operation each block of plaintext is XORed with the previous ciphertext block before being encrypted. An initialization is used fot the first block.When I invert a single bit in the previous block,the matching bit in the decrypted cleartext will be inverted.

### Padding
Block cipher encrypts data message in *blocks* of a fixed size. For instance, AES treats 16-byte data as one block, and DES uses a block size of 8 bytes. If a message is shorter than a block, the algorithm needs to fill it up with extra characters, which is called as padding. If a message is longer than a block, it's divided into multiple blocks and is encrypted separately.f you have 15 bytes and need to add one more byte to fill up the block, you append hex (01). If you need to add 2 bytes, you append hex (02 02). 3 bytes requires you add the 3-byte pad of hex(03 03 03) etc. This allows a form of error checking, because there is some  redundancy when more that a single byte is added. If the last byte has the hex value of 04, then the previous 3 bytes have the same value. If not, then that is a padding error.

### Padding Oracle Attack
In order to perform a Padding Oracle Attack we need two things:

- A padding oracle

- A way to capture and replay an encrypted message to that orcale.

Luckily we can do both of these things.

The mathematical formula for CBC decryption is
<a href="https://www.codecogs.com/eqnedit.php?latex=P_{i}&space;=&space;D_{k}(C_{i})&space;\oplus&space;C_{i-1},&space;C_{0}=IV" target="_blank"><img src="https://latex.codecogs.com/gif.latex?P_{i}&space;=&space;D_{k}(C_{i})&space;\oplus&space;C_{i-1},&space;C_{0}=IV" title="P_{i} = D_{k}(C_{i}) \oplus C_{i-1}, C_{0}=IV" /></a>

Suppose the attacker has two ciphertext blocks *C1*, *C2* and they want to decrypt the second block to get plaintext *P2*. The attacker changes the last byte of *C1*  (creating *C1′*) and sends (*IV , C1′, C2*)} to the server. The server then returns whether or not the padding of the last decrypted block (*P2′*) is correct (equal to ***0x01***).If the padding is correct, the attacker now knows that the last byte of *DK (C2) ⊕ C1′* is ***0x01***.Therefore, *DK(C2)=C1′ ⊕0x01*. If the padding is incorrect, the attacker can change the last byte of *C1′* to the next possible value. At most, the attacker will need to make 256 attempts (one guess for every possible byte) to find the last byte of *P2*.If the decrypted block contains padding information or bytes used for padding then an additional attempt will need to be made to resolve this ambiguity.

After determining the last byte of *P2*, the attacker can use the same technique to obtain the second-to-last byte of *P2*. The attacker sets the last byte of *P2* to ***0x02*** by setting the last byte of *C1* to *DK(C2)⊕0x02*.The attacker then uses the same approach described above, this time modifying the second-to-last byte until the padding is correct ***(0x02, 0x02)***.

If a block consists of 128 bits (AES, for example), which is 16 bytes, the attacker will obtain plaintext *P2* in no more than 255⋅16 = 4080 attempts.

### Implemantation of Padding Oracle Attack

```python
def attack_message(msg):
    cipherfake=[0] * 16
    plaintext = [0] * 16
    current = 0
    message=""
    number_of_blocks = int(len(msg)/BLOCK_SIZE)
    blocks = [[]] * number_of_blocks
    for i in (range(number_of_blocks)):
        blocks[i] = msg[i * BLOCK_SIZE: (i + 1) * BLOCK_SIZE]

    for z in range(len(blocks)-1):
        for itera in range (1,17):
            for v in range(256):
                cipherfake[-itera]=v
                data = bytes(cipherfake)+blocks[z+1]
                data = data.hex()
                if is_padding_ok(data): 
                    current=itera
                    plaintext[-itera]= v^itera^blocks[z][-itera]
            for w in range(1,current+1):
                cipherfake[-w] = plaintext[-w]^itera+1^blocks[z][-w] 
        for i in range(16):
            if plaintext[i] >= 32: 
                char = chr(int(plaintext[i]))
                message += char

    return str.encode(message)
```

Function to check response:
```python
def is_padding_ok(msg):
    conn.recv()
    conn.send('2')
    conn.recv()
    conn.send(msg)
    answer = conn.recvline()
    if answer == b'This is a valid ciphertext!\n':
        return True
    elif answer == b'Invalid ciphertext\n':
        return False
```

Lets put that together:
```python
from pwn import *
from pwnlib.replacements import sleep

HOST = "docker.hackthebox.eu"
PORT = 30338
BLOCK_SIZE = 16
conn = remote(HOST,PORT)

def is_padding_ok(msg):
    conn.recv()
    conn.send('2')
    conn.recv()
    conn.send(msg)
    answer = conn.recvline()
    if answer == b'This is a valid ciphertext!\n':
        return True
    elif answer == b'Invalid ciphertext\n':
        return False

conn.send('1')
msg = conn.recvline().rstrip().decode("utf-8") 
msg = bytes.fromhex(msg)

def attack_message(msg):
    cipherfake=[0] * 16
    plaintext = [0] * 16
    current = 0
    message=""
    number_of_blocks = int(len(msg)/BLOCK_SIZE)
    blocks = [[]] * number_of_blocks
    for i in (range(number_of_blocks)):
        blocks[i] = msg[i * BLOCK_SIZE: (i + 1) * BLOCK_SIZE]

    for z in range(len(blocks)-1):
        for itera in range (1,17):
            for v in range(256):
                cipherfake[-itera]=v
                data = bytes(cipherfake)+blocks[z+1]
                data = data.hex()
                if is_padding_ok(data): 
                    current=itera
                    plaintext[-itera]= v^itera^blocks[z][-itera]
            for w in range(1,current+1):
                cipherfake[-w] = plaintext[-w]^itera+1^blocks[z][-w] 
        for i in range(16):
            if plaintext[i] >= 32: 
                char = chr(int(plaintext[i]))
                message += char

    return str.encode(message)


print(attack_message(msg=msg))
```
By running this attack we get the flag:
```
b'HTB{CBC_0r4cl3}'
```