## Listing

The code is as follows 
```C++
#include <windows.h>
#include <tchar.h> 
#include <stdio.h>
#include <strsafe.h>
#pragma comment(lib, "User32.lib")

void DisplayErrorBox(LPTSTR lpszFunction);

int _tmain(int argc, TCHAR *argv[])
{
   WIN32_FIND_DATA ffd;
   LARGE_INTEGER filesize;
   TCHAR szDir[MAX_PATH];
   size_t length_of_arg;
   HANDLE hFind = INVALID_HANDLE_VALUE;
   DWORD dwError=0;

   if(argc != 2)
   {
      _tprintf(TEXT("\nUsage: %s <directory name>\n"), argv[0]);
      return (-1);
   }

   StringCchLength(argv[1], MAX_PATH, &length_of_arg);

   if (length_of_arg > (MAX_PATH - 3))
   {
      _tprintf(TEXT("\nDirectory path is too long.\n"));
      return (-1);
   }

   printf(TEXT("\n%s\n\n"), argv[1]);


   StringCchCopy(szDir, MAX_PATH, argv[1]);
   StringCchCat(szDir, MAX_PATH, TEXT("\\*"));

   // Find the first file in the directory.

   hFind = FindFirstFile(szDir, &ffd);

   if (INVALID_HANDLE_VALUE == hFind) 
   {
      DisplayErrorBox(TEXT("FindFirstFile"));
      return dwError;
   } 
   
   // List all the files in the directory with some info about them.

   do
   {
      if (ffd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
      {
         _tprintf(TEXT("  %s   <DIR>\n"), ffd.cFileName);
      }
      else
      {
         filesize.LowPart = ffd.nFileSizeLow;
         filesize.HighPart = ffd.nFileSizeHigh;
	 if(filesize.QuadPart/(1024*1024*1024) > 1){
		_tprintf(TEXT("  %s   %ld gb\n"), ffd.cFileName, filesize.QuadPart/(1024*1024*1024));
	 }
	 else if(filesize.QuadPart/(1024*1024) > 1){
                _tprintf(TEXT("  %s   %ld mb\n"), ffd.cFileName, filesize.QuadPart/(1024*1024));
         }
	 else if(filesize.QuadPart/(1024) > 1){
                _tprintf(TEXT("  %s   %ld kb\n"), ffd.cFileName, filesize.QuadPart/(1024));
         }
	 else{
         	_tprintf(TEXT("  %s   %ld bytes\n"), ffd.cFileName, filesize.QuadPart);
	 }
      }
   }
   while (FindNextFile(hFind, &ffd) != 0);
 
   dwError = GetLastError();
   if (dwError != ERROR_NO_MORE_FILES) 
   {
      DisplayErrorBox(TEXT("FindFirstFile"));
   }

   FindClose(hFind);
   return dwError;
}


void DisplayErrorBox(LPTSTR lpszFunction) 
{ 
    // Retrieve the system error message for the last-error code

    LPVOID lpMsgBuf;
    LPVOID lpDisplayBuf;
    DWORD dw = GetLastError(); 

    FormatMessage(
        FORMAT_MESSAGE_ALLOCATE_BUFFER | 
        FORMAT_MESSAGE_FROM_SYSTEM |
        FORMAT_MESSAGE_IGNORE_INSERTS,
        NULL,
        dw,
        MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
        (LPTSTR) &lpMsgBuf,
        0, NULL );

    // Display the error message and clean up

    lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT, 
        (lstrlen((LPCTSTR)lpMsgBuf)+lstrlen((LPCTSTR)lpszFunction)+40)*sizeof(TCHAR)); 
    StringCchPrintf((LPTSTR)lpDisplayBuf, 
        LocalSize(lpDisplayBuf) / sizeof(TCHAR),
        TEXT("%s failed with error %d: %s"), 
        lpszFunction, dw, lpMsgBuf); 
    MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK); 

    LocalFree(lpMsgBuf);
    LocalFree(lpDisplayBuf);
}

```

And the working is as follows

![image](https://user-images.githubusercontent.com/64488123/167396569-b5093498-b5c7-47a4-8df3-23b13435aa35.png)

## Captcha The Flag

We get a URL https://web.ctf.devclub.in/dev/4/ and chall description is "Are you too fast to be human?" So from the looks of it we might need to script some python to automate whatever comes with the site.
In the site, it basically asks us to fill up the captcha
![image](https://user-images.githubusercontent.com/64488123/167369054-5f99a5b9-f32b-4d8d-9537-e6d471f763e7.png)
and in the source code we see the following
![image](https://user-images.githubusercontent.com/64488123/167369121-9ce99c48-b359-49be-8898-5c9c300ebb03.png)
so that means the image is essentially this base64 content, which we can verify if we convert this b64 to raw bytes and open the resulting file.

Now all that needs to be done is simply scripting some python which would visit the site, extract this b64, make a file out of it, read the contents and post that content into the submit input
Using selenium and pytesseract, this is as easy as the code given below
```python
from selenium import webdriver
import os
from PIL import Image
from selenium.webdriver.common.keys import Keys
import pytesseract

driver = webdriver.Firefox() # using firefox as our webdriver/browser

driver.get("https://web.ctf.devclub.in/dev/4/")

s = driver.find_elements_by_tag_name('img')
im = s[0].get_attribute('src').split(',')[-1] # getting the base64 string from the page

with open("/tmp/temp","w") as f: # storing the string on disk
    f.write(im)

os.system('base64 -d /tmp/temp > /tmp/temp_img.jpg') # base64 decode it and making it an image

image = '/tmp/temp_img.jpg'
val = pytesseract.image_to_string(Image.open(image), timeout=1.5) # extracting strings from the image

val = val.strip()
print(val)

# sending the extracted captcha to the login input form
a = driver.find_element_by_id('login')
a.send_keys(val)
a.send_keys(Keys.ENTER)

driver.close()
```
