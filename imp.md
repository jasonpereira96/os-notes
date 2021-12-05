## boot
boot - how does it work?
what is a bootloader and its relationship to BIOS
how do we start running?
What is happening in the execution from the entry point in the kernel all the way until you start running the first user program init
configure the hardware in various ways, in particlar the interrupt tables
initial page table
once we've started running init -  we have another page table
the free list of free pages

## Memory mamagesment
Structure and function of page tables
The fact that we have one for every process
mmap homework
sbrk
malloc is in userspace

## page faults and how they are handled
lazy allocation in mmap
we get to see the address that the page fault occured at in cr2
kalloc() / kfree() and how it works

## kernel page table and process page table

Take a look at what's there in a process page table - the whole thing

When do we swtich page tables? What does it mean to swtich page tables?


## Process 
we have our own unique virtual memory address space
unique set of regs
how do we backup the regs during a context swtich

user vs kernel reg contents 
down the road while switching, we start running with some other processes kernel reg contents
Eventually we come back to where we called swtch and then those kernel reg contents are restored

So we're saving both user regs and kernel regs
Where is the value of rax stored?
Fuck! There are 2 values of rax!
**Where are they stored? The kernel and user register copies in memory?**
Where on the stack? There are 2 stacks

All the user reg values are on the top of the kstack (They're push in trap.asm)
We point the tf to that so we can access them if we need to

rax is pushed by the compiler
rax is a caller save register - so if the compiler has anything important in rax at the time, it has to push it onto the stack befire calling a function. That was a trick question.

There are 2 copies of them and they're both in the kstack, one at the top and one at the bottom


## File diesctrptors
- stdin (0)
- stdout (1)
- stderr (2)

How does 1 translate to writing to the termiail

Look at the ofile[] array. How it is made and used

## Mode switch
- user mode
- supervisor mode

How do you get from user mode to supervisor mode? - ONLY via syscall and int
- syscall
- sysret /  iret

## Very important
to take a look at the switch code and refresh on that
How do we switch to another process and another process -  from yield() onwards

## swtch() - very important

There is a special kernel thread - it has it own context which is stored in the **The cpu struct**

## important
the fact that there's a seperate context for the schduler and one for every struct proc

## system calls - be well familiar
All the way from when the user calls write()
Try to follow a syscall all the way through
Read `read()`

## Permamnent storage
- xv6 fs
- the underlying block layer
- `bread()`
- driver
- since the read can take a long time, we go to sleep
- when do we go to sleep
- how do we tell the driver to read a block
- how do we learn that we got a block that has been read

## logging system
- when we write stuff to disk, how do we know that it actually is on disk?
- a little bit about fault tolerence

## Locks
- spinlocks
- sleep locks

## Signals
- delivery of signals





