# **CSCI 4250/6250 – Computer Security. Homework #2: Return-Oriented Programming**

Due date: 11:59pm, 2/27/2019


## **NOTICE**

I posted my rop script in class (rop-inclass.py) for your reference. You can directly execute it to get a shell.
You are not required to use this because it depends on pwntools.
You can craft your script based on homework description below, which is cleaner (see rop.py for example).
It can be invoked by ``./your_program `python ./rop.py` ``
Note that my ROP may not work on every VM. You might need to re-generate the ROP payload and replace mine with yours.

## **Description**

In this project, you will produce return-oriented programming (ROP) attacks. You will use ROPGadget tool (https://github.com/JonathanSalwan/ROPgadget) to extract gadgets from the target binary and library. Then you will compose exploits that you can launch ROP attacks to the target program. 


Your target is still the vulnerable `ls` program, which I believe you are all very familiar with. You can reuse your previous scripts. Your task is to spawn a shell using ROP.

**\*\* Use your VM for this project \*\***



## Step 1: Copy Target Binary

In this project, each student has his/her own vulnerable binaries. You can get your binaries from github:  
`$ git clone git@github.com:UGASecurityClass/hw2.git` if you have added your SSH key, or

`$ git clone https://github.com/UGASecurityClass/hw2.git` then

`$ cd hw2`

Your target binaries is `./bin/proj1_dep_username`. You also need to disable ASLR in this project.

`$ echo 0 | sudo tee /proc/sys/kernel/randomize_va_space`

## Step 2: Execute the shell: call execve(“/bin/sh”, null, null) - 5 points


In this part, you will generate chain of ROP gadgets to launch the shell. You can use “--ropchain” option that ROPGadget tool provides to automatically compose the exploit to execute “execve(“/bin/sh”, null, null)”. Your target binary is a tiny application and you cannot find enough gadgets for the attack. For example, when you execute “ROPgadget --binary ./bin/proj1_dep_username --ropchain”, it will fail. 
In this project, your main target is “libc.so” library that dynamically linked to the process at run-time. You will need to find out the information of libc library file (e.g., file location, base address of the process that libc loaded) and use ROPgadget tool to find ROP chains from the library. 



1.	You can use a debugger (gdb) to find linked libraries and their base address. “info shared” command after the program runs.
2.	You can also read memory layout without gdb: `$ ldd ./bin/proj1_dep_username`
3.	Now you can run `ROPgadget` to extract gadgets from libc library.
    * `$ ROPgadget --binary “libc_library_path” --offset “baseaddr_of_libc” --ropchain`
    * You can copy the python code into a file (e.g., rop.py) and insert padding bytes.
    * You can execute the target program with exploits: ``./bin/proj1_dep_username `python ./rop.py` ``
    * If it fails, use gdb to find out the reason and fix it. Otherwise, you should be able to get a shell.

## Bonus: - 5 points!!!

Again we have a bonus part. I believe copying the result of ROPGadget is not really a challenge for most of your. In this bonus part, you will do a serious exploit. Remember the shellcode in the in-class quiz that invokes 3 system calls to get a root shell? Now, you will implement it in a ROP fashion. 

Concretely, you are going to do the following:
1. Invoke system call #0x31 (`geteuid`): The result is that you have the uid of root in eax.
2. Invoke system call #0x46 (`setreuid`): Assign the value of eax to ebx and ecx to construct the parameters for `setreuid`. Then invoke `setreuid`.
3. Invoke system call #0xb (execve)

Looks easy? Not really! The bonus for this exercise is **5 points**!!!


## Submission

1. Describe your experience (including screen shots and commands) and explain all steps you have done.
2.	Submit a report (accept only PDF format) and ROP chain code (python) through eLC.

## References

1. Return-to-libc: http://blog.fkraiem.org/2013/10/26/return-to-libc/
2. ROPgadget: http://shell-storm.org/project/ROPgadget/
3. GDB cheatsheet: http://darkdust.net/files/GDB%20Cheat%20Sheet.pdf
4. For bonus part, you will find in-class quiz useful.


