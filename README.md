# ctfs

DISsecT@CTU repository for all ctf-related stuff.

## Major League Cyber profile
https://www.majorleaguecyber.org/teams/DISsecT%40CTU#

## ctftime profile
https://ctftime.org/team/80103

## Structure
One folder per CTF, which contains one folder for each task.
Example of single ctf folder:
```
├── 2019
│   ├── ISITDTUCTF_QUALS                        # CTF directory
│   │   ├── readme.md                           # CTF summary
│   │   └── rev_Pytecode_100pts                 # Challenge directory
│   │       ├── pytecode                        # Challenge file
│   │       └── readme.md                       # Challenge write up
│   └── Redpwn_CTF
│       ├── pwn_BronzeRopchain_50pts
│       ├── pwn_ROT26_50pts
│       │   ├── img                             # Challenge write up images directory
│       │   │   ├── fail.png
│       │   │   └── success.png
│       │   └── readme.md                       # Challenge write up
│       ├── pwn_Zipline_50pts
│       ├── readme.md                           # CTF summary 
│       └── re_JavaIsEZ_120pts
```

## Naming conventions
```
<> - compulsory
{} - only in case it is needed
```

### CTF folder
`<YYYY>/<CTF_NAME>_{QUALS|FINALS}`

examples:
`2019/WPICTF`
`2019/ISITDTU`


### TASK folder
`<category>_<TASK_NAME>_<points>pts` 

examples:
`pwn_debugme_100pts`
