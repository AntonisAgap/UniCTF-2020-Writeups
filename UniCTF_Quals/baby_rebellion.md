# Baby Rebellion  HTB x Uni CTF 2020 - Quals 

The creator provided us with three certificates .crt 
- andromeda.crt
- corius.crt
- mechi.crt

and a file called 
- challenge 

Let's open a certificate to see what's inside it:
```bash
 $ openssl x509 -in andromeda.crt -text -noout
```
```
 Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            f2:45:8e:91:cd:3d:df:65
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, ST = Nevada, L = Unknown, O = Cyborg, emailAddress = andromeda@cyb.org
        Validity
            Not Before: Aug 17 18:48:05 2020 GMT
            Not After : Aug 17 18:48:05 2021 GMT
        Subject: C = US, ST = Nevada, L = Unknown, O = Cyborg, emailAddress = andromeda@cyb.org
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:ae:7a:71:65:28:0f:3b:af:b7:bd:0c:cf:96:8f:
                    4a:ec:3c:bb:e3:26:6a:16:78:28:25:b5:15:e5:f6:
                    41:ff:ba:fc:a3:96:96:91:7e:c8:69:bd:5e:02:39:
                    01:95:5c:dc:22:b7:f8:f7:01:70:13:dd:41:7d:24:
                    18:95:1f:37:96:47:6f:bd:96:78:af:69:6f:25:ec:
                    3d:eb:6f:6b:45:bd:76:99:f6:06:40:ff:9b:9d:6d:
                    47:4a:64:90:21:91:7c:53:d1:12:b8:d0:c7:03:95:
                    51:1f:c7:30:72:34:60:50:4e:85:6e:cf:1c:9b:a3:
                    71:4f:ff:f8:fe:96:5c:75:4a:d7:2d:a0:b0:71:04:
                    33:59:0c:df:09:7d:2c:19:86:d1:fa:e4:21:2d:c1:
                    31:27:f1:90:48:9d:2b:15:26:dc:60:c9:78:88:ee:
                    01:c9:19:05:a4:49:b7:ff:22:df:6b:b8:0c:c6:35:
                    71:22:a3:90:3b:d8:27:09:84:00:ff:75:d6:20:b2:
                    9e:ea:f3:8b:23:68:d8:2a:7e:e9:62:e7:a9:cc:f0:
                    3d:35:2b:2a:e3:c7:9b:3d:56:49:ed:4d:1f:c0:6f:
                    3a:a1:32:fa:45:8f:bd:4c:0d:14:06:54:97:51:9b:
                    f9:bd:f3:05:d1:7b:b6:53:da:b5:75:fa:ce:48:83:
                    01:15
                Exponent: 3 (0x3)
    Signature Algorithm: sha256WithRSAEncryption
         91:40:73:6d:2d:14:da:a1:1c:88:a4:2d:e5:d7:ef:54:8b:4b:
         4e:2f:63:62:1d:21:7d:29:05:16:bc:56:bd:75:44:2b:96:f0:
         84:eb:fa:7b:16:e5:4a:7f:44:78:b1:c6:55:8c:a2:c5:66:12:
         96:d6:50:8c:ef:ea:b3:98:1d:c6:21:ce:fd:f4:7b:b7:2f:6d:
         41:04:bc:01:b8:6e:00:34:61:a9:84:67:83:96:72:79:ea:86:
         30:6b:a2:61:b7:86:f8:cf:52:fa:78:9b:86:3b:ab:5c:0e:27:
         d8:f3:4c:a3:27:3b:a1:fd:9b:36:f7:17:49:53:20:d1:89:c9:
         19:5a:3a:3b:be:7c:74:c0:b1:52:27:33:3e:f0:f4:ad:fe:82:
         4f:f1:f9:12:21:3c:c1:01:82:ce:f0:a0:c7:4e:0c:ae:15:f1:
         45:4b:8b:ac:38:24:48:1a:c1:8f:de:ca:1f:21:ed:ad:0f:7a:
         4f:e2:a1:31:28:9f:38:a9:82:b8:8b:de:90:39:6c:32:5e:e9:
         3e:4a:18:e2:98:8e:d6:bc:9c:1b:1c:52:91:c7:05:26:21:41:
         95:8f:2c:2e:9e:43:6e:3f:c9:57:2d:a7:8a:32:74:2a:e1:79:
         76:ee:bf:4a:47:54:79:72:80:09:cd:a1:4f:d8:d2:8a:1a:46:
         8b:d1:1a:24
```
Interesting... It seems that a message was encrypted with RSA using a 2048 bit key. It's a big key so we can't factorize the modulus,but the exponent is the lowest value, e=3.

If we check the other two certificates we can see that their message was encrypted also with a 2048 bit RSA and an exponent of 3. Let's keep that in mind.

Ok, let's check the challenge file.

```bash
$ cat challenge
```
```
To: "Andromeda" <andromeda@cyb.org>, "Mechi" <mechi@cyb.org>, "Corius" <corius@cyb.org>
From: "Dr Greez" <dr.greez@cyb.org>
Subject: Request for supplies
MIME-Version: 1.0
Content-Type: application/pkcs7-mime; smime-type=enveloped-data; name=smime.p7m
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename=smime.p7m

MIIFkQYJKoZIhvcNAQcDoIIFgjCCBX4CAQAxggSXMIIBggIBADBsMF8xCzAJBgNV
BAYTAlVTMQ8wDQYDVQQIDAZOZXZhZGExEDAOBgNVBAcMB1Vua25vd24xDzANBgNV
BAoMBkN5Ym9yZzEcMBoGCSqGSIb3DQEJARYNbWVjaGlAY3liLm9yZwIJAKDnC94/ ...
```
Hmmm, the file is s/mime file of a pkcs#7 enveloped data object. To read that we need to extract the pkcs7 object and parse it.

```bash
$ openssl smime -in challenge -pk7out -out challenge.pm7
$ openssl asn1parse -in challenge.pm7
```
```
    0:d=0  hl=4 l=1425 cons: SEQUENCE
    4:d=1  hl=2 l=   9 prim: OBJECT            :pkcs7-envelopedData
   15:d=1  hl=4 l=1410 cons: cont [ 0 ]
   19:d=2  hl=4 l=1406 cons: SEQUENCE
   23:d=3  hl=2 l=   1 prim: INTEGER           :00
   26:d=3  hl=4 l=1175 cons: SET
   30:d=4  hl=4 l= 386 cons: SEQUENCE
   34:d=5  hl=2 l=   1 prim: INTEGER           :00
   37:d=5  hl=2 l= 108 cons: SEQUENCE
   39:d=6  hl=2 l=  95 cons: SEQUENCE
   41:d=7  hl=2 l=  11 cons: SET
   43:d=8  hl=2 l=   9 cons: SEQUENCE
   45:d=9  hl=2 l=   3 prim: OBJECT            :countryName
   50:d=9  hl=2 l=   2 prim: PRINTABLESTRING   :US
   54:d=7  hl=2 l=  15 cons: SET
   56:d=8  hl=2 l=  13 cons: SEQUENCE
   58:d=9  hl=2 l=   3 prim: OBJECT            :stateOrProvinceName
   63:d=9  hl=2 l=   6 prim: UTF8STRING        :Nevada
   71:d=7  hl=2 l=  16 cons: SET
   73:d=8  hl=2 l=  14 cons: SEQUENCE
   75:d=9  hl=2 l=   3 prim: OBJECT            :localityName
   80:d=9  hl=2 l=   7 prim: UTF8STRING        :Unknown
   89:d=7  hl=2 l=  15 cons: SET
   91:d=8  hl=2 l=  13 cons: SEQUENCE
   93:d=9  hl=2 l=   3 prim: OBJECT            :organizationName
   98:d=9  hl=2 l=   6 prim: UTF8STRING        :Cyborg
  106:d=7  hl=2 l=  28 cons: SET
  108:d=8  hl=2 l=  26 cons: SEQUENCE
  110:d=9  hl=2 l=   9 prim: OBJECT            :emailAddress
  121:d=9  hl=2 l=  13 prim: IA5STRING         :mechi@cyb.org
  136:d=6  hl=2 l=   9 prim: INTEGER           :A0E70BDE3FCA6562
  147:d=5  hl=2 l=  11 cons: SEQUENCE
  149:d=6  hl=2 l=   9 prim: OBJECT            :rsaEncryption
  160:d=5  hl=4 l= 256 prim: OCTET STRING      [HEX DUMP]:B58E82BEA0E7A56624A98D12ABE2B6DC36C677A82D29AE3BA41A3EA60D71E3012EBB3B77C934E7DDEF1B773EAD7F6BB3151DF788CA456D896BCCA38B650F94A5AC1753A36C388A10DF5E8E3827D695B9F84AE512A5CEE43C78AF16353F4E0D90B9E2FC4ABD200D590F6CE531D1D7CFB2F774CAEBE442CA2E28A42C54C3E9383B6CDBC5C17AF2534AEC3921331AEC6EE929E63BB08CB1056039A8F5F6E30A751B0EC3DC6667A953508BAFADABEA06C41DD50B5056FAAF49AA5CDDB83A7E3ADA096C68D20AD76F1C29998745697371E3F715CD33BFEE04EFAED936148F9C70B600D2F80DF459EB67858509A6CC42EF22469D30C9C629EFA827CC3CDF0BEAEACF6A
  420:d=4  hl=4 l= 387 cons: SEQUENCE
  424:d=5  hl=2 l=   1 prim: INTEGER           :00
  427:d=5  hl=2 l= 109 cons: SEQUENCE
  429:d=6  hl=2 l=  96 cons: SEQUENCE
  431:d=7  hl=2 l=  11 cons: SET
  433:d=8  hl=2 l=   9 cons: SEQUENCE
  435:d=9  hl=2 l=   3 prim: OBJECT            :countryName
  440:d=9  hl=2 l=   2 prim: PRINTABLESTRING   :US
  444:d=7  hl=2 l=  15 cons: SET
  446:d=8  hl=2 l=  13 cons: SEQUENCE
  448:d=9  hl=2 l=   3 prim: OBJECT            :stateOrProvinceName
  453:d=9  hl=2 l=   6 prim: UTF8STRING        :Nevada
  461:d=7  hl=2 l=  16 cons: SET
  463:d=8  hl=2 l=  14 cons: SEQUENCE
  465:d=9  hl=2 l=   3 prim: OBJECT            :localityName
  470:d=9  hl=2 l=   7 prim: UTF8STRING        :Unknown
  479:d=7  hl=2 l=  15 cons: SET
  481:d=8  hl=2 l=  13 cons: SEQUENCE
  483:d=9  hl=2 l=   3 prim: OBJECT            :organizationName
  488:d=9  hl=2 l=   6 prim: UTF8STRING        :Cyborg
  496:d=7  hl=2 l=  29 cons: SET
  498:d=8  hl=2 l=  27 cons: SEQUENCE
  500:d=9  hl=2 l=   9 prim: OBJECT            :emailAddress
  511:d=9  hl=2 l=  14 prim: IA5STRING         :corius@cyb.org
  527:d=6  hl=2 l=   9 prim: INTEGER           :A9ED2806CD914835
  538:d=5  hl=2 l=  11 cons: SEQUENCE
  540:d=6  hl=2 l=   9 prim: OBJECT            :rsaEncryption
  551:d=5  hl=4 l= 256 prim: OCTET STRING      [HEX DUMP]:1A10597041DE0A661F19BE335E38A9E7FA4433FA5FD1ECA32F05CF87D7CCAC0EB071FEF38078680E37AF7008261E785252210476F40D47787BFD6E487E5D9F12BC7AC099F8E7F1BFF59CBDBAA89BE9DF66989B3A84031B0AA3CA0FEB51D209DC37E9EC14B54759D0754870DBF06CACE73218F3EFD53D5837091EC43D5F082ECA770D26BA1CEA02E84539FEC0F44CE68D22BDDA5FDE8275E582ED5CC045E53EDDF640C52CC5715CD603EF32B5E4440038ED9FFF9A666FC4FB7C4CF44970A1D71D8A755BD16B43AA6D784B351EDD0BA1ABAC440941F7D6AD3CADF2DED02FC3B62A017A69F115720818AE4BB8FC958F0655E125C76694D9D8B1878A97A6C4C3C158
  811:d=4  hl=4 l= 390 cons: SEQUENCE
  815:d=5  hl=2 l=   1 prim: INTEGER           :00
  818:d=5  hl=2 l= 112 cons: SEQUENCE
  820:d=6  hl=2 l=  99 cons: SEQUENCE
  822:d=7  hl=2 l=  11 cons: SET
  824:d=8  hl=2 l=   9 cons: SEQUENCE
  826:d=9  hl=2 l=   3 prim: OBJECT            :countryName
  831:d=9  hl=2 l=   2 prim: PRINTABLESTRING   :US
  835:d=7  hl=2 l=  15 cons: SET
  837:d=8  hl=2 l=  13 cons: SEQUENCE
  839:d=9  hl=2 l=   3 prim: OBJECT            :stateOrProvinceName
  844:d=9  hl=2 l=   6 prim: UTF8STRING        :Nevada
  852:d=7  hl=2 l=  16 cons: SET
  854:d=8  hl=2 l=  14 cons: SEQUENCE
  856:d=9  hl=2 l=   3 prim: OBJECT            :localityName
  861:d=9  hl=2 l=   7 prim: UTF8STRING        :Unknown
  870:d=7  hl=2 l=  15 cons: SET
  872:d=8  hl=2 l=  13 cons: SEQUENCE
  874:d=9  hl=2 l=   3 prim: OBJECT            :organizationName
  879:d=9  hl=2 l=   6 prim: UTF8STRING        :Cyborg
  887:d=7  hl=2 l=  32 cons: SET
  889:d=8  hl=2 l=  30 cons: SEQUENCE
  891:d=9  hl=2 l=   9 prim: OBJECT            :emailAddress
  902:d=9  hl=2 l=  17 prim: IA5STRING         :andromeda@cyb.org
  921:d=6  hl=2 l=   9 prim: INTEGER           :F2458E91CD3DDF65
  932:d=5  hl=2 l=  11 cons: SEQUENCE
  934:d=6  hl=2 l=   9 prim: OBJECT            :rsaEncryption
  945:d=5  hl=4 l= 256 prim: OCTET STRING      [HEX DUMP]:5F8B4C626D3D8812DF42775016FB5D8FA2F8549A0371CF79EC3F57363A232645F25EB9EA300E93063F21835E1265F49938C0BD2388C8EF49A4F56B25535F38165FC19D40B7E592B1E311629273AF1A0E931AA150017813D8D12FF38DD563EBF78482837C74E3DCE46767C165E14967D9E3A95E00286976DCDC95896E878BF08BB8A8A76742EEE03184CB5B71A5CF97043E7BFB601AFD300927027C9264644A6AEB295DB892F5D2E4DB3873C3A088C3E75AD195AA458CE926494B411F3C366265270698BB8C90375D8114B80B36F97F35272BB573827E83B6148A49A920AA772130E0BAEE29044E0BD7D891A3111A6ACB9446F882659132F655AF1DEF09496EA9
 1205:d=3  hl=3 l= 221 cons: SEQUENCE
 1208:d=4  hl=2 l=   9 prim: OBJECT            :pkcs7-data
 1219:d=4  hl=2 l=  29 cons: SEQUENCE
 1221:d=5  hl=2 l=   9 prim: OBJECT            :aes-256-cbc
 1232:d=5  hl=2 l=  16 prim: OCTET STRING      [HEX DUMP]:5F78552BA3D2F1568309953F73694305
 1250:d=4  hl=3 l= 176 prim: cont [ 0 ]
 ```

From the output we can see that the same message was sent to the three recipients of 256 length. From the last hex dump we can see that the data sent after this dump is encrypted by AES-256 in CBC mode with an IV of
```5F78552BA3D2F1568309953F73694305```

## RSA basics
The public key in the RSA system is a tuple of integers *(N,e)*, where *N* is the product of two primes *p* and *q*. 

The secret key is given by an integer *d* satisfying <a href="https://www.codecogs.com/eqnedit.php?latex=ed\equiv&space;1(mod(p-1)(q-1))" target="_blank"><img src="https://latex.codecogs.com/gif.latex?ed\equiv&space;1(mod(p-1)(q-1))" title="ed\equiv 1(mod(p-1)(q-1))" /></a>

Encryption of a message *M* produces the ciphertext
<a href="https://www.codecogs.com/eqnedit.php?latex=C\equiv&space;M^e(modN)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C\equiv&space;M^e(modN)" title="C\equiv M^e(modN)" /></a>

,which can be decrypted using *d* by computing <a href="https://www.codecogs.com/eqnedit.php?latex=C^d\equiv&space;M(modN)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?C^d\equiv&space;M(modN)" title="C^d\equiv M(modN)" /></a>

The secret key may be given by <a href="https://www.codecogs.com/eqnedit.php?latex=d_{p}\equiv&space;d(mod\,&space;p-1)\:&space;and\:&space;d_{q}\equiv&space;d(mod\,&space;q-1)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?d_{p}\equiv&space;d(mod\,&space;p-1)\:&space;and\:&space;d_{q}\equiv&space;d(mod\,&space;q-1)" title="d_{p}\equiv d(mod\, p-1)\: and\: d_{q}\equiv d(mod\, q-1)" /></a>
 if the Chinese Remainder Theorem is used to improve the speed of decryption.

 ## Chinese Remainder Theorem
 The Chinese Remainder Theorem tells us that when given pairwise coprime positive integers 
 <a href="https://www.codecogs.com/eqnedit.php?latex=n_{1},n_{2},...,n_{k}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?n_{1},n_{2},...,n_{k}" title="n_{1},n_{2},...,n_{k}" /></a> and arbitary integers <a href="https://www.codecogs.com/eqnedit.php?latex=a_{1},a_{2},...,a_{k}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?a_{1},a_{2},...,a_{k}" title="a_{1},a_{2},...,a_{k}" /></a> the system of simultaneous congruences
 
<a href="https://www.codecogs.com/eqnedit.php?latex=x\equiv&space;a_{1}(mod\,&space;n_{1})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?x\equiv&space;a_{1}(mod\,&space;n_{1})" title="x\equiv a_{1}(mod\, n_{1})" /></a>

<a href="https://www.codecogs.com/eqnedit.php?latex=x\equiv&space;a_{2}(mod\,&space;n_{2})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?x\equiv&space;a_{2}(mod\,&space;n_{2})" title="x\equiv a_{2}(mod\, n_{2})" /></a>

.
.
.

<a href="https://www.codecogs.com/eqnedit.php?latex=x\equiv&space;a_{k}(mod\,&space;n_{k})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?x\equiv&space;a_{k}(mod\,&space;n_{k})" title="x\equiv a_{k}(mod\, n_{k})" /></a>

has a solution,and the solution is unique modulo <a href="https://www.codecogs.com/eqnedit.php?latex=N=n_{1}n_{2}...n_{k}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?N=n_{1}n_{2}...n_{k}" title="N=n_{1}n_{2}...n_{k}" /></a>

So to find a solution to a system of congruences using the Chinese Remainder Theorem we should:

1. Compute <a href="https://www.codecogs.com/eqnedit.php?latex=N=n_{1}n_{2}...n_{k}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?N=n_{1}n_{2}...n_{k}" title="N=n_{1}n_{2}...n_{k}" /></a>
2. For each <a href="https://www.codecogs.com/eqnedit.php?latex=i&space;=&space;1,2,...,k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?i&space;=&space;1,2,...,k" title="i = 1,2,...,k" /></a> compute 

    <a href="https://www.codecogs.com/eqnedit.php?latex=y_{i}&space;=&space;\frac{N}{n_{i}}=n_{1}n_{2}...n_{i-1}n_{i&plus;1}...n_{k}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?y_{i}&space;=&space;\frac{N}{n_{i}}=n_{1}n_{2}...n_{i-1}n_{i&plus;1}...n_{k}" title="y_{i} = \frac{N}{n_{i}}=n_{1}n_{2}...n_{i-1}n_{i+1}...n_{k}" /></a>
3. For each  <a href="https://www.codecogs.com/eqnedit.php?latex=i&space;=&space;1,2,...,k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?i&space;=&space;1,2,...,k" title="i = 1,2,...,k" /></a>  compute <a href="https://www.codecogs.com/eqnedit.php?latex=z_{i}\equiv&space;y_{i}^{-1}mod\,&space;n_{i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?z_{i}\equiv&space;y_{i}^{-1}mod\,&space;n_{i}" title="z_{i}\equiv y_{i}^{-1}mod\, n_{i}" /></a> using Euclid's extended algorithm.
4. The integer <a href="https://www.codecogs.com/eqnedit.php?latex=x=\sum_{i=1}^{k}a_{i}y_{i}z{i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?x=\sum_{i=1}^{k}a_{i}y_{i}z{i}" title="x=\sum_{i=1}^{k}a_{i}y_{i}z{i}" /></a>  is a solution to the system of congruences, and <a href="https://www.codecogs.com/eqnedit.php?latex=x\,&space;mod\,&space;N" target="_blank"><img src="https://latex.codecogs.com/gif.latex?x\,&space;mod\,&space;N" title="x\, mod\, N" /></a> is the unique solution modulo *N*.

 
## Attacking RSA using Hastad's Broadcast Attack

Now that we know <a href="https://www.codecogs.com/eqnedit.php?latex=e=3" target="_blank"><img src="https://latex.codecogs.com/gif.latex?e=3" title="e=3" /></a> and thus <a href="https://www.codecogs.com/eqnedit.php?latex=M&space;=&space;m^{3}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?M&space;=&space;m^{3}" title="M = m^{3}" /></a>, we will have to solve this system of equations to find *M* which is the plaintext message:

<a href="https://www.codecogs.com/eqnedit.php?latex=\left\{\begin{matrix}&space;M\equiv&space;c_{1}[n_{1}]\\&space;M\equiv&space;c_{2}[n_{2}]\\&space;M\equiv&space;c_{3}[n_{3}]&space;\end{matrix}\right." target="_blank"><img src="https://latex.codecogs.com/gif.latex?\left\{\begin{matrix}&space;M\equiv&space;c_{1}[n_{1}]\\&space;M\equiv&space;c_{2}[n_{2}]\\&space;M\equiv&space;c_{3}[n_{3}]&space;\end{matrix}\right." title="\left\{\begin{matrix} M\equiv c_{1}[n_{1}]\\ M\equiv c_{2}[n_{2}]\\ M\equiv c_{3}[n_{3}] \end{matrix}\right." /></a>

Using the Chinese Remainder Theorem we showed before we can construct and find the solution to this system. Let's start coding...

## Implementing the attack
Firstly we need to extract from the files the ciphertext of each message and the modulo it was encrypted with.

### *Extracting the modulo*
To extract the modulo of each certificate we use this simple openssl command:
```
openssl x509 -in andromeda.crt -modulus
```
### *Extracting the ciphertext*
To extract the ciphertext sent to each recipient we use the hex dumps from the pkcs7 object before.

### Code
Let's use the ciphetexts and moduli to code the attack and find the plaintext
```python
import gmpy2

e = 3

n1 = "B5FC37C1BE11A17DDFF95873938DDEC917F340792C00AA18E78D90D861718D449216CA710C2BF54513789F11250FEB9D1912DCC3A79F85F0B2E30DA44713C1C728B76890D236E504D33D97DD2B7DD0A962F2E3475293CA511943A6D0953FD5FDFCA7DF5C8F68217AA7B2172AE695186E8FF3AE7DFA9B1D00227B5D79740EA79AA5ADF9BB7A06992BCDE4728C6B553F2A0C8593418535952B3889304AD1588B08EEB839A84E70547BF14B6C3CC3F9B43AE0B13562BA525DFEB0567F7EF932C68035AB724A5D448E446B04C270748746BFA741F5D9E8B2F901C9171E267D564C318FD758A4F9C43BB797FC087F81C616B2065562EBEAD45A544DA04B0F6303DA37"
n2 = "CFFC46EC62D10A6342BEC1A120FE445682053786E32E0E687FC30C7BFFA450D19CDD81E16B53DF25B638CA165189A73A12AC6056596F0AC793EF6C624E8FEFE6D01329E81F0EC735EE95C99AE92F2F4FF91F702BF3D3933C7D7D5247D96EDA3FE9694CD2B1C0E030AAB4FDD4881CC042CD0FC9EC6E5891EB9BFAAAD33549DDC300181A645506EA546E5BB9A1DB9DB48360BC30AE14989525C27C02390808EAB58331756041635314F42BC302FB6B84ED846E69D71008474B9D876CF355A6CB30BCB897D28523300357205BBD3B22B6ED9070C0F7C3241C226580ECE542817461913A775FAF687AD1ED7A24141357BBF4920B146F8806BECDF91C77BE05B0671F"
n3 = "AE7A7165280F3BAFB7BD0CCF968F4AEC3CBBE3266A16782825B515E5F641FFBAFCA39696917EC869BD5E023901955CDC22B7F8F7017013DD417D2418951F3796476FBD9678AF696F25EC3DEB6F6B45BD7699F60640FF9B9D6D474A649021917C53D112B8D0C70395511FC730723460504E856ECF1C9BA3714FFFF8FE965C754AD72DA0B0710433590CDF097D2C1986D1FAE4212DC13127F190489D2B1526DC60C97888EE01C91905A449B7FF22DF6BB80CC6357122A3903BD827098400FF75D620B29EEAF38B2368D82A7EE962E7A9CCF03D352B2AE3C79B3D5649ED4D1FC06F3AA132FA458FBD4C0D14065497519BF9BDF305D17BB653DAB575FACE48830115"

c1 = "B58E82BEA0E7A56624A98D12ABE2B6DC36C677A82D29AE3BA41A3EA60D71E3012EBB3B77C934E7DDEF1B773EAD7F6BB3151DF788CA456D896BCCA38B650F94A5AC1753A36C388A10DF5E8E3827D695B9F84AE512A5CEE43C78AF16353F4E0D90B9E2FC4ABD200D590F6CE531D1D7CFB2F774CAEBE442CA2E28A42C54C3E9383B6CDBC5C17AF2534AEC3921331AEC6EE929E63BB08CB1056039A8F5F6E30A751B0EC3DC6667A953508BAFADABEA06C41DD50B5056FAAF49AA5CDDB83A7E3ADA096C68D20AD76F1C29998745697371E3F715CD33BFEE04EFAED936148F9C70B600D2F80DF459EB67858509A6CC42EF22469D30C9C629EFA827CC3CDF0BEAEACF6A"
c2 = "1A10597041DE0A661F19BE335E38A9E7FA4433FA5FD1ECA32F05CF87D7CCAC0EB071FEF38078680E37AF7008261E785252210476F40D47787BFD6E487E5D9F12BC7AC099F8E7F1BFF59CBDBAA89BE9DF66989B3A84031B0AA3CA0FEB51D209DC37E9EC14B54759D0754870DBF06CACE73218F3EFD53D5837091EC43D5F082ECA770D26BA1CEA02E84539FEC0F44CE68D22BDDA5FDE8275E582ED5CC045E53EDDF640C52CC5715CD603EF32B5E4440038ED9FFF9A666FC4FB7C4CF44970A1D71D8A755BD16B43AA6D784B351EDD0BA1ABAC440941F7D6AD3CADF2DED02FC3B62A017A69F115720818AE4BB8FC958F0655E125C76694D9D8B1878A97A6C4C3C158"
c3 = "5F8B4C626D3D8812DF42775016FB5D8FA2F8549A0371CF79EC3F57363A232645F25EB9EA300E93063F21835E1265F49938C0BD2388C8EF49A4F56B25535F38165FC19D40B7E592B1E311629273AF1A0E931AA150017813D8D12FF38DD563EBF78482837C74E3DCE46767C165E14967D9E3A95E00286976DCDC95896E878BF08BB8A8A76742EEE03184CB5B71A5CF97043E7BFB601AFD300927027C9264644A6AEB295DB892F5D2E4DB3873C3A088C3E75AD195AA458CE926494B411F3C366265270698BB8C90375D8114B80B36F97F35272BB573827E83B6148A49A920AA772130E0BAEE29044E0BD7D891A3111A6ACB9446F882659132F655AF1DEF09496EA9"

c1 = int(c1, 16)
c2 = int(c2, 16)
c3 = int(c3, 16)
n1 = int(n1, 16)
n2 = int(n2, 16)
n3 = int(n3, 16)

N = n1*n2*n3
N1 = N//n1
N2 = N//n2
N3 = N//n3

u1 = gmpy2.invert(N1, n1)
u2 = gmpy2.invert(N2, n2)
u3 = gmpy2.invert(N3, n3)

M = (c1*u1*N1 + c2*u2*N2 + c3*u3*N3) % N

m = gmpy2.iroot(M,e)[0]
print(hex(int(m)))
```
output:
```
0x1ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff008bc717f3a42012d5311aa8f8ffb8f736c7e9e80fccaeaf2f618be5e277534f99
```
Success! This should be the AES-256 key which was used, it's only formatted in encryption-block formatting:
https://tools.ietf.org/html/rfc2313#section-8.1

Let's see how it is formatted.

```
   A block type BT, a padding string PS, and the data D shall be
   formatted into an octet string EB, the encryption block.

              EB = 00 || BT || PS || 00 || D .           (1)

   The block type BT shall be a single octet indicating the structure of
   the encryption block. For this version of the document it shall have
   value 00, 01, or 02. For a private- key operation, the block type
   shall be 00 or 01. For a public-key operation, it shall be 02.

   The padding string PS shall consist of k-3-||D|| octets. For block
   type 00, the octets shall have value 00; for block type 01, they
   shall have value FF; and for block type 02, they shall be
   pseudorandomly generated and nonzero. This makes the length of the
   encryption block EB equal to k.
```
Hmm.. So the key is this hex string:
```
8bc717f3a42012d5311aa8f8ffb8f736c7e9e80fccaeaf2f618be5e277534f99
```
Perfect! We have the key and the IV, let's go decrypt the message.

As we mentioned we need to decrypt the data after the last hexdump
```
1205:d=3  hl=3 l= 221 cons: SEQUENCE
 1208:d=4  hl=2 l=   9 prim: OBJECT            :pkcs7-data
 1219:d=4  hl=2 l=  29 cons: SEQUENCE
 1221:d=5  hl=2 l=   9 prim: OBJECT            :aes-256-cbc
 1232:d=5  hl=2 l=  16 prim: OCTET STRING      [HEX DUMP]:5F78552BA3D2F1568309953F73694305
 1250:d=4  hl=3 l= 176 prim: cont [ 0 ]
 ```
 So *1250+hl=1250+3=1253*, so we need to get the hex bytes after the 1253 byte.

 Alright, to find the flag we must execute these commands:
 ```
 $ openssl smime -in challenge -pk7out > base64file
 ```
Remove the PKCS7 headers so we can decode the base64 string.
```
base64 -d base64file | xxd -s 1253 -p | xxd -r -p > encrypted
```
Now that we have the encrypted message in ASCII form let's decrypt it using the AES-256-CBC IV and key we found earlier.
```
openssl aes-256-cbc -d -iv 5F78552BA3D2F1568309953F73694305 -K 8bc717f3a42012d5311aa8f8ffb8f736c7e9e80fccaeaf2f618be5e277534f99 -in encrypted
```
output:
```
Hello everyone,
We are out of microchips. Me and my team need more supplies! Hurry up, everyone has to be microchipped! Deliver the package here:
HTB{37.220464, -115.835938}
```
Success! We found the flag!