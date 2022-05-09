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
