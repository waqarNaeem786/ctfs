# fd -> file descriptor:
 - They are the permission for input, output and error in the linux file system the functions like read(), write() use them as the
   first argument then the variable to which the input or output will be stored as the second argument and is the length of the 
   stream of input and output
    - 0 -> is used for user input
	- 1 -> is used for user output
	- 2 -> is used displaying error
	
 - In the fd challenge of the pwnable kr the c code is provided to us which takes the user input using the read() 
 - The read() function is used to take the user input.
 - For the user input taking we use the 0 file descriptor.
 - The challenge here was that we have to pass the file descriptor as the user argument.
 - The argument is then subtracted from the 0x1234 which in decimal is 4660 
 - So when the user subracts the 0 from the 4660 it will become -4660, which is not applicable.
 - In order to return the 0 as the value we will subtract the 4660 from the 4660 which will return the 0 file descriptor.
 - At last flag will be ours.
   
 ```
fd@ubuntu:~$ echo "LETMEWIN" | ./fd 4660
good job :)
Mama! Now_I_understand_what_file_descriptors_are! 
 ```
