# Bandit0:

```
ssh bandit0@bandit.labs.overthewire.org -p 2220
password: bandit0
ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
```


# bandit1:
```
ssh bandit1@bandit.labs.overthewire.org -p 2220
password: ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
flag: 263JGJPfgU6LtdEvgfWU1XP5yac29mFx

```
# bandit2:
```
ssh bandit2@bandit.labs.overthewire.org -p 2220
password: 263JGJPfgU6LtdEvgfWU1XP5yac29mFx
flag: MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

```
# bandit3:

```
ssh bandit3@bandit.labs.overthewire.org -p 2220
password: MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx
flag: 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
```

# bandit4:

```
ssh bandit4@bandit.labs.overthewire.org -p 2220
password:2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
flag:4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
```

# bandit5:

```
ssh bandit5@bandit.labs.overthewire.org -p 2220
password:4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
flag:HWasnPhtq9AVKe0dmk45nxy20cvUa6EG

```

# bandit6:

```
ssh bandit6@bandit.labs.overthewire.org -p 2220
password:HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
flag:morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj

```


# bandit7:

```
ssh bandit7@bandit.labs.overthewire.org -p 2220
password:morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj
flag:dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc

```


# bandit8:

```
ssh bandit8@bandit.labs.overthewire.org -p 2220
password:dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
flag:4CKMh1JI91bUIZZPXDqGanal4xvAg0JM

```


# bandit9:

```
ssh bandit9@bandit.labs.overthewire.org -p 2220
password:4CKMh1JI91bUIZZPXDqGanal4xvAg0JM
flag:FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey

```


# bandit10:

```
ssh bandit10@bandit.labs.overthewire.org -p 2220
password:FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey
flag:dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr

```

# bandit11:

```
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
ssh bandit11@bandit.labs.overthewire.org -p 2220
password:dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
flag:7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4

```

# bandit12:

```
mktemp -d -> creates a temporary directory in the /temp and returns its path
xxd -r -> reversts the hexdump back in to binary file
ssh bandit12@bandit.labs.overthewire.org -p 2220
password:7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4
flag:FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn

```

# bandit13:

```
scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .
ssh bandit13@bandit.labs.overthewire.org -p 2220
password:FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
flag: downladed the sshkey.private

```

# bandit14

```
The bandit was logged using the:
chmod 700 sskey.private
ssh -i sshkey.private bandit13@bandit.labs.overthewire.org -p 2220
flag:MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
```

# bandit 15

```
ssh bandit14@bandit.labs.overthewire.org -p 2220
openssl s_client -connect localhost:30001
password:8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo

```

# bandit16
```
ssh bandit16@bandit.labs.overthewire.org -p 2220
password:kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
openssl s_client -connect localhost:31790 -quiet

```

# bandit17

```
using the sshkey17.private file.
flag: x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO

```

# bandit18

```
assh bandit18@bandit.labs.overthewire.org -p 2220
password:x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO

```

# bandit19

```
ssh bandit19@bandit.labs.overthewire.org -p 2220
password:cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8
setuid and setgid executables can used to escalate the prevalges by piping it with regular commands:
./bandit20-do id | cat /etc/bandit_pass/bandit19
```

# bandit20
```
ssh bandit20@bandit.labs.overthewire.org -p 2220
password:0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
bg -> takes the process id and sends the job to the background
fg -> takes the process id and sends the job to the fore ground
jobs -> lists all the runing jobs
& -> could be used to send the job to the background
echo -n "0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO" | nc -l -p 6969 &
```

# bandit21

```
ssh bandit21@bandit.labs.overthewire.org -p 2220
password:EeoULMCra2q0dSkYj561DX7s1CpBuOBt
```

# bandit22

```
ssh bandit22@bandit.labs.overthewire.org -p 2220
password: tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q
```

# bandit23

```
ssh bandit23@bandit.labs.overthewire.org -p 2220
password:0Zf11ioIjMVN551jX3CmStKLYqjk54Ga
A script placed in crontab executes all the shell scripts present in /var/spool/bandit24/foo directory.
make temp directory with mktemp -d 
place your script in the file and cp it to the /var/spool/bandit24/foo
wait for a sec and you will get the result
The script:
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/tmp.tdfCBhJ8VH/pass

```

# bandit24 

```
ssh bandit24@bandit.labs.overthewire.org -p 2220
password:gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8

```
# bandit25

```
ssh bandit25@bandit.labs.overthewire.org -p 2220
password:iCi86ttT4KSNe1armKiwbQNmB3YJP3q4

```

# bandit26

```
login using the private key
password:s0773xxkk0MXfdqOfPRVr9L3jJBUOgCZ
vimscript can be used to bypass the shell by doing this step:
shell=/bin/bash
and then running the shell the command mode
```

# bandit27
```
ssh bandit27@bandit.labs.overthewire.org -p 2220
password:upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB

```

# bandit28
```
ssh bandit28@bandit.labs.overthewire.org -p 2220
password: Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN
git logs -> show the commits
git show <commit id> -> prints out all the data in the commit
```

# bandit29
```
password: 4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7
ssh bandit29@bandit.labs.overthewire.org -p 2220
to check all the branches in git: git branch -a
To switch to other branch: git checkout <branch name>
```

# bandit30

```
ssh bandit30@bandit.labs.overthewire.org -p 2220
password: qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL
git tag -> it is like a bookmark on the commit

```

# bandit31

```
ssh bandit31@bandit.labs.overthewire.org -p 2220
password:fb5S2xb7bRyFmAvQYQGEqsbhVyJqhnDy

```

# bandit 32
```
ssh bandit32@bandit.labs.overthewire.org -p 2220
password: 3O9RfhqyAlVBEZpVb6LYStshZoqoSx5K
$0 -> runs the un interactive shell because it holds the path to the shell $0=/bin/bash
```

# bandit33
```
ssh bandit33@bandit.labs.overthewire.org -p 2220
password: tQdtbs5D5i2vJwkO8mEyYEyTL8izoeJ0

```
