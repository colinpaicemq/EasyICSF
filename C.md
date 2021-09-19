# C programs


You can invoke these programs using JCL, see below.  For example:
```
// Create an Importer key using Diffi-Hellman with Private and Public keys.
// SET PR='-private ECCB512Z ' 
// SET PU='-public MYPUB ' 
/* 
// SET TYPE='I' 
//ISTEST   EXEC PGM=GAESXDH,REGION=0M, 
// PARM='-key AESDH&TYPE.  -replace y -type &TYPE &PR &PU' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
```
See 

-  [high level programs](CMAIN.md)
-  [JCL to run them](CJCL.md)
-  [Individual C helper programs](CHelper.md) 


