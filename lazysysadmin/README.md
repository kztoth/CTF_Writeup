# LazySysAdmin

Source: [https://www.vulnhub.com/entry/lazysysadmin-1,205/](https://www.vulnhub.com/entry/lazysysadmin-1,205/) 

First off we need to find our target on the network so let's pop open net discover.

```
netdiscover -r 10.0.2.0/24
```

![netdiscover](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/netdiscover.PNG)

We see that it is at 10.0.2.4. Now let's find some better information so we run an nmap scan to see what ports it has open.

![nmap](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/nmap.PNG)

We find that it has SSH, HTTP, Samba, mysql and irc as open ports so let's go look at HTTP first.

![http](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/http.PNG)

I saw that near the bottom it said that 'The answer is within you'. So I decided to look at the source of it but there turned out to be nothing special there. After some poking around on this page I decided to use dirBuster to find other available webpages. 

![dirb](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/dirb.PNG)

The directories that stick out to me are phpmyadmin and wordpress. I didn't have any information about the login currently so I picked word press to investigate first.

![wordpress](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/wordpress.PNG)

This is the only post on this server. I think that we have found a username. Looking at the author of this post we find something that could be related to a password.

![author](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/author.PNG)

After trying to plug some passwords into phpmyadmin I decided I was currently at a dead end so I went back and did some more enumeration. Using 'enum4linux 10.0.2.4' I was able to find information on what samba was hosting.

![enum4linux](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/enum4linux.PNG)

share$ seems like it would be interesting to access so let's try to connect with smbclient. Surprisingly there was no password on this file so we were able to get in with no trouble.

![smbclient](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/smbclient.PNG)

There were 2 interesting files here: todolist.txt and deets.txt.

![smbfiles](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/smbfiles.PNG)

Well here is an actual password but it is not for the wordpress site so it might be something else. Let's head into wp and take a look there. 

![wp-config](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/wp-config.PNG)

Looks like the password was changed a little bit at least. Now that we have actual credentials let's head back to phpmyadmin and login. After loging in and trying to look at any tables it is clear that there is nothing available to us as everything returns an error.

![error](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/error.PNG)

Well this seems like another dead end so let's head back to the nmap and see what else there is. We have not tried ssh yet but now we have a potential username, togie, and 2 different passwords to try. I tried 12345 first because it was easier to type and that happened to be the correct one. 

![ssh](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/ssh.PNG)

Navigating to root completes the challenge and gives us our flag.

![flag](https://github.com/kztoth/CTF_Writeup/blob/master/lazysysadmin/pics/flag.PNG)