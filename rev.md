## Excess Chars

Using ghidra on the binary

![image](https://user-images.githubusercontent.com/64488123/167382960-81af125e-c456-41cd-9e12-49b400dc0428.png)

We see a printFlag function which is neither called in main neither in getInput, which means its essentially a pwn.

Since it's a basic stack overflow I wont explain much on it. To be very brief, simply put AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAa's get program counter(eip in this case) to get 0x41414141 and then try to guess the offset of the eip, and then inject your return address(address of printFlag function) in your input and boom you have flag

![image](https://user-images.githubusercontent.com/64488123/167383475-dc627ff7-4a50-47f3-8341-e20c3d4931b0.png)

## Ten Little Phrases

Using ghidra on the binary

![image](https://user-images.githubusercontent.com/64488123/167383823-8d15fa66-f2b5-4f73-b15a-4e4aa43568f3.png)

Same case as above, the only differrence is that it takes 10 lines and then does stuff, and picks characters with randomly from the strings, using some brute force we need not know the exact location, as we can simplt replace the matching filler byte with the byte we need to consturct the return address

![image](https://user-images.githubusercontent.com/64488123/167384898-f2f43e67-8fad-4568-8e71-8fc5fa6e63a2.png)


**NOTE: In both these cases, everything except \xXX is all garbage and can be anything**
