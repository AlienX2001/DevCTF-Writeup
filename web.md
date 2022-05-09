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

We see there is a message that the password has been reset for foobar1. Ok, in the post request we see the USER_ID and the new PASSWORD, lets do 1 thing and lets try changing the USER_ID to foobar2

![image](https://user-images.githubusercontent.com/64488123/167373523-9bf797f1-5cf7-42d7-bb6a-a0dd844f989d.png)

In here we see we got the message that the password has been reset for foobar2 as well, which is the hack here, as we never authenticated as foobar2. Hence we see that we can change any user's password, so lets try changing admin's password

![image](https://user-images.githubusercontent.com/64488123/167373829-69160da2-130d-4415-a70e-30a2cce3ae53.png)

and boom we have admin creds.
Now we can login as admin

