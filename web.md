## The Good Old Days

The URL is https://web.ctf.devclub.in/web/5/ and the chall description is "U need a time machine".
After visiting the site it gives this

![image](https://user-images.githubusercontent.com/64488123/167370684-0f63df45-ae5d-40b5-80f5-c4005feae534.png)

We see that it recognizes our windows version, and with my linux VM it recognized my browser as well as that I used linux, now the first thing which struck my mind on how websites identify their client's browser and OS version is through user-agent.
The page says the community members use old windows laptops, hence that means it needs an user agent which would be coming from an old windows laptop which could possibly be from internet explorer, since that's pretty old.
So we simply google internet explorer's user agent string, and went all the way back to windows xp, copy pasted it onto our request using burp or by using curl or something and got our flag

![image](https://user-images.githubusercontent.com/64488123/167371875-5620f4ee-9d00-4973-a15b-8ce19e7acf97.png)


## UnChangable Password

The URL is https://web.ctf.devclub.in/web/2/login.php and then chall description is "You can create an account ... sure. But we **DARE** you change the Passwords..".
After visiting the site we get greeted to a login site

![image](https://user-images.githubusercontent.com/64488123/167372089-a0a1c73e-395e-4b08-b9c2-c381e4bcd78d.png)

Here we can simply make our own account and change our own password. Cool!
The chall description is about changing passwords...ok, lets make 2 accounts for testing purposes and look at the http requests with some proxy like burp.
So we make 2 accounts, 1st 1 having USERID and NAME as foobar1 and 2nd as foobar2 respectively.
Now lets change the password of foobar1. The traffic is as follows

![image](https://user-images.githubusercontent.com/64488123/167373152-51080095-9495-4c2b-a3f7-67e14fd61dbc.png)

Now lets let the traffic play on repeater

![image](https://user-images.githubusercontent.com/64488123/167373244-2441e958-1f30-4715-8616-eaae3aef04ed.png)

### Unintended of the above

when we catted the app.py we got the following 

![image](https://user-images.githubusercontent.com/64488123/167380736-dac8d055-b150-466f-a551-803f14e632ac.png)

Here we see an echo which has an endpoint /echo which is some kinda API which echo's stuff. lets try it

![image](https://user-images.githubusercontent.com/64488123/167380955-26804693-f116-4a12-9e89-793b458b67a0.png)

Trying the SSTI payload in here

![image](https://user-images.githubusercontent.com/64488123/167381119-6d74c966-5f89-4958-a857-c44edecf9728.png)

We might have gotten SSTI through here too

We see there is a message that the password has been reset for foobar1. Ok, in the post request we see the USER_ID and the new PASSWORD, lets do 1 thing and lets try changing the USER_ID to foobar2

![image](https://user-images.githubusercontent.com/64488123/167373523-9bf797f1-5cf7-42d7-bb6a-a0dd844f989d.png)

In here we see we got the message that the password has been reset for foobar2 as well, which is the hack here, as we never authenticated as foobar2. Hence we see that we can change any user's password, so lets try changing admin's password

![image](https://user-images.githubusercontent.com/64488123/167373829-69160da2-130d-4415-a70e-30a2cce3ae53.png)

and boom we have admin creds.
Now we can login as admin


## The Sequel

We are given a URL https://web.ctf.devclub.in/web/1/login.php and the chall description says "Can you hack this super secret login page ????"
The site presents us with a login page

![image](https://user-images.githubusercontent.com/64488123/167374888-a801487a-9be1-4541-9c2c-ed66973f6893.png)

As the name "sequel" suggest SQL, hence wetry some SQLi by first using a ' which gives us this error

![image](https://user-images.githubusercontent.com/64488123/167375086-866c8931-b92f-48e6-b382-9d90750f8515.png)

Hence it's a confirmed SQLi in a post request, hence we save the http request using burp into a file and then we simply fireup sqlmap and let it do its stuff, and let it enumerate the database

![image](https://user-images.githubusercontent.com/64488123/167375686-03d2c3fb-eac9-4119-b1fb-badfa6a0554f.png)

Ohk, so this tbl_users seems interesting, dumping the stuff from this table and cracking the hash,  we get the user credentials for the portal


## Flag Auction

We are given a URL https://web.ctf.devclub.in/web/4/ and the challenge description says "Yay !!! The Flag is on sale !!!! But are you okay with paying the "Gold Price" or will you pay the "Iron Price" ?"
We are presented with a site

![image](https://user-images.githubusercontent.com/64488123/167377234-111f23e4-07f9-400b-98f2-7e645c2a4b1a.png)

Whenever we try to buy anything this page momentarily pops up

![image](https://user-images.githubusercontent.com/64488123/167377394-cef1bd43-de2d-47cd-8f69-641ac54aebb9.png)

and then a alert box comes in stating some irrelevant stuff. Its irrelevant because our interest lies in this page, where the product name is written, note that it is written in bold. Lets run it through burp

![image](https://user-images.githubusercontent.com/64488123/167377708-a6fd50df-79e2-4e48-955f-92d71b437d09.png)

We see the CTF Flag in bold

We see upon changing the request the response changes as well

![image](https://user-images.githubusercontent.com/64488123/167378168-831ce7c3-1ea5-4aef-a500-247f0a749be1.png)

And the title says python flash, so it might be made on flask. And knowing flask, it has a template injection vulnerability as we can see here

![image](https://user-images.githubusercontent.com/64488123/167378631-229418d0-398e-4f3d-9913-e7bbaad6154d.png)

Hence we simply follow this article over here https://medium.com/@nyomanpradipta120/ssti-in-flask-jinja2-20b068fdaeee (yes, i am not going through all those steps in this writeup, just read how to do an SSTI from that article). Getting RCE with this

![image](https://user-images.githubusercontent.com/64488123/167379885-a07c718a-4fe3-4133-86cb-d5e284fdf880.png)

We enumerate the environment variables by looking at `/proc/self/environ`

![image](https://user-images.githubusercontent.com/64488123/167380071-51594f2e-b138-456a-8213-e2c36ffb388c.png)

We see a FLAG_DIR and catting all files in them we get the flag

![image](https://user-images.githubusercontent.com/64488123/167380437-04d1433d-e867-490e-a6ca-6e3b0d07d9f5.png)

### uninteded for the above

There was an unintended to /echo path 

![image](https://user-images.githubusercontent.com/64488123/167381374-b9ccb6ea-7678-44d0-8bf5-aa04f792374d.png)

This was found by catting the source code of the app.py using the 1st SSTI 
