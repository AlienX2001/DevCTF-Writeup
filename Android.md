## PinBreak

We simply open the apk as archive, see a .db file, open it in sqlite

![image](https://user-images.githubusercontent.com/64488123/167407060-43eebdd2-e2a0-4eef-8d69-c77df0996af0.png)

Get the hash, crack it using https://crackstation.net and get our pin to unlock the app

## SoDifficult

We first decompile the apk using the following http://www.javadecompilers.com/apk and get the following in the source code

![image](https://user-images.githubusercontent.com/64488123/167407626-12fca9c7-0bb5-4aad-8896-29ed28079e44.png)

We see there is a shared library getting loaded and there is a checkPassword function which calls checkPassword3. Now decompiling the x86_64 version of the library using ida we see that there is a long list of if statement

![image](https://user-images.githubusercontent.com/64488123/167408033-883020ca-72d2-4aa1-8554-ef6e2851182b.png)

hence using the z3 python library we craft this code and reverse the conditions (converting from != to ==) and get the output as flag.

```python
from z3 import *
s=Solver()

v4 = IntVector('v4', 20)

s.add((v4[3] + v4[14] + v4[11] * v4[14])==11451)
s.add( 2 * v4[6] - v4[11] == 125)
s.add((v4[11] * v4[7] )== 6745)
s.add((v4[5] - v4[4] - v4[6] )== -168)
s.add((v4[18] + v4[14] * v4[10] * v4[9] )== 577829)
s.add(v4[10]== 102)
s.add((v4[17] + v4[5] + v4[18]) == 203)
s.add(( v4[0] + v4[4] * v4[17])== 5770)
s.add((v4[18] * v4[3] + v4[8] + v4[15] )== 12569)
s.add((v4[14] + v4[9] + v4[0] * v4[3]) == 12343)
s.add((v4[5] * v4[3] + v4[5] - v4[3] )== 5953)
s.add((v4[7] + v4[11] * v4[19] * v4[2] )== 1211321)
s.add((v4[7] + v4[18] - v4[5] )== 123)
s.add((v4[15] + v4[18] )== 152)
s.add((v4[0] * v4[14] * v4[3] )== 1436886)
s.add((v4[3] + v4[2] )== 225)
s.add((v4[0])== 99)
s.add((v4[12] - v4[10] * v4[3] )== -12464)
s.add((v4[0] * v4[14] + v4[14] + v4[9])== 11848)
s.add((v4[9] + v4[0] + 2 * v4[10])== 351)
s.add((v4[5] - v4[5] * v4[16] * v4[13] )== -284837)
s.add((v4[19] + v4[17] - v4[12] * v4[3] )== -9908)
s.add((v4[0] + v4[12] - v4[18])== 80)
s.add((v4[0] + v4[10] * v4[9] )== 4995)
s.add((v4[12] + v4[10] - v4[17])== 131)
s.add((v4[14] - v4[14] * v4[3] )== -14396)
s.add(v4[16] == 114)
s.add((v4[9] - (v4[5] + v4[18]) )== -102)
s.add((v4[2] )== 102)
s.add((v4[18] - v4[4] * v4[11] )== -10064)
s.add((v4[18] * v4[19] - v4[0] )== 12526)
s.add((v4[17] * v4[5] * v4[2] )== 264894)
s.add((v4[6] + v4[11] * v4[12] )== 7900)
s.add((v4[15] + v4[4] - v4[11] )== 63)
s.add((v4[5] + v4[10] - v4[1] )== 35)
s.add((v4[10] + v4[18] * v4[5] * v4[15] )== 252501)
s.add((v4[19] + v4[5] - v4[8] )== 79)
s.add((v4[2] + v4[16] + v4[4] * v4[8] )== 10381)
s.add((v4[16] * v4[13] + v4[4] + v4[1] )== 6037)

print(s.check())
print(s.model())


flag=""
arr=[]
for i in v4:
  arr.append(s.model()[i])
print(arr)
for j in arr:
    flag += chr(j)

print(flag)
```

## Network

We first decompile the apk using the following http://www.javadecompilers.com/apk and get the following in the source code

![image](https://user-images.githubusercontent.com/64488123/167440612-48c7d21d-55a0-44c1-bc3d-745ae30d46df.png)

We see it trying to connect to google.com/flag with some weird sha256 base64 stuff, and hence its failing, readiing a bit more about this certificate pinner stuff, seems it mitigates the risk of MiTM https://square.github.io/okhttp/4.x/okhttp/okhttp3/-certificate-pinner/. Now since its not continuing on normal use lets try go through the code flow. We see 2 strings from jni, the stringFromJNI and stringFromJNIw which is being taken from the loaded shared library so lets decompile it and have a look.
this is the stringFromJNIw function

![image](https://user-images.githubusercontent.com/64488123/167443509-5cace000-83e4-478c-a5ac-30bc1186b715.png)

This function essentially gives back "https:" and this makes sense in the screenshot we can see that there is a URL being formed
And the request body is being taken from stringFromJNI

![image](https://user-images.githubusercontent.com/64488123/167443715-a9b852b0-82fe-454a-a5f0-6dacc96ff7a7.png)

Now this function's reversing had multiple jumps to here and there and took a lot of time, after solving and making a python script for it, this is what it boils down to

```python
arr=[77,84,73,122,78,68,85,50,78,122,103,53,77,71,70,105,89,50,82,108,90,103]
from base64 import *
s=b""
for i in arr:
  s+=chr(i).encode()
key=(b64decode(s+b"=="))
print(key,len(key))


from Crypto.Cipher import AES
  
decipher = AES.new(key, AES.MODE_ECB)



var_56 ="4d6d6766"
var_52 ="6c375439"
var_48 ="62674573"
var_44 ="2b525930"
var_40 ="44"
var_39 ="4a785a59"
var_35 ="3d3d67"

cipher3=b""
cipher3+=bytes.fromhex(var_56)[::-1]
cipher3+=bytes.fromhex(var_52)[::-1]
cipher3+=bytes.fromhex(var_48)[::-1]
cipher3+=bytes.fromhex(var_44)[::-1]
cipher3+=bytes.fromhex(var_40)[::-1]
cipher3+=bytes.fromhex(var_39)[::-1]
cipher3+=bytes.fromhex(var_35)[::-1]
cipher3=(b64decode(cipher3))
# print(cipher3)
print("GET URL : ",decipher.decrypt(cipher3).decode("UTF-8"))

tmp_40="344f6f49"
tmp_36="2f502b71"
tmp_32="70434f32"
tmp_28="72574f48"

tmp_56="59394b77"
tmp_52="30426b48"
tmp_48="6c7a416d"
tmp_44="5973697a"

tmp_72="6b454657"
tmp_68="7434782b"
tmp_64="77364d61"
tmp_60="35684a56"

tmp_88="6e686d74"
tmp_84="7230326a"
tmp_80="504a4b31"
tmp_76="45717865"

cipher3=b""
cipher3+=bytes.fromhex(tmp_88)[::-1]
cipher3+=bytes.fromhex(tmp_84)[::-1]
cipher3+=bytes.fromhex(tmp_80)[::-1]
cipher3+=bytes.fromhex(tmp_76)[::-1]

cipher3+=bytes.fromhex(tmp_72)[::-1]
cipher3+=bytes.fromhex(tmp_68)[::-1]
cipher3+=bytes.fromhex(tmp_64)[::-1]
cipher3+=bytes.fromhex(tmp_60)[::-1]

cipher3+=bytes.fromhex(tmp_56)[::-1]
cipher3+=bytes.fromhex(tmp_52)[::-1]
cipher3+=bytes.fromhex(tmp_48)[::-1]
cipher3+=bytes.fromhex(tmp_44)[::-1]

cipher3+=bytes.fromhex(tmp_40)[::-1]
cipher3+=bytes.fromhex(tmp_36)[::-1]
cipher3+=bytes.fromhex(tmp_32)[::-1]
cipher3+=bytes.fromhex(tmp_28)[::-1]

cipher3=(b64decode(cipher3))
print(cipher3)
print("GET URL : ",decipher.decrypt(cipher3).decode("UTF-8"))
```
