## BOF GUIDE
### Background
- C code complies into machine code which will be X64 in our case.
- C Code is straightforward, so things need to be declared in advance, like the sizes of variables.
- In C, variables are addressed in memory, so there is nothing to check sizes; that's the programmer's job
- Locally declared variables like on the stack, a special part of memory
- After a function finishes theirs no way to hard code a place to return to so that is also stored on the stack
- When a function is called a stack frame is created, boundaries for the end of the stack
- EXAMPLE:
```c
int main(){
int foo = 0x1337;
int bar = 0xdeadbeef;
}
```
- Stack could look something like this 
```
0xdeadbeef       <---- bar
0x1337           <---- foo 
RANDOM           <---- Base pointer RBP
0xcafebabe       <---- Return address
```
## Exploitation
- Notice how the variables first declared are closer to the return address; this is because the stack is upside down and grows upwards(Low memory addresses)
- After the function reaches the end, it needs to jump back into the caller function(In this case its libc stuff but ignore that for now) by executing the code at the return address
- This means that if we can overwrite that address, we will be able to redirection execution to anything such as another function
- A ret2win is a very basic version of this exploit as a win function exists but under normal circumstances is not called, but we can force that to happens
- For example, if we were able to overwrite starting at bar with A(In hex A is 0x41), the stack would look something like this after execution
```
0x4141414141414141   <---- bar
0x4141414141414141   <---- foo 
0x4141414141414141   <---- Base pointer RBP
0x4141414141414141   <---- Return address
```
- After the function is done, it will do a ret instruction which will move the return address into the instruction pointer, thus executing whatever we placed their

## How to find the offset
- Once you have confirmed buffer overflow, you need to find the right size to overwrite before the return address is hit, as writing too much can cause a Segmentation fault(Which means that you tried to access nonexistent data) 
- The two main methods are either static or dynamic, here is a brief guide for both
### Static
- The returned pointer is stored 8 bytes after the base pointer so it's at RBP+0x8.
- Looking at the disassembly we see that foo is at RBP-0x8, this means that we are 16 bytes away from the return address
- Thus sending 16 bytes and then the return address should correctly place it
```
Dump of assembler code for function main:
   0x0000000000401126 <+0>:     push   rbp
   0x0000000000401127 <+1>:     mov    rbp,rsp
   0x000000000040112a <+4>:     sub    rsp,0x10
   0x000000000040112e <+8>:     mov    DWORD PTR [rbp-0x8],0x1337     <----rbp-0x8
   0x0000000000401135 <+15>:    mov    DWORD PTR [rbp-0x4],0xdeadbeef
   0x000000000040113c <+22>:    lea    rax,[rbp-0x8]
   0x0000000000401140 <+26>:    mov    rdi,rax
   0x0000000000401143 <+29>:    mov    eax,0x0
   0x0000000000401148 <+34>:    call   0x401030 <gets@plt>
   0x000000000040114d <+39>:    mov    eax,0x0
   0x0000000000401152 <+44>:    leave
   0x0000000000401153 <+45>:    ret
```


### Dynamic
- Instead of staring at source code just spam characters and hope for a crash
- Running inside a debugger like gdb will make it easier to understand what's going on ( Using the gef plugin )
- Using something like pwn cyclic to generate an identifiable string we can just send it and hope for a crash
- Using info frame we can get the exact address of the RIP and examine whats their
- Searching the offset we can find the amount of bytes we need to overwrite


```
 ~/ctf/idek/test >> gdb foo
Reading symbols from foo...
gef➤  pattern create 100   
[+] Generating a pattern of 100 bytes (n=4)  # Creating a pattern
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa
[+] Saved as '$_gef0'
gef➤  run
Starting program: /home/ex4722/ctf/idek/test/foo   # Run and send pattern
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

Program received signal SIGSEGV, Segmentation fault.  # CRASHED
0x0000000000401153 in main () at foo.c:7
     0x401148 <main+34>        call   0x401030 <gets@plt>
     0x40114d <main+39>        mov    eax, 0x0
     0x401152 <main+44>        leave
 →   0x401153 <main+45>        ret
[!] Cannot disassemble from $PC
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────── source:foo.c+7 ────
      2  int foo = 0x1337;
      3  int bar = 0xdeadbeef;
      4  gets(&foo);
      5  return 0;
      6
 →    7  }
      8
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  info frame
Stack level 0, frame at 0x7fffffffdbb0:
 rip = 0x401153 in main (foo.c:7); saved rip = 0x6161616661616165
 source language c.
 Arglist at 0x6161616461616163, args:
 Locals at 0x6161616461616163, Previous frame's sp is 0x7fffffffdbb0
 Saved registers:
  rbp at 0x7fffffffdba0, rip at 0x7fffffffdba8
gef➤  x/gx 0x7fffffffdba8
0x7fffffffdba8: 0x6161616661616165
gef➤  pattern search 0x6161616661616165     # Search for pattern that is in RIP now
[+] Searching for '0x6161616661616165'
[+] Found at offset 16 (little-endian search) likely
```
