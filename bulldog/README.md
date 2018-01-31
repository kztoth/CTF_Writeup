# Bulldog

VM Source: [https://www.vulnhub.com/entry/bulldog-1,211/](https://www.vulnhub.com/entry/bulldog-1,211/)

This was an easy boot to root of a web server. It is about bulldog industries rebuilding their site because of a recent security breach.

The first step is find the server on the network.

![netdiscover](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/netdiscover.PNG)

We found it at 10.0.2.4. Lets go take a look at the website now.

![index](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/index.PNG)

![notice](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/notice.PNG)

There are only 2 pages that are easily reachable. There is not much to go on so far so let's do some enumeration. I decided my first step would be to look at if there were any more web pages available so I ended up using dirbuster to run over a list of common directory names.

![dirb](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/dirb.PNG)

Just looking at what was found the links that pop out to me are /dev/shell/ and /admin/. These could both be ways to gain access closer to what we want. Heading to /dev/shell/ tells us that we must authenticate first to gain access. At /admin/ there is a basic Django login page which I had no credential for yet so I decided to go check out /dev/ to see if that contained anything helpful.

![dev](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/dev.PNG)

Well here is some useful information finally. We have 7 potential usernames so we just need to find their passwords. From previous vulnhub challenges I have made a habit of inspecting the source http to see if there is any weird comments hiding in there. Once again it pays off and we find hashes.

![inspect_dev](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/inspect_dev.PNG)

Now that we have some hashes we can head over to [crackstation.net](https://crackstation.net) and plug them in. We find that 2 of the passwords come up.

![crackstation](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/crackstation.PNG)

Nick and Sarah are using bulldog and bulldoglover respectively which is probably not a good idea considering that is just the name of the company they are working for. Let's use this information to log into the web-shell.

![web-shell](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/web-shell.PNG)

This is trying to be a restricted shell so let's try something it doesn't want like 'id'.

![hacker](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/id.PNG)

Seems like they are on to us. After playing around a bit I figured out that you could use | or & to get around their restrictions.

![blocked_command](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/blocked_command.PNG)

![asdf](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/asdf.PNG)

Now that we have a way to execute what we want we can use wget to grab an exploit from a server we set up and then execute it for a reverse shell. I used a script that I found on [pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) for this and I hosted it with a basic python server for the web-shell to grab from.

![wget-command](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/wget-command.PNG)

![script_serve](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/script_serve.png)

With the file now on the target server we are able to run the script while we have netcat open on our machine and we are in.

![runscript](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/runscript.PNG)

![connection](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/connection.PNG)

Poking around some we find a folder called bulldogadmin. Looking inside there is a hidden folder called .hiddenadmindirectory.

![hidden](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/hidden.png)

Using cat to look at the note it seems as if the customPermissionApp should contain the password. Checking that file for strings proves that to be true.

![password](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/password.PNG)

It looks like the password is SUPERultimatePASSWORDyouCANTget. Now all we need to do is go collect our flag.

![flag](https://github.com/kztoth/CTF_Writeup/tree/master/bulldog/pics/flag.png)