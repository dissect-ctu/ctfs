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
├── 2019-ISITDTUCTF_QUALS
│   ├── readme.md - contains ctf summary
│   └── [Pwn]debugme_100pts
│       ├── {attached_files} (challenge files, exploits, solvers... )
│       └── readme.md - contains challenge write up
```

## Naming conventions
```
<> - compulsory
{} - only in case it is needed
```

### CTF folder
`<YYYY>-<CTF_NAME>_{QUALS|FINALS}`

examples:
`2019-WPICTF`
`2019-WPICTF_QUALS`


### TASK folder
`[<category>]<TASK_NAME> <points>pts` 

examples:
`[pwn]debugme 100pts`
