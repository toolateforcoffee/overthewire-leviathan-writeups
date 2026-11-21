# Understanding the challenge

  For every leviathan challenge we don't get any info regarding what we have to do. 

  After accessing, by remote access, into the machine of the user leviathan1 on host leviathan.labs.overthewire.org we get to analyze a single file: `./check` 

## The check executable

  Let's check the executable permissions info:
```
leviathan1@leviathan:~$ ls -l check
-r-sr-x--- 1 leviathan2 leviathan1 15080 Oct 25 13:39 check
```
When a binary has the s permission in the owner's execute position, any user who has the permission to execute the file (in this case we have permission) will run it with the same privileges of the file's owner.

  Let's execute the check:
```
leviathan1@leviathan:~$ ./check
password: random
Wrong password, Good Bye ...
leviathan1@leviathan:~$
```
  The `check` executable is just a password checker. I suppose, if we give it the correct password, then it will give us the access code for the next leviathan challenge.

## Disassembly Content

  I would use gdb to analyze the main function of the executable:
  `gdb -batch -ex "set disassembly-flavor intel" -ex "disass main check`
  (You can do the same directly on interactive mode)

  Note: I prefer Intel assembly syntax, as that's the easiest one.

  Output: 

```nasm
Dump of assembler code for function main:
   0x080491d6 <+0>:     lea    ecx,[esp+0x4]
   0x080491da <+4>:     and    esp,0xfffffff0
   0x080491dd <+7>:     push   DWORD PTR [ecx-0x4]
   0x080491e0 <+10>:    push   ebp
   0x080491e1 <+11>:    mov    ebp,esp
   0x080491e3 <+13>:    push   ebx
   0x080491e4 <+14>:    push   ecx
   0x080491e5 <+15>:    sub    esp,0x20
   0x080491e8 <+18>:    mov    eax,gs:0x14
   0x080491ee <+24>:    mov    DWORD PTR [ebp-0xc],eax
   0x080491f1 <+27>:    xor    eax,eax
   0x080491f3 <+29>:    mov    DWORD PTR [ebp-0x20],0x786573
   0x080491fa <+36>:    mov    DWORD PTR [ebp-0x13],0x72636573
   0x08049201 <+43>:    mov    DWORD PTR [ebp-0x10],0x746572
   0x08049208 <+50>:    mov    DWORD PTR [ebp-0x1c],0x646f67
   0x0804920f <+57>:    mov    DWORD PTR [ebp-0x18],0x65766f6c
   0x08049216 <+64>:    mov    BYTE PTR [ebp-0x14],0x0
   0x0804921a <+68>:    sub    esp,0xc
   0x0804921d <+71>:    push   0x804a008
   0x08049222 <+76>:    call   0x8049050 <printf@plt>
   0x08049227 <+81>:    add    esp,0x10
   0x0804922a <+84>:    call   0x8049060 <getchar@plt>
   0x0804922f <+89>:    mov    BYTE PTR [ebp-0x24],al
   0x08049232 <+92>:    call   0x8049060 <getchar@plt>
   0x08049237 <+97>:    mov    BYTE PTR [ebp-0x23],al
   0x0804923a <+100>:   call   0x8049060 <getchar@plt>
   0x0804923f <+105>:   mov    BYTE PTR [ebp-0x22],al
   0x08049242 <+108>:   mov    BYTE PTR [ebp-0x21],0x0
   0x08049246 <+112>:   sub    esp,0x8
   0x08049249 <+115>:   lea    eax,[ebp-0x20]
   0x0804924c <+118>:   push   eax
   0x0804924d <+119>:   lea    eax,[ebp-0x24]
   0x08049250 <+122>:   push   eax
   0x08049251 <+123>:   call   0x8049030 <strcmp@plt>
   0x08049256 <+128>:   add    esp,0x10
   0x08049259 <+131>:   test   eax,eax
   0x0804925b <+133>:   jne    0x8049288 <main+178>
   0x0804925d <+135>:   call   0x8049080 <geteuid@plt>
   0x08049262 <+140>:   mov    ebx,eax
   0x08049264 <+142>:   call   0x8049080 <geteuid@plt>
   0x08049269 <+147>:   sub    esp,0x8
   0x0804926c <+150>:   push   ebx
   0x0804926d <+151>:   push   eax
   0x0804926e <+152>:   call   0x80490b0 <setreuid@plt>
   0x08049273 <+157>:   add    esp,0x10
   0x08049276 <+160>:   sub    esp,0xc
   0x08049279 <+163>:   push   0x804a013
   0x0804927e <+168>:   call   0x80490a0 <system@plt>
   0x08049283 <+173>:   add    esp,0x10
   0x08049286 <+176>:   jmp    0x8049298 <main+194>
   0x08049288 <+178>:   sub    esp,0xc
   0x0804928b <+181>:   push   0x804a01b
   0x08049290 <+186>:   call   0x8049090 <puts@plt>
   0x08049295 <+191>:   add    esp,0x10
   0x08049298 <+194>:   mov    eax,0x0
   0x0804929d <+199>:   mov    edx,DWORD PTR [ebp-0xc]
   0x080492a0 <+202>:   sub    edx,DWORD PTR gs:0x14
   0x080492a7 <+209>:   je     0x80492ae <main+216>
   0x080492a9 <+211>:   call   0x8049070 <__stack_chk_fail@plt>
   0x080492ae <+216>:   lea    esp,[ebp-0x8]
   0x080492b1 <+219>:   pop    ecx
   0x080492b2 <+220>:   pop    ebx
   0x080492b3 <+221>:   pop    ebp
   0x080492b4 <+222>:   lea    esp,[ecx-0x4]
   0x080492b7 <+225>:   ret
End of assembler dump.
```

## Main Sections

### First Section, Canary Value

  **Stack Overflow** 
  A stack overflow happens when, in the stack, a certain portion of memory uses more data (more bytes) than it was supposed to use causing that data to flow onto other addresses (upward, towards higher addresses) and stack positions.

  **How Does The CPU Avoid This Exploitaion?**

  By using a stack canary. A stack canary is a 4Bytes random number that the CPU checks in order to see if an attacker tried to cause a stack overflow to gain access to certain data.

  **How it works**

  The CPU pushes the stack canary value onto the stack (instruction at address `0x080491e8`). If a stack overflow happened then data would flood onto already occupied stack positions modifying pre-existing data, causing the pushed stack canary to have a different value. 

  The CPU compares the pushed value with the original stack canary and if they are different it throws a stack failure exception (`<__stack_chk_fail@plt>`)

  **How does the CPU get the stack canary?**

  By executing this instruction:
  `0x080491e8 <+18>:    mov    eax,gs:0x14`

  The CPU has access to the Stack canary value through a specific thread.
  
  While a program consists of data living inside the disk, and a process is created when a program is ran and its data is moved into the RAM we can think about the thread as a sort of independent worker within the process that have a specific role. Every thread has a storage for itself inside the RAM, the TLS (Thread Local Storage). Every TLS has a base pointer register, meaning a register that points to the base of the TLS.

  gs is a register (living inside the CPU) that points to the base of the thread containing the Stack canary value.

  `gs:0x14` is the specific address for the Stack canary data inside the thread. It moves of 0x14 positions after the address pointed by gs.


