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

  **How Does The CPU Avoid This Exploitation From Hackers?**

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

  This value is saved into eax register and then moved to address ebp-0xc.

  eax register is then setted to zero (`0x080491f1 <+27>:    xor    eax,eax`)

  ## Second Section

  ```nasm
  0x080491f3 <+29>:    mov    DWORD PTR [ebp-0x20],0x786573
  0x080491fa <+36>:    mov    DWORD PTR [ebp-0x13],0x72636573
  0x08049201 <+43>:    mov    DWORD PTR [ebp-0x10],0x746572
  0x08049208 <+50>:    mov    DWORD PTR [ebp-0x1c],0x646f67
  0x0804920f <+57>:    mov    DWORD PTR [ebp-0x18],0x65766f6c
  0x08049216 <+64>:    mov    BYTE PTR [ebp-0x14],0x0
  ```

  In this section some hex values are moved into specific memory points. By converting them into ascii characters we can see that:

  ```text
  0x786573 -> sex
  0x72636573 -> secr --> secret
  0x746572 -> et ------|
  0x646f67 -> god
  0x65766f6c -> love
  ```
  ## Third Section : The Three Characters

  ```nasm
  0x08049222 <+76>:    call   0x8049050 <printf@plt>
  0x08049227 <+81>:    add    esp,0x10
  0x0804922a <+84>:    call   0x8049060 <getchar@plt>
  0x0804922f <+89>:    mov    BYTE PTR [ebp-0x24],al
  0x08049232 <+92>:    call   0x8049060 <getchar@plt>
  0x08049237 <+97>:    mov    BYTE PTR [ebp-0x23],al
  0x0804923a <+100>:   call   0x8049060 <getchar@plt>
  0x0804923f <+105>:   mov    BYTE PTR [ebp-0x22],al
  0x08049242 <+108>:   mov    BYTE PTR [ebp-0x21],0x0
  0x08049249 <+115>:   lea    eax,[ebp-0x20]
  0x0804924c <+118>:   push   eax
  0x0804924d <+119>:   lea    eax,[ebp-0x24]
  0x08049250 <+122>:   push   eax
  0x08049251 <+123>:   call   0x8049030 <strcmp@plt>
  0x08049256 <+128>:   add    esp,0x10
  0x08049259 <+131>:   test   eax,eax
  0x0804925b <+133>:   jne    0x8049288 <main+178>
  ```
  The program ask for an input from the user (asking for the password).
  There are three getchar() function calls. It seems that the program considers only the first three characters we put as input and the others one remain in the input buffer. 0x0 is the null terminator used for marking the end of the string.

  This string (first three characters from our input) is then pushed onto the stack.
  The value `[ebp-0x24]` is the one that was previously moved inside that address in the instruction at address `0x080491f3 <+29>`. We already know that it corresponds to "sex". That value too will be pushed now onto the stack (instruction `0x0804924d`  and `0x08049250`)

  Both of these two values will be used as parameters for the strcmp function and will be compared.

  `test eax,eax` will do an AND operation between eax and eax itself and won't modify the operands. This is used for verifying if eax is zero or not and updating the eflags. If eax is zero ZF flag becomes 1, SIGN FLAG assumes the value of the leftmost bit. CF is setted to zero.

  If the value at `ebp-0x24` and the first characters of our input are not equal then the program will jump to instruction `0x08049288` and print "Wrong password, Good Bye ...":

  ```nasm  
  0x08049288 <+178>:   sub    esp,0xc
  0x0804928b <+181>:   push   0x804a01b
  0x08049290 <+186>:   call   0x8049090 <puts@plt>
  ```
  ---
  ```nasm
  (gdb) x/s 0x0804a01b
  0x804a01b:      "Wrong password, Good Bye ..."
  ```

  ## Fourth Section: the Effective And Real User IDS
  
  ```nasm
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
  ```

  Real User ID : The ID of the user that actually launched a program.
  Effective User ID: The ID of the user whose privileges are used for running the program.

  This section of code firstly get the effective user id and moves this value from eax (usually contains the return value) to ebx (general purpose register).
  The same getuid() function is called another time, and that value is again stored inside eax register.

  Remember that the calling convention used is cdecl, where parameters are read and then pushed from right to left, BUT then interpreted in the function call from left to right. 

  **Why**: First ebx is pushed (rightmost argument), and then eax(first argument). The Stack grows downward, meaning eax is on top of ebx. The function reads the memory from lower addresses to upper addresses, meaning it will read first eax and then ebx. For this reason the first parameter for that function call is eax and then ebx.

  The function called at address `0x0804926e` might look like this : `setreuid(eax, ebx)`

  setreuid will set the real and effective ids to the previous already setted effective id.

  ## FIFTH SECTION: The system() function

  ```nasm
  0x08049279 <+163>:   push   0x804a013
  0x0804927e <+168>:   call   0x80490a0 <system@plt>
  0x08049283 <+173>:   add    esp,0x10
  0x08049286 <+176>:   jmp    0x8049298 <main+194>
  ```

  ```nasm
  (gdb) x/s 0x804a013
  0x804a013:      "/bin/sh"
  ```
  
  `call   0x80490a0 <system@plt>` will call the <system@plt> function using as parameter "/bin/sh". 

  system() is a function from the c standard library (lbc) that creates and manages processes. by doing system(command) it will create a child process, in particular it executes a shell and execute in it the command passed as parameter. Basically it does:
  /bin/sh -c "command"

  Basically, after entering the correct three characters as input we will get a sh shell, and will have the same privileges as the effective user, leviathan2:

  After entering "sex" or "sexblablabla"(doesn't matter, only the first three characters will be considered) we enter in a shell (sh):

  ```nasm
  leviathan1@leviathan:~$ ./check
  password: sex
  $
  ```

  After executing `cat /etc/leviathan_pass/leviathan2` we get the access code for leviathan2.

