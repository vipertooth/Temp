# Review
I find that both *exploit_nss.py* and *exploit_defaults_mailer.py* do not do anything malicious.  There is a chance that a vulnerable system has important memory overwritten. This is unlikely however because these exploits target the environment variable portion of memory.  

## Artifact Review
 The exploits create a total of 5 artifacts on the system.  4 out of the five are static therefore sha1sum has already been generated.
 
### exploit_nss.py 
1. The function *create_libx()* will create 2 artifacts:
	- Folder in working directory = "libnss_X"
	- File in libnss_X folder = "X1234.so.2"
	- Sha1sum = "4ce284e127fdedb31dabd999eaf8633aa1106ef1  X1234.so.2"
	- Simple /bin/sh library

There is no cleanup in code to remove artifacts. 

### exploit_defaults_mailer.py 
1. Function *create_bin* creates 1 artifact:
	- File "sshell" in /tmp/
	- Sha1sum = "f5812a962792c4af8dee4d2e38f7db322d620604  sshell"
	- Simple /bin/sh binary

2. Function *create_shell* creates 1 artifact:
	- File "gg" in /tmp/
	- Sha1sum = "e9cf74f05d8f33f669700f6f157a51e8d484bf1f  gg"
	- Simple script to make "sshell" root.root and 4755 (SUID)

3. Exploit trigger will run gg which creates 1 artifact: 
	-  File "pwned" in /tmp/
	-  This will just be the output of the id command

There is no cleanup in code to remove artifacts.

## Binary Analysis 
Two files, one for each exploit script, were analyzed to verify content.  Both files were found to be almost the same. After the files were unzipped xxd was used to get a quick look at what the program contains.

[xxd image here]

A quick look at the hex shows that the program is most likely a simple execute /bin/sh. Running strace against both files confirms 

[strace image here]

Strace confirms that the 2 static binaries just set uid/guid 0 then execute /bin/sh.  The following is an example of what the program looks like before compiling. 

```c
int main() {
        setuid(0);
		setguid(0);
        execve("/bin/sh");
}
```
