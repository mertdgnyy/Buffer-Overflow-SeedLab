# Buffer-Overflow-SeedLab
### Ali Mert Doganay / CS458								
### B00773044			
	
First of all, to perform Buffer Overflow attack, I disabled the countermeasures. Address Space Randomization is one of the countermeasures to randomize the starting address of heap and stack. I learned that, this makes guessing the precise address location difficult which is the essential part of the Buffer Overflow attack. I simply disabled it by using the command in Figure 1:

![Picture1](https://user-images.githubusercontent.com/54585515/168015871-cb0abad4-fced-432b-988a-51485a85de95.png)

*Figure 1: “Disabling Countermeasure: Address Space Randomization”*

The second countermeasure is the /bin/dash which is a dash shell that prevents itself from being executed in a Set-UID process. In our case, since we use Ubuntu OS, it is pointed as /bin/sh. /bin/sh can drop the user privilige from effective userID (root) to process’s real user ID. Therefore, with using Set-UID in Ubuntu OS, we do not want to drop our privilige, so I linked it with the /bin/zsh which does not have such countermeasure (Figure 2).

![2](https://user-images.githubusercontent.com/54585515/168015987-202a81f1-058a-4f53-89f7-1bd1059923a0.png)

*Figure 2: “Linking /bin/sh to /bin/zsh to eliminate countermeasure”*

Also the final countermeasures are the using “-z execstack” and “-fno-stack-protector”. Execstack is essential for us to declaring the stack of running problem executable or non-executable. By default, it is declared as non-executable, therefore adding “-z execstack” is vital to be executed from the stack. Also “-fno-stack-protector” is used for implementing Buffer Overflow since the security mechanism called Stack-Guard prevents Buffer Overflows and this command enables us to disable this feature.
### Task 1: Getting Familiar with Shellcode
When I compiled the c file “call_shellcode.c” with typing “make” I created executable c files named a32.out and a64.out.

A shellcode is the code to launch a shell . “a32.out” and “a64.out are code files that will open up shell if it’s executed. I understood that I need to inject malicious code in the following part using shell since shell is used for code injection attacks that targets program’s privilige. I executed both a32.out and a64.out executables, and to understand the user priviliges, I also typed “id” in the shell and saw that our uid (userID) assigned as 1000 which is user seed and we do not have euid. That means our shell ran successfully, but we do not have the root privilige, It can be also understood from the “$” sign (Figure 3).

![3](https://user-images.githubusercontent.com/54585515/168015998-e3a173a0-5d67-43f2-a0c6-630d76118703.png)

*Figure 3: “compling call_shellcode.c /executing call_shellcode”*

### Task 2: Understanding the Vulnerable Program
In order to compile the code, I simply typed “make” (Figure 5). In addition to launch the shell in root mode, I learned that we need to change the mode of the program to 4755 using chmod for enabling setUD bit (figure 4) and also chown mode to root for changing the ownership, this way we get root-owned Set-UID program. 4755 comes from setud bit and owner,group and other user’s priviliges. In our case, “Makefile” provided these features for us inside of itself.

 
![4](https://user-images.githubusercontent.com/54585515/168016000-c80ccdae-1bc1-44a0-b1b9-dd3bbf513c87.png)

*Figure 4:” setting chmod to 4755”*

It can be seen from the Figure 6 that our compiled program which is stored in stack is highlighed as red that means it made a root-owned SET-UID program, where green colors are the executable files.

![5](https://user-images.githubusercontent.com/54585515/168016011-65efdd35-37ab-428b-8956-f2fb3a8ff6c2.png)

*Figure 5: “compling stack.c”*

![6](https://user-images.githubusercontent.com/54585515/168016261-a546fe1d-65ef-4c28-97da-ff3e4ddf4706.png)

*Figure 6: “storing the compiled program in the stack”*

As seen in the Makefile, our values of L1 through L4 are represented as L1 = 100, L2 = 160, L3 = 200, L4= 10 (Figure 7). These values represent the BUF_SIZE in the stack code.

 ![7](https://user-images.githubusercontent.com/54585515/168016272-711f6d2a-5db6-48dd-8af1-d114e65f93ad.png)
 
*Figure 7: “L values in Makefile”*

Before building the “exploit.py”, firstly I want to talk about the content of stack.c and why it has a Buffer Overflow problem.
 
![8](https://user-images.githubusercontent.com/54585515/168016285-89fbe095-d120-48d1-a537-602e4c12702f.png)

![9](https://user-images.githubusercontent.com/54585515/168016301-2f118b5a-866d-4e35-a4f4-6cbc629b7fb8.png)

*Figure 8: “stack.c”*

As seen in the Figure 8, in line 27, “str” is the buffer to store data and it is given as 517 characters. After calling the “dummy_fuction(str)” in line 37, function “dummy_fuction()” is called, which is in the line 45, that calls the function “bof()” with the given argument “str” in the line 49. 

bof is the main function that has a Buffer Overflow problem. Because, in line 20, “strcpy” copies the “str” into the “buffer”, and the “buffer” has a value of “BUF_SIZE”. Therefore, if given str value exceeds the buffer, Buffer Overflow will occur.

For testing the program with 4 different L values (BUF_SIZE), before compling it, I created a file named badfile for the “str” input of the stack.c. I randomly typed a string, in this case, my name  “Ali Mert Doganay ” which has 18 input size (Figure 9). I ran the 4 programs and as shown in Figure 10, except the stack-L4, all of them returned properly. The reason for giving an error is my “str” input which comes from badfile has a input value of 18, but as shown in the figure 7, L4 which is the BUFF_SIZE value is given as 10, in this case str value exceeds the buffer and gives segmentation fault.

![10](https://user-images.githubusercontent.com/54585515/168016308-b427bc14-38e7-4ca0-9a3c-ddd16be63181.png)

*Figure 9: “inside of the badfile”*

![11](https://user-images.githubusercontent.com/54585515/168016313-d106523f-a776-4884-a506-ab043e482870.png)

*Figure 10: “results of the stack-L1, stack-L2, stack-L3, and stack-L4”*

### Task 3: Launching Attack on 32-bit Program

In order to use this vulnerability and gain access to the root shell with successful attack, we need to find the address of the running program in the memory. To do that, as I mentioned previously, I disabled the Address Space Randomization for storing the process in the same memory in the stack. 

First of all I created a new empty badfile. I used gdb for finding offset and where the return address is stored in the stack to create the “badfile” with correct buffer payload using the exploit.py. In figure 11, First of all, I set a breakpoint on the function “bof” using the command “b bof” and run the program. Then, I found the frame pointer (ebp) value as “0xffffcb18” with the command “p $ebp”. Therefore return address will be in the “0xffffcb18 + 4” by the layout of the stack. After, I typed “p &buffer” to find buffer’s starting address which comes “0xffffcaac”. To find distance of return address and buffer’s starting point, firstly, I used  “p/d ebp – buffer” command in gdb , which in my case it is 108. Finally, I add 4 bytes to 108 and found that offset is 112. 

![12](https://user-images.githubusercontent.com/54585515/168016325-e3063a1c-a377-4424-9427-c03d8b897c0c.png)

*Figure 11: “Using gdb and finding ebp and buffer address”*

Before modifying the exploit.py, it is needed to know that when we run a program inside the gdb, stack location changes and they are not the same as we run the program without the gdb. When running the program using gdb, stack location will be little deeper than we run the program without gdb. Therefore, to modify exploit.py, I cannot use the exact return value from gdb that I mentioned previously, I need to give a larger number for it.

Figure 12 shows the initial content of the exploit.py without modifying it. 

![13](https://user-images.githubusercontent.com/54585515/168016333-edd9572d-2df8-4ef7-adef-157731fd2d69.png)

*Figure 12: “Initial exploit.py”*

My modifications is in Figure 13 that first of all, I replaced the 32 bit shellcode from the previous task because we are attacking on 32-bit program. I changed the value “ret” to my “ebp address + x value”since I ran the program without gdb and I need to give higher value. In my case I gave the value as “0xffffcb18 + 140” which is 0xffffcba4 in hex format that I will explain in the following part. For my offset, I put the “ebp – buffer’s starting address + 4” as I mentioned previously, and it is 112. Finally I changed the start value to “517 – len(shellcode)” for putting the shellcode at the end. 

![14](https://user-images.githubusercontent.com/54585515/168016346-e50c24a0-b889-4eb1-af62-2ef160751c88.png)

*Figure 13: “Last version of the exploit.py”*

I overwritten the ret value with ebp + 140 which is 0xffffcb18 + 140. Because as shown in the figure 14, I have 112 bytes of offset that I calculated previously. From the content array in exploit.py in Figure 13, we need to put the new overwritten address within the 4 byte. Also, when I print out the shellcode I found that it is 27 bytes. When I sum of them, I found 143. Since badfile has 517 bytes, rest of the 374 bytes should be of NOP instructions. My ebp value is 0xffffcb18, and I have 374 + 4 bytes before starting of the shellcode address. Since 378 byte equals to 0x17A in hex format. I added ebp with 0x17A and it gives the address 0xffffcc92 which should be the starting address of shellcode. Therefore, choosing a value between ebp and starting address of shellcode is enough for to determine the address of the new ret value. I chose value 140 and it is between them, so it will not be a problem.

![15](https://user-images.githubusercontent.com/54585515/168016351-b706bef1-569b-4c0c-a9da-0e2ab623effa.png)

*Figure 14: “why I chose new ret value as ebp + 140”*

When I modified the exploit.py, I started my attack as figure 15. I create my badfile with executing exploit.py and, I successfully perform buffer overflow attack and gain root access with indicating that euid is zero. 

![16](https://user-images.githubusercontent.com/54585515/168016357-1963e2dc-8630-4fc0-b450-8971364781b7.png)
 
*Figure 15: “Successful attack”*

### Comment for the project = For the project3, I did the Task 1 through Task3. I spent approximately 2.5-3 days for completing the tasks, therefore I could not attempt to finish the rest of the tasks, but I will try to finish the tasks 8 and 9 for the final exam. 
