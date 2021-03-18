# Chapter 1 Introduction

## 1.1 What Operating Systems Do

* controls the hardware
* coordinates its use among the various application programs

#### System View

* work as a **resource allocator** intimately involved with the hardware
  * decide how to **allocate** resources to programs and users **efficiently and fairly**
* work as a **control program**
  * manages the execution of user programs to prevent errors and improper use of computer
  * especially concerned with operation and control of I/O device

**Common Definition**

* operating system is the one program running at all time on the computer -&gt; **kernel**
* **kernel**
  * system programs
    * associate with operating system, but not necessarily part of the kernel
  * application programs

## 1.2 Computer-System Organization

### computer system operation

* accessing shared memory

  * CPUs and device controllers connected through a common bus that provides access to shared memory
  * CPU and device controller can execute in parallel \(competing for memory cycles\)
  * to ensure orderly access -- memory controller synchronizes access to the memory

* bootstrap program
  * initial program to run
  * stored within ROM or EEPROM \(firmware\)
  * initialize all aspect of the system
    * from CPU register to device controllers to memory contents
  * **must locate the operating system kernel and load it into memory to start executing the system**

    \*\*\*\*
* system process \(system daemons\)

  * after kernel is loaded and executing, some services are provided outside of kernel --&gt; system process
  * system process are loaded into memory at boot time & runs entire time the kernel is running

* interrupt \(hardware & software\)
  * Hardware interrupt
    * triggered by sending signal to the CPU \(by system bus\)
  * Software interrupt
    * triggered by **system call** \(monitor call\)
  * interrupt vector
    * stored in low memory
    * array of address indexed by a unique device number, given with the interrupt request
    * array of pointers to interrupt routines that are predefined
    * provide the address of the interrupt service routine for the interrupting device
    * interrupt routine is called indirectly through this table
  * when the cpu is interrupted....
    * stop what it is doing & immediately transfer execution to a fixed location
      * fixed location -- contains start address where service routine for interrupt
    * interrupt service routine executes and completed -&gt; CPU resumed previous interrupted computation

### storage structure





