# Stack

These are all stack based buffer overflow challenges of increasing difficulty.

## Challenge 0

This challenge was just an introduction to the concept. Inputting enough characters will change the variable following the buffer and win.

![stack0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack0.PNG)

## Challenge 1

This challenge gets more specific with your overflow and forces you to find the correct value for the variable you are overwriting.

![stack1_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack1_0.PNG)

As we can see we overwrote the variable with 0x51 which is Q in ascii. So now we know the offset to what we are trying to overwrite.

![stack1_1_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack1_1_2.png)

Using gdb to disassemble the main function we find a compare instruction at main + 71. This compares the input string to 0x61626364 which translates to abcd in ascii. This system is little endian so the exploit will be 'offset + dcba'.

![stack1_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack1_3.PNG)

## Challenge 2

So far we have input our string with gets and command arguments. This challenge changes it up again by using environment variables.

![stack2_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack2_0.PNG)

At this point I had enough fun typing out the overflow string by hand so I decided it was time to write a python script that I can edit instead. The script just prints the offset and will be edited to print the exploit after it once we know the value. 

![stack2_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack2_1.PNG)

When we run the program we find that the offset is the same as the previous challenge which is at Q.

![stack2_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack2_2.PNG)

Now we just need to find out what the value needs to be so we open up gdb again.

![stack2_3_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack2_3_2.png)

Looking at the compare at main + 84 we find that the value needs to be 0x0d0a0d0a. 0x0d is the ascii value of return. This caused a bit of a headache trying to put it into the environment variable with the above process because it would take it as a new line instead of just appending the hex value. I figured out a workaround by both setting the variable and then calling the function from the python script.

![stack2_5](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack2_5.PNG)

## Challenge 3

This challenge is basically the same as the number challenge 1 except instead of finding the right value for an if statement we need to jump to the right place in the code.

![stack3_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack3_0.PNG)

Looking at the main function we can see it compares the variable we are trying to overwrite to 0x0 at main + 29. It will then jump to exit if that is true or it will try and call that variable at main + 61. This means there should be some other place in the code that will give us our flag.

![stack3_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack3_1.PNG)

Using 'info functions' we can see that there is another function called win.

![stack3_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack3_2.PNG)

I disassembled win to see what was in it and all it has is just printing the flag it seems. This is the location that we need to jump to so we just need to set the variable to it's address 0x08048424.

![stack3_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack3_3.PNG)

Because this challenge uses gets I use the python script to put the exploit into a file which I can then use through gdb. 

![stack3_4](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack3_4.PNG)

Using gdb to input the file gave us the flag.

![stack3_5](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack3_5.PNG)

## Challenge 4

This is the same concept as the previous challenge but instead there is no variable or function being called. 

![stack4_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack4_1.PNG)

As we can see from the assembly all that happens in this program is it calls gets. Assuming this was the same as the previous I checked the functions and found it had the same win function.

![stack4_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack4_2.PNG)

So now how are we going to do anything when there is nothing to manipulate? Well there is still stuff on that stack that we can manipulate. Specifically the instruction pointer. This is a pointer for where the program will return to next after any function calls return. Our goal is now to change that pointer value to the value of win so that return is effectively calling win by popping that address into eip.

Using the command 'x win' I found the function location and then ran the program to find what the offset needed to be.

![stack4_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack4_3.PNG)

In all the past challenges we have seen that 0x51515151 seemed to be the preferred offset but this one is 0x54545454. This is because there are some other things on the stack between buffer and the instruction pointer. 0x54 is T so I changed the python script to a longer offset and input the address we were jumping to.

![stack4_5](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack4_5.PNG)

We put our exploit in and we get out our flag. There is a SIGSEGV because the program does not know where to go back to once the win function returns.

![stack4_6](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack4_6.PNG)

## Challenge 5

This challenge starts to move towards more real world application. With the previous 2 challenges using another function I decided the first thing to do after confirming there was an overflow was to run 'info functions'.

![stack5_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack5_1.PNG)

It turns out that this time we are back to just having one function. Let's take a look at the disassembly.

![stack5_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack5_2.PNG)

At this point I was confused as to how I would get the flag so I decided to look at the about page for this challenge.

![stack5_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/1%20-%20Stack/pics/stack5_3.PNG)

Okay it seems as if the goal here is to just run some shellcode. It gave a hint that I should just use some that is already made but I decided it would be a good learning experience to write my own so that is what I am currently researching. Will be updating this after I get it working.

## Challenge 6



## Challenge 7



