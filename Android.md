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
