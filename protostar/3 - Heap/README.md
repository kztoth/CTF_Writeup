# Heap

Attacks on the heap.

## Challenge 0

This first heap problem is pretty simple so let's just test what happens when we run it.

![heap0_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap0_0.PNG)

This is the typical result we have been observing from the challenges up to this point. Taking a look at the disassembly we can see that it calls malloc twice, copies our input string, then loads a value into eax which is then called at < main+113 >.

![heap0_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap0_1.PNG)

Now let's add a breakpoint at < main+111 > and take a look at the heap. We can use 'info proc mappings' to find the address of the heap.

![heap0_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap0_2.PNG)

Now that we have the address of the heap and our breakpoint set let's run the program. At the break point we can view the heap with 'x/32wx 0x0804a000'.

![heap0_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap0_3.PNG)

With the heap we see that our string is on there first and then the next item on the stack looks like an address. Using 'info functions' we find that there are 3 functions: main, nowinner, and winner.

![heap0_4](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap0_4.PNG)

The value on the heap is the same as nowinner so that means we just need to overwrite it with winner. We use that knowledge to write a simple python exploit with an offset of 0x58 before the value we need to change.

```python
import sys;
offset = 0x58
exploit = ""
exploit += "A"*offset
exploit += "\x64\x84\x04\x08"
sys.stdout.write(exploit)
```

Unlike the other python scripts before this we use sys.stdout.write() instead of just print because print adds 0x0a to the end of the string for a new line. If we use that here we will jump to 0x080484640a which is far off from what we need. Now we just put that in a file which we pass to heap0 and we get our flag.

![heap0_5](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap0_5.PNG)

## Challenge 1

This is slightly more complex than the previous but still the same concept. After a little playing around we discover that this challenge requires 2 arguments to be passed into it.

![heap1_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap1_0.PNG)

When a large argument is given we find that argument 1 overflows the heap so that it's value is the destination that argument 2 is being copied to. Now we can do a bit of investigation to find how big the buffer is.

![heap1_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap1_1.PNG)

0x46 is F so we have an offset of 20 before the address. Now we just need something to overwrite with the strcpy. We can look at the disassembly to see if there is anything helpful.

![heap1_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap1_2.PNG)

The only thing after the strcpy is a call to puts. Going back to previous challenges we know that we can overwrite the global offset table to point to something we want to execute. Using 'info functions' we see that it has a winner function again so we can examine that to find where to jump to.

![heap1_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap1_3.PNG)

Now that we have the address to puts in the global offset table and the address we need to jump to we can write our exploit and get the flag.

```python
import sys
offset = 20
exploit = ""
exploit += "A"*offset
exploit += "\x74\x97\x04\x08 "
exploit += "\x94\x84\x04\x08"
sys.stdout.write(exploit)
```

![heap1_5](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap1_4.PNG)

## Challenge 2

This challenge introduces the use-after-free vulnerability. Looking at the [source code](https://exploit-exercises.com/protostar/heap2/) we see that there are 4 different commands after running it. 'auth [input]' to malloc space for the auth object, 'reset' to free the auth object, 'service[input]' to input a string, and 'login' to check if you have cleared the challenge. Unfortunately this is a poorly written challenge and can be beaten by chance just playing around.

![heap2_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap2_0.PNG)

This happens because the size of auth is not correct because there are 3 different things in the code named auth. There is a struct auth, int auth, and struct auth pointer and the compiler happened to not pick the struct. 

The way it wants you to beat it is to auth, reset, then create services until you change the right value within auth.

![heap2_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/3%20-%20Heap/pics/heap2_1.PNG)

As you can see after freeing auth with reset it still points at the space where it was. That space in the heap is free so malloc will put the next item there which happens to be service which is why they have the same address. When you try to login the program still checks auth and because it still has a pointer you can use it after it was freed.

## Challenge 3







