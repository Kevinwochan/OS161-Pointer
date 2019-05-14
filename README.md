# OS161-Pointer

Setup
We assume after ASST0 and ASST1 that you now have some familiarity with setting up for OS/161 development. If you need more detail, refer back to ASST0.

Clone the ASST2 source repository from gitlab.cse.unsw.edu.au. Note: replace XXX with your 3 digit group number.

% cd ~/cs3231
% git clone gitlab@gitlab.cse.unsw.EDU.AU:19t1-comp3231-grpXXX/asst2.git asst2-src
Note: The gitlab repository is shared between you and your partner. You can both push and pull changes to and from the repository to cooperate on the assignment. If you are not familiar with cooperative software development and git you should consider spending a little time familiarising yourself with git.

Building and Testing Your Assignment
Configure OS/161 for Assignment 2
Before proceeding further, configure your new sources.

% cd ~/cs3231/asst2-src
% ./configure
Unlike previous the previous assignment, you will need to build and install the user-level programs that will be run by your kernel in this assignment.

% cd ~/cs3231/asst2-src
% bmake
% bmake install
For your kernel development, again we have provided you with a framework for you to run your solutions for ASST2.

You have to reconfigure your kernel before you can use this framework. The procedure for configuring a kernel is the same as in ASST0 and ASST1, except you will use the ASST2 configuration file:

% cd ~/cs3231/asst2-src/kern/conf
% ./config ASST2
You should now see an ASST2 directory in the compile directory.
Building for ASST2
When you built OS/161 for ASST1, you ran make from compile/ASST1 . In ASST2, you run make from (you guessed it) compile/ASST2.
% cd ../compile/ASST2
% bmake depend
% bmake
% bmake install
If you are told that the compile/ASST2 directory does not exist, make sure you ran config for ASST2.

Command Line Arguments to OS/161
Your solutions to ASST2 will be tested by running OS/161 with command line arguments that correspond to the menu options in the OS/161 boot menu.
IMPORTANT: Please DO NOT change these menu option strings!

Running "asst2"
For this assignment, we have supplied a user-level OS/161 program that you can use for testing. It is called asst2, and its sources live in src/testbin/asst2.
You can test your assignment by typing p /testbin/asst2 at the OS/161 menu prompt. As a shortcut, you can also specify menu arguments on the command line, example: sys161 kernel "p /testbin/asst2". 

Note: If you don't have a sys161.conf file, you can use the one from ASST1.

The simplest way to install it is as follows:

% cd ~/cs3231/root
% wget http://cgi.cse.unsw.edu.au/~cs3231/19T1/assignments/asst1/sys161.conf -O sys161.conf
Running the program produces output similar to the following prior to starting the assignment.

Unknown syscall 55
Unknown syscall 55
Unknown syscall 55
Unknown syscall 55
     :
     :
Unknown syscall 55
Unknown syscall 55
Unknown syscall 3
Fatal user mode trap 4 sig 10 (Address error on load, epc 0x400814, vaddr 0xeeeee00f) 
asst2 produces the following output on a (maybe partially) working assignment.
 
OS/161 kernel [? for menu]: p /testbin/asst2
Operation took 0.000212160 seconds
OS/161 kernel [? for menu]:
**********
* File Tester
**********
* write() works for stdout
**********
* write() works for stderr
**********
* opening new file "test.file"
* open() got fd 3
* writing test string
* wrote 45 bytes
* writing test string again
* wrote 45 bytes
* closing file
**********
* opening old file "test.file"
* open() got fd 3
* reading entire file into buffer
* attempting read of 500 bytes
* read 90 bytes
* attempting read of 410 bytes
* read 0 bytes
* reading complete
* file content okay
**********
* testing lseek
* reading 10 bytes of file into buffer
* attempting read of 10 bytes
* read 10 bytes
* reading complete
* file lseek  okay
* closing file
Unknown syscall 3
Fatal user mode trap 4 sig 10 (Address error on load, epc 0x400814, vaddr 0xeeeee00f)
Note that the final fatal error is expected, and is due to exit() (system call 3) not being implemented by OS/161. If exit()returns to userland (which would not happen in a complete OS implementation), the userland exit library code simply accesses an illegal memory address in order to cause a fault, which subsequently causes the program (and system) to stop. You can distinguish this expected fault from other faults by the address accessed: 0xeeeee00f.

The Assignment Task: File System Calls
Of the full range of system calls that is listed in kern/include/kern/syscall.h, your task is to implement the following file-based system calls: open, read, write, lseek, close, dup2. Note: You are writing the kernel code that implements part of the system call functionality within the kernel. The C stubs that user-level applications call to invoke the system calls are already automatically generated when you build OS/161.

Note that the basic assignment does not involve implementing fork() (that's part of the advanced assignment). However, the design and implementation of your system calls should not assume a single process.

It's crucial that your syscalls handle all error conditions gracefully (i.e., without crashing OS/161.) You should consult the OS/161 man pages (also included in the distribution) and understand fully the system calls that you must implement. Your system calls must return the correct value (in case of success) or error code (in case of failure) as specified in the man pages. Some of the auto-marking scripts rely on the return of error codes, however, we are lenient as to which specific code in the case of potential ambiguity as to the most appropriate error code.

The file userland/include/unistd.h contains the user-level interface definition of the system calls. This interface is different from that of the kernel functions that you will define to implement these calls. You need to design the kernel side of this interface and put it in kern/include/syscall.h. As you discovered (ideally) in Assignment 0, the integer codes for the calls are defined in kern/include/kern/syscall.h. You need to think about a variety of issues associated with implementing system calls. Perhaps, the most obvious one is: can two different user-level processes (or user-level threads, if you choose to implement them) find themselves running a system call at the same time?

Notes on open(), read(), write(), lseek(), close(), and dup2()
For any given process, the first file descriptors (0, 1, and 2) are considered to be standard input (stdin), standard output (stdout), and standard error (stderr) respectively. For this basic assignment, the file descriptors 1 (stdout) and 2 (stderr) must start out attached to the console device ("con:"), 0 (stdin) can be left unattached. You will probably modify runprogram() to achieve this. Your implementation must allow programs to use dup2() to change stdin, stdout, stderr to point elsewhere.

Although these system calls may seem to be tied to the filesystem, in fact, these system calls are really about manipulation of file descriptors, or process-specific filesystem state. A large part of this assignment is designing and implementing a system to track this state. Some of this information (such as the cwd) is specific only to the process, but others (such as offset) is specific to the process and file descriptor. Don't rush this design. Think carefully about the state you need to maintain, how to organise it, and when and how it has to change.

While this assignment requires you to implement file-system-related system calls, you actually have to write virtually no low-level file system code in this assignment. You will use the existing VFS layer to do most of the work. Your job is to construct the subsystem that implements the interface expected by userland programs by invoking the appropriate VFS and vnode operations.

While you are not restricted to only modifying these files, please place most of your implementation in the following files: function prototypes and data types for your file subsystem in kern/include/file.h, and the function implementations and variable instantiations in kern/syscall/file.c.

A note on errors and error handling of system calls
The man pages in the OS/161 distribution contain a description of the error return values that you must return. If there are conditions that can happen that are not listed in the man page, return the most appropriate error code from kern/include/kern/errno.h. If none seem particularly appropriate, consider adding a new one. If you're adding an error code for a condition for which UNIX has a standard error code symbol, use the same symbol if possible. If not, feel free to make up your own, but note that error codes should always begin with E, should not be EOF, etc. Consult UNIX man pages to learn about error codes. Note that if you add an error code to kern/include/kern/errno.h you need to add a corresponding error message to the file user/lib/libc/string/strerror.c.
