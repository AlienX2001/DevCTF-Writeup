
Opening the malware (*No, I do not know why these fellas would give us an actual malware to work upon, why wont they make their own challs. Yes, it was a lot of fun to break apart this malware, got me into a bit of malware analysis, and had that real world aspect of it*) with ghidra.
Since this was a real malware hence it was heavily obfuscated hence first look for strings.

## payload

The first part asks us the resource where the payload is located hence in the strings we search for anything related to resrouce and find a windows API function FindResourceW which gets called here

![image](https://user-images.githubusercontent.com/64488123/167387445-46764635-b5c1-445e-8d9d-27c686faf7af.png)

Reading about this function in here https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-findresourcew we understand the resource has an lpString and in here we see that the lpString is "asdasdasdasdsad" hence this is the answer.

## Mutex

The 2nd part asks us the mutex used by the payload.
#### Now what are mutex's?

For our level of abstrsction its enough to know that mutex is essentially something may or may not be attached to a process. If a process has a mutex assosiated with it, then it will have only 1 instance of it running at a time. If while 1 instance is running another instance of this process tries to get started, it will match the mutex, and since another process with same mutex exists(meaning this process is already running) hence it wont start another instance of the same process.

Doing some goole fu we find that we can create a mutex for a process using CreateMutexA Windows API call, search for it in the binary we get the following place where it is getting called

![image](https://user-images.githubusercontent.com/64488123/167389016-4f9230e7-a3e2-4e34-85ad-2575379109e2.png)

Hence the mutex name here is "avcjkvcnmvcnmcvnmjdfkjfd"

## Which Address

This questions gives us an address and asks "Which WINAPI function is send to CreateRemoteThreadAPI as a 4th parameter at location 0x10002174?".
So lets go to that address

![image](https://user-images.githubusercontent.com/64488123/167389513-f4c8c070-23cd-4bbc-a94e-887365292c05.png)

We see that there is indeed a CreateRemoteThreadAPI taking 4 arguments now looking directly at assembly at this stage

![image](https://user-images.githubusercontent.com/64488123/167389754-119b1ae4-4f13-4ce3-9119-798e21f6368d.png)

We see that there is a call to GetProcAddress which Retrieves the address of an exported function or variable from the specified dynamic-link library (DLL) according to Microsoft in here https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress. 
We know after a call, the return values gets set at rax/eax/ax (depends on the datatype). Since its a dword, hence 4 bytes hence its eax. We see the value moves into "[EBP + local_184]" and after a while this value get into ecx and CreateRemoteThread is called. So now again looking at the psuedocode, we see that the GetProcAddress has a parameter "LoadLibraryA". Hence this is the Windows API function being sent.

## Privileged

In this question we are asked which Privilege token is the malware trying to gain.

#### About tokens and their privleges in windows

https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens

Essentially in windows identification of a user/process/etc. are done using different types of tokens, and security tokens are 1 such type of tokens, which define the security restrictions and access rights of a process.
Note: All these token are all in memory of the particular process we are interested in.

Hence 1 of the things of security tokens are that they start with the characters "Se" hence we search for this string pattern 

![image](https://user-images.githubusercontent.com/64488123/167391639-202a8e70-30e4-4db6-a053-12c47a67923c.png)

And we see the SeDebugPrivilege, which is the token we need here.
