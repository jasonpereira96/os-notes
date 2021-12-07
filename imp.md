## boot
- boot - how does it work?
- what is a bootloader and its relationship to BIOS
- how do we start running?
- What is happening in the execution from the entry point in the kernel all the way until you start running the first user program init 
- configure the har- dware in various ways, in particlar the interrupt tables 
- initial page table- 
- once we've started running init -  we have another page table
- the free list of free pages

## Memory mamagesment
- Structure and function of page tables
- The fact that we have one for every process
- mmap homework
- `sbrk()`
- malloc is in userspace

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
### Delivery of signals
- error case - if during the handling of a syscall, an interrupt happens, then we have to unwind the stack of the syscall and return an error
- we need to wake up a process if it is sleeping, since we need to deliver the signal immeidiately


## Log 
begin_op();
log_write() is a proxy for bwrite()

in log_write()
log.lh.block[i] = b->blockno; // add the modified buffer to the in memory log
// this happens several times

end_op(); // calls commit()


commit();
```c
void commit
{
  if (log.lh.n > 0) {
    write_log();     // Write modified blocks from cache to log
    write_head();    // Write header to disk -- the real commit
    install_trans(); // Now install writes to home locations
    log.lh.n = 0;
    write_head();    // Erase the transaction from the log
  }
}
```

There are 2 file tables
The global file table ftable which has all the files and
The proc->ofile which has a list of the processes open files
The syscall open() populates the global one first and then the per process one


inode  numbers  only  have  a  unique  meaning  on  a  single  disk


`link()` creates a directory entry
`unlink()` removes a directory entry

## questions
- `iderw()` working ???? 
- multi cpu spin locks

# Notes

For XCHG
https://pdos.csail.mit.edu/6.828/2005/readings/i386/LOCK.htm
The LOCK prefix causes the LOCK# signal of the 80386 to be asserted during execution of the instruction that follows it. In a multiprocessor environment, this signal can be used to ensure that the 80386 has exclusive use of any shared memory while LOCK# is asserted.

You can never switch to another process directly
You have to switch to the scheduler first and then the scheduler will switch to another process
The xv6 scheduler has its own thread (saved registers and stack) (one per CPU) because it is sometimes not
safe for it execute on any process’s kernel stack


In  our  example, sched called swtch to  switch  to cpu->scheduler,  the  per-CPU
scheduler  context. That  context  had  been  saved  by scheduler’s  call  to swtch (2781).
When  the swtch we  have  been  tracing  returns,  it  returns  not  to sched but  to sched-
uler,  and  its  stack  pointer  points  at  the  current  CPU’s  scheduler  stack

### sleep() and wakeup()
- sleep goes to sleep by setting proc->state to SLEEPING and calling `sched()`. This calls the schduler and makes it switch to something else.

- wakeup(chan) blows through the table and looks for any process which slept on channel chan. It then sets its state to RUNNABLE so that it can eventually be scheduled by the scheduler.

## Pipe()
`pipe()` uses 

## Boot
The  BIOS  loads  the  boot  sector  at  memory address 0x7c00 and  then  jumps  (sets  the  processor’s %ip)  to  that  address.

## `entry.S`
- Here entry sets up 2 page tables
- Initially, the kernel is running at low physical addresses.
- We need to map it to high virtual addresses

### Steps
- Map 0:x400000 to 0:x400000
- Map KERNBASE:KERNBASE + x400000 to 0:x400000
- do some other stuff
- Enable Paging
- JMP to `main()` (Since paging is enabled and page tables are set up, virtual addresses work and the kernel doesn't crash)

Each  process  has  two  stacks:  a  user
stack  and  a  kernel  stack  (p->kstack).  When  the  process  is  executing  user  instructions,
only  its  user  stack  is  in  use, and  its  kernel  stack  is  empty. When  the  process  enters  the
kernel  (for  a  system  call  or  interrupt),  the  kernel  code  executes  on  the  process’s  kernel
stack;  while  a  process  is  in  the  kernel,  its  user  stack  still  contains  saved  data,  but  isn’t
actively  used. A  process’s  thread  alternates  between  actively  using  its  user  stack  and  its
kernel  stack.  The  kernel  stack  is  separate  (and  protected  from  user  code)  so  that  the
kernel  can  execute  even  if  a  process  has  wrecked  its  user  stack.

### Note:
Changing  page  tables while  executing  in  the  kernel  works  because setupkvm causes  all  processes’ page  tables
to  have  identical  mappings  for  kernel  code  and  data. **This is why we add all kernel mappings to all the user processes page tables, so that the kernel doesn't crash when we switch page tables in kernel code. Fuck!**

**This is how the kernel stack is automatically used when executing interrpt handlers**

`switchuvm()` also  sets  up  a  task state  segment SEG_TSS that  instructs  the  hardware  to  execute  system  calls  and  interrupts on the process’s kernel stack.


### Calls trace
- running process 1
- trap() (interrupt handler was just called)
- yield()
- sched()
- switch() to scheduler
- returns in scheduler()
- switch() to another process's kernel thread
- returns in yield() 
- falls through to trap()
- resumes running user code (process 2)

### TSS
We set this to the top of the stack of the kstack, so that when a syscall/ interrpt happens, the CPU automatically starts using the kstack
The three privilege-level stack pairs (SS0:ESP0, SS1:ESP1, SS2:ESP2);
Read during an inter-level CALL or INT to establish a new stack.
