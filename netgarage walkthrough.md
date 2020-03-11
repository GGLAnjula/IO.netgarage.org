**IO.Netgarage.org Walk-through**

**Introduction**

IO is the most mature game, but is never the less in continually updated as technology develops. It provides recent radare2 and gdb builds.
Level 1
Initially Level 1 username and the password is given in the Io.netgarage.org web site and what we have to do is just open putty on our PC and ssh to the given address.

![001](https://user-images.githubusercontent.com/36528620/76451021-ca92a680-63f4-11ea-9363-b6b52181afe4.png)

Now we are in Level 01. 

In each level there is a directory called **levels** and inside that directory there are multiple files related to each level. At the moment we don’t have any permission to open or execute those files. Among those files Level01 is our target file and it doesn’t have any source code. That means we have to use reverse engineering methods to get into Level02.

Using **_“file”_** command we can identify that this Level01 is an executable file.

Using **_“string”_** command we can get the string outputs from the file Level01.

![level01-01](https://user-images.githubusercontent.com/36528620/76449494-31629080-63f2-11ea-9d10-f82467e1b704.PNG)


We can reveal that program will be asking for a 3-digit passcode.

Let’s try to run a program under **_gdb”_** and disassemble the main function.


![level01-02](https://user-images.githubusercontent.com/36528620/76451361-4d1b6600-63f5-11ea-937c-44abfac97b94.PNG)

In there we can see that the program compares a fixed value with the value of register **_“eax”_**
Fixed value is 0x10f and by looking at that we can assume this is a hexadecimal value.
Let’s convert it into decimal value using **_“p”_** command in gdb.
It gives value as 271 and we can try it with **_level01_** file.
 
![level01-03](https://user-images.githubusercontent.com/36528620/76451461-70deac00-63f5-11ea-9307-a2a5ba7853d0.PNG)


After that step we are free to view **_“. pass”_** hidden file inside the level2 directory which is located inside the home directory. Using **_“cat”_** command we can retrieve the password for level2.


**Level 2**

Now we are in Level2 and inside the **_“levels”_** directory there is an executable file called level02. Once we execute that file, it will display that **_“source code is available in level02.c”_**.

 Let’s view the above source code (level02.c).

![level02-01](https://user-images.githubusercontent.com/36528620/76451582-a1264a80-63f5-11ea-9b17-5f66cb7bc343.PNG)

By reading the source code of level02.c file we can say that,
1.	The number of args must be 2, (argv [0] being the caller's name).
2.	The two arguments should be numbers
3.	The catcher function will be called on the event SIGFPE (launched for example for a division by 0)
4.	The return value of the function is argv [1]/argv [2]
 
 We need to execute **_“catcher”_** function instead of **_“SIGFPE”_**. For that we need to use an integer value outside of the bound of the integer definition. So most-negative value to be out of range is -2147483648 (sqrt ( -2147483648) = 2147483648). Now we are sending the value – 2147483648 to the abs, the result will be also -2147483648 (because of binary max bound and negative values).
 
 ![level02-02](https://user-images.githubusercontent.com/36528620/76451709-d92d8d80-63f5-11ea-85e9-e8f83667e5f3.PNG)
 
Same as the previous level just **_cat_** the **.pass** file inside the **level3** directory which is located inside the **home** directory.

Now we have a password for level3.

**Level 3**

In here as well we have to view the source code of **_level03.c_** file.

![level03-01](https://user-images.githubusercontent.com/36528620/76451855-198d0b80-63f6-11ea-8030-1c5dc585a088.PNG)

Simply we can see that we need to execute the **_good_** function instead of **bad**. Mean is this program introduce the concept of buffer overflow to overwrite nearby local variable.
Stack Hierarchy. 

![level03-07](https://user-images.githubusercontent.com/36528620/76452463-2e1dd380-63f7-11ea-82ca-f9c9c614f06f.png)

  
The variable **_functionpointer_** is store below the variable **_buffer_**.

And there is a command called **_memcpy_**. After referring the man page of memcpy we can see that did not restrict length of argv [1] to be copied to buffer. Hence, by copying more than what the buffer [50] can hold, we start to overwrite the memory below it. This is known as **_buffer overflow_**.

**Step 01**

Disass main

![step 01-1](https://user-images.githubusercontent.com/36528620/76459682-a4740300-6402-11ea-9415-1cb83796f9c9.PNG)

By referring the source code, we know that **_printf_** function will have a string as argument1 and **_functionpointer_** as argument2.

**DWORD PTR [esp], 0x80486c0** means that esp+0x4 address will assigning the value of 0x80486c0.

(DWORD= 4bytes).

Let’s see what is included in memory address 0x80486c0.

![step 01-2](https://user-images.githubusercontent.com/36528620/76459697-a8a02080-6402-11ea-976f-4875bd809a65.PNG)

Now we can confirm that argument1 definitely contain the string. That means **_functionpointer_** is located at **_esp+0x4_**.
   
  ![step 01-3](https://user-images.githubusercontent.com/36528620/76459716-adfd6b00-6402-11ea-9353-2b265b49f9f8.PNG)

From this we know that **_functionpointer_** is at ebp-oxc.

**Step 02**

Finding the buffer

Same as previous step we can do the same to find the buffer.

 ![step 02](https://user-images.githubusercontent.com/36528620/76459917-13e9f280-6403-11ea-91da-508cfd0609e0.PNG)

Buffer is at ebp-0x58

**Step 03**

Finding the actual buffer size

For this we need to find the distance between **_buffer_** and **functionpointer**.

**0x58-0xc
**76

So actual size of buffer is 76 bytes.

**Step 04**

Finding the address of **good**

Using **_print_** command, I’m going to find the address of function **_good_**.  

![level03-02](https://user-images.githubusercontent.com/36528620/76457519-f450cb00-63fe-11ea-9ac6-79f6b41b75d7.PNG)

Now we know the address of good function and we can launch a buffer overflow attack.

![level03-05](https://user-images.githubusercontent.com/36528620/76455844-2c0a4380-63fc-11ea-9507-1bd3357bc470.PNG)

**Step 05**

Read the password from **_.pass file_**.

![level03-06](https://user-images.githubusercontent.com/36528620/76457318-9328f780-63fe-11ea-912e-df3c988f0d29.PNG)


**Level 4**

Same as the previous levels we can look at the source code of the **_level04.c_** file for get an idea about the level04.

![level04-04](https://user-images.githubusercontent.com/36528620/76458126-1139ce00-6400-11ea-9cd4-407574000fc1.png)

By looking at the deep of the above result we can see that **whoami** file has been called without using an absolute path.

The OS will look at $PATH for list of directories to search the binary for in a left to right order. When it finds a binary that has the requested name, it will execute that binary.

![level04-03](https://user-images.githubusercontent.com/36528620/76458013-d768c780-63ff-11ea-9045-a15d480efa5d.png)


So we can run our own **whoami** program that spawn a shell. As long as our directory comes before the actual location of **whoami**, our binary will be run instead.

Let’s create our own **whoami** program.

![level04-01](https://user-images.githubusercontent.com/36528620/76457709-498cdc80-63ff-11ea-94ae-c9771bb7c093.PNG)


Now make a directory called **new** and put above **whoami** file inside that new directory.
Edit **_PATH_** variable as our new directory.
 
![level04-02](https://user-images.githubusercontent.com/36528620/76457765-65907e00-63ff-11ea-84b1-e13f271d9c43.PNG)

Compile the program with the same name as **whoami**.
Finally run the **evel04** file inside the levels directory for gaining the password for level05.



