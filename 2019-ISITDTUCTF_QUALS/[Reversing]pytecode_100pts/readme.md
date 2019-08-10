## `ISITDTUCTF 2019 Pytecode`

__ctf__: `ISITDTU` 

__challenge__: `Pytecode`

__category__: `Reversing`

__points__: `100`

__solver__: `ordbl0ke`

__description__: _blank_

---

### Write-up

We were given a file without any description. Simply running `file` utility and `cat`-ting revealed we are dealing with (as the challenge name suggests) dissassembly of python byte code:

```
C0rr3ct func:
  6           0 LOAD_CONST               1 ('Wow!!!You so best^_^')
              3 PRINT_ITEM          
              4 PRINT_NEWLINE       
              5 LOAD_CONST               0 (None)
              8 RETURN_VALUE        

Ch3cking func:
  8           0 LOAD_CONST               1 (0)
              3 STORE_FAST               1 (check)

  9           6 LOAD_GLOBAL              0 (ord)
              9 LOAD_FAST                0 (flag)
             12 LOAD_CONST               1 (0)
             15 BINARY_SUBSCR       
             16 CALL_FUNCTION            1
             19 LOAD_CONST               2 (52)
...
```

This is clearly and output of Python's `dis` module. I couldn't find a tool that transforms this disassembly to python source code. There are tools like [uncompyle](https://pypi.org/project/uncompyle6/), which make it easy to transform bytecode to source code and thus make the reversing much simpler, but not for thi disassembly version. 

Therefore, the only option left was to learn how to read these instruction (to be fair, the necessities for this challenge weren't too difficult). 

This 'language' can be read as accumulator-oriented machine's assembly. It also has some global and local fields for variables, filled with values that are indexed by the instructions.  `LOAD_`instructions place stuff on stack, `STORE_` instructions places stuff from stack to these memory fields. Operators operate on operands on stack, functions are being called depending on what's on stack. 

The most important thing to understand is this chunk of code:

```
 6 LOAD_GLOBAL              0 (ord)
 9 LOAD_FAST                0 (flag)
12 LOAD_CONST               1 (0)
15 BINARY_SUBSCR       
16 CALL_FUNCTION            1
```

First, the symbol ord is put on stack, then index of flag variable and then the index into field of values, which contains value `0`. Then `BINARY_SUBSCR` instruction performs the `[]` operation and with `CALL_FUNCTION` the `ord()` is applied. Long story short, these instructions compose the following expression: `ord(flag[0])`. 

I am not going to interpret the whole code line by line, the principles remain the same. After a while, one composes the following python pseudo-code:

```python
ord(flag[0]) + 52 == ord(flag[-1]) 
and ord(flag[-1]) - 2 == ord(flag[7])

flag[:7] == 'ISITDTU'

flag[9] == flag[14] 
and flag[14] == flag[19] 
and flag[19] == flag[24] #'_'?

ord(flag[8]) == 49 and flag[8] == flag[16] # 1

flag[10:14] == 'd0nT'

int(flag[18]) + int(flag[23]) + int(flag[28]) == 9
    and flag[18] == flag[28] # 3

flag[15] == 'L'

ord(flag[17]) (xor) -10 == -99 #chr(107) = k

ord(flag[20]) + 2 == ord(flag[27]) 
and ord(flag[27]) <= 123 
and ord(flag[20]) >= 97

ord(flag[27]) % 100 == 0 # flag[27] == 100, flag[20] == 98 ? b,d

flag[25] == 'C'

ord(flag[26]) % 2 == 0 
and ord(flag[26]) % 3 == 0 
and ord(flag[26]) % 4 == 0
and isdigit(flag[26]) 

int(flag[23]) == 3

flag[22] == lower(flag[13]) # t

sum for i in flag = 2441

=> flag = ISITDTU{1_d0nT_L1k3_b:t3_C0d3}
```

These conditions reveal the flag pretty straightforward. One nasty thing is the `:` character which needs to be calculated from the sum of `ord`s applied on the flag, which we are given. At that point we know all the characters but this one, so one can compute without any problems. It's just something about `:` there that makes you feel like something's wrong.



The downloaded file can be found nearby in `pytecode`

### References

1. Python `dis` module: https://docs.python.org/2/library/dis.html
