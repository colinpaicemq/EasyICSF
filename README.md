# EasyICSF
This is a major update to my EasyICSF package.  It provides functions which are missing in ICSF.
For the old documentation and Rexx code see [the old documentation](READMEOLD.md). 

I have blogged on many ICSF topics, 
see [All I ever wanted to know about ICSF, and data set encryption.](https://colinpaice.blog/?p=26839)   This coverss
on

-  [What are the ICSF bits?](https://colinpaice.blog/?p=26332)
-  [Using ICSF within a single image or sysplex](https://colinpaice.blog/?p=26378)

-  [Using ICSF in a multi system environment](https://colinpaice.blog/?p=26751)
-  [Generating ICSF keys using Diffie-Hellman](https://colinpaice.blog/?p=26785)
-  [Using ICSF API function CSNBKGN2 to generate keys](https://colinpaice.blog/?p=26545)
-  [What ICSF APIs do I need to use for AES CIPHER keys](https://colinpaice.blog/?p=26614)


This Github package provides load modules which can be run in batch to: 

- [list the contents of a PKDS and CKDS](#List-the-contents-of-the-CKDS-and-PKDS)
- [export a key from CKDS into a data set so it can be imported on another system](#Export-a-key)
- [import a key from a data set into the CKDS](#Import-a-key)
- [set meta data attributes](#Set-meta-data-attributes), such as archive, and validity dates
- [generate keys securely on multiple systems](#Generate-a-secure-shared-key-on-multiple-systems-using-Diffie-Hellman), using private/public keys and Diffie-Hellman implementation
- [generate a checksum for a key](#Generate-the-checksum-for-a-key) in the CKDS.

It also provides examples of how to use the ICSF application programming interface to 

- generate keys: cipher, exporter, importer

### Passing parameters 

You can pass parameters to the program using PARM='...', or with the PARMDD pointing to a data sets.
If you use PARMDD, trailing blanks are removed and the data is concatentated together.  In this case starting 
the parameters in column 2 (or higher) so you get a leading blank.

## Install the load modules

You need to ftp the loadmodule to z/OS.  For example with FTP you need parameters like



quote site filetype=seq
quote site recfm=fb
quote site lrecl=80
quote site blksize=400
quote site tracks
quote site primary=5
bin 
put ISPFLOAD.XMIT.BIN 'myuserid.ISPFLOAD'

Then from TSO use the command RECEIVE INDSN('myuserid.ISPFLOAD')  

##  Copy the JCL

You can copy the members CC* from  the C directory, or just use cut and paste from this document.





## List the contents of the CKDS and PKDS


```
//IBMUSLI JOB 'COLIN',CLASS=A,REGION=0M,COND=(4,LE) 
//JOBLIB JCLLIB ORDER=(COLIN.ICSF.C.JCL,CBC.SCCNPRC) 
//*COMPILE       EXEC  CCPROC,PROG=LIST,BINDOPTS=BINDGSK 
//RUN1     EXEC PGM=LIST,REGION=0M,PARMDD=MYPARMS 
//* start parmameters in column 2 
//MYPARMS DD * 
  -label ATOB*
/* 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=300) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
//CKDSA    DD SYSOUT=*,DCB=(LRECL=300) 
//CKDSI    DD SYSOUT=*,DCB=(LRECL=300) 
//CKDSARCH DD SYSOUT=*,DCB=(LRECL=300) 
//PKDSA    DD SYSOUT=*,DCB=(LRECL=300) 
//PKDSI    DD SYSOUT=*,DCB=(LRECL=300) 
//PKDSARCH DD SYSOUT=*,DCB=(LRECL=300) 
//TKDSA    DD SYSOUT=*,DCB=(LRECL=300) 
//TKDSI    DD SYSOUT=*,DCB=(LRECL=300) 
//TKDSARCH DD SYSOUT=*,DCB=(LRECL=300) 
//SUMMARY  DD SYSOUT=*,DCB=(LRECL=300) 
```

Where the parameters, all of them optional are

-label name 
: the specific label you want to display.  The label values are upper case.  You can specify a prefix followed by * to select 
those starting with the value.

-type  C|P|T 
: for just the CKDS, PKDS, or the TKDS

-datetype value" 
: where value is c Created, u Updated, r Referenced, a for Archived, s for Start date, e for End date, x for Recalled

-days n
: where n is the number of days from now

- +n is greater than n days,
- -n is less than n days
- n is exactly n days


-label value
: where value is the name of a label to be listed

-help
: this text

The DD statements are

CKDSA 
: CKDS records which are active

CKDSI
: CKDS records which are inactive

CKDSARCH
: CKDS records which are archived 

PKDSA 
: PKDS records which are active

PKDSI
: PKDS records which are inactive

PKDSARCH
: PKDS records which are archived 

TKDSA 
: TKDS records which are active

TKDSI
: TCKDS records which are inactive

TKDSARCH
: TKDS records which are archived 

SYSOUT
:  Error messages, such as  CKDS rc 8 rs 16004 RACF failed your request to use the key label or token 

SYSPRINT
: A list of the labels processed, and what records are being processed

### Example output

```
CKDS ACTIVE   AESDHZ                                                           EXPORTER AES      cr:20241204 08:28 up:  :         
              Service for reference          CSFKRR2                                                                              
              Record create days             2024/12/04 08:84:01.9                                                                
              Last reference days            2024/12/04                                                                           
              Record Archive                 no                                                                                   
              Prohibit Archive               no                                                                                   
Key:AESDHZ                                                                                                                        
 AES , EXPORTER , INTERNAL , VARIABLE , SYMMETRIC , Key Encrypted ,                                                               
  variable key                                                                                                                    
  {                                                                                                                               
    Length of record:163                                                                                                          
    Internal token                                                                                                                
    Version 5                                                                                                                     
    Key Verification:AES master                                                                                                   
    Wrapping Method:AESKW                                                                                                         
    Hash: SHA-256                                                                                                                 
    Variable length payload                                                                                                       
    Key use alg AES                                                                                                               
    Key type:Exporter                                                                                                             
    No Name within Key                                                                                                            
Installation defined                                                                                                              
00000000 : F140F2F0 F2F461F1 F261F0F4 40F0F87A  1 2024/12/04 08:  @    a  a  @  z                                                 
00000010 : F2F87AF4 F00040                      28:40.             z  .@                                                          
   }                                                                                                                              
```

## Export a key
This extracts a key from the CKDS and encrypts it with a Key Exporting Key (KEK).
It writes the encrypted key to the data set specified in //CSFKEYS, in the same format 
that the ICSF utility CSFKGUP.  You can send this data set to 
the remote system and import it.

### Example JCL
```
//IBMEXP  JOB 'COLIN',CLASS=A,REGION=0M,COND=(4,LE) RESTART=ISTEST 
//JOBLIB JCLLIB ORDER=(COLIN.ICSF.C.JCL,CBC.SCCNPRC) 
// SET DB='-debug 0      ' 
// SET KEK='-kek   AESDHZ          ' 
// SET KEY='-key   NORMAL3         ' 
//ISTEST   EXEC PGM=EXPORT,REGION=0M, 
//   PARM='&KEY.  &KEK.   &DB         ' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//CSFKEYS  DD DSN=COLIN.EXPORT.AES,DISP=(OLD) 
```


Where the parameters are

-key value
: the label of the record in the CKDS to export

-kek value
: the label of an exporter KEK in the CKDS to encrypt the data with

-debug n 
: where n is a value of 0 or larger. This is optional  If n > 0 it provides verbose diagnostic information.

## Import a key
This reads a data set containing the encrypted key, decrypts it with an importer KEK and writes it to the CKDS.
The data set record is in the same format that the ICSF utility CSFKGUP does

### Example JCL
```
//IBMIMP  JOB 'COLIN',CLASS=A,REGION=0M,COND=(4,LE) 
//JOBLIB JCLLIB ORDER=(COLIN.ICSF.C.JCL,CBC.SCCNPRC) 
//ISTEST   EXEC PGM=IMPORT,REGION=0M, 
// PARM='-replace y    -debug 8            ' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//CSFKEYS  DD DSN=COLIN.EXPORT.AES,DISP=(SHR) 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
```

The data is in the data set in //CSFKEYS.  The data contains the name of the original key, and the name of the kek used to encrypt it, and the type of the key (cipher, exporter, importer).
By default these values are used in the import.

The parameters are

-replace value
: where value is y|n.    If the record already exists in the CKDS, to replace it or not

-key value
: you can specify a label to use if you do not want to use the value in the input record

-kek value
: you can specify the name of an importer KEK if you do not want to use the value in the input record.

## Set meta data attributes 

```
//IBMSET  JOB 'COLIN',CLASS=A,REGION=0M,COND=(4,LE) 
//JOBLIB JCLLIB ORDER=(COLIN.ICSF.C.JCL,CBC.SCCNPRC) 
//COMPILE EXEC  CCPROC,PROG=CKDSSET 
//* 
// SET KEY='-key DSENCRYPTION  ' 
// SET ST=' -start 20241101' 
// SET EN=' -end   20251101' 
//ISTEST1  EXEC PGM=CKDSSET,REGION=0M, 
//   PARM='&KEY. -end   20251101 -debug 0 ' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
//ISTEST2  EXEC PGM=CKDSSET,REGION=0M, 
//   PARM='&KEY. -start 20241101 ' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
//ISTEST3  EXEC PGM=CKDSSET,REGION=0M, 
//   PARM='&KEY. -prohibit on    ' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
//ISTEST4  EXEC PGM=CKDSSET,REGION=0M, 
//   PARM='&KEY. -archive on     ' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
//ALL  EXEC PGM=CKDSSET,REGION=0M, 
//   PARM='&KEY. -archive on -prohibit on &ST &EN' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
```

The syntax is 

```
-key value  < -start yyyymmdd > < -end yyyymmdd > < -archive on|off  > < -probibit on|off >
```

where

-key value 
: value is the label in the CKDS

-start yyyymmdd
: where yyyymmdd is the start of the validity date range

-end yyyymmdd 
: where yyyymmdd is the end of the validity date range

-archive on|off 
: you can set [archive](https://www.ibm.com/docs/en/zos/3.1.0?topic=ckds-archiving-record-in-kdsr-format) on or off

-probibit on|off
: you can set [prohibit archive](https://www.ibm.com/docs/en/zos/3.1.0?topic=ckds-prohibiting-archival-record-in-kdsr-format) on or off.


## Generate a secure shared key on multiple systems using Diffie-Hellman 
You can generate the same value key on two systems, using the Diffie-Hellman technique.  It depends on each system having its own private/public key, and the public key of the other system.
You need a phrase which is used to generate the value.

### Example JCL

```
//IBMGENDH JOB 'COLIN',CLASS=A,REGION=0M,COND=(4,LE) 
//RUN      EXEC PGM=GENDH,REGION=0M,PARMDD=MYPARMS 
//* start parmameters in column 2 
//MYPARMS DD * 
 -ptype INTERNAL,AES,EXPORTER 
 -key AESDHZ 
 -private AAA 
 -public  BBB 
 -replace Y 
 -party cOlinSeed 
 -debug 8 
/* 
//STEPLIB  DD DISP=SHR,DSN=COLIN.LOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//CEEDUMP  DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
```

Where the parameters are 

-ptype value
: where value are the "rules" needed by the API.  With commas to separate them, to make them more readable.

- For a CIPHER specify -ptype INTERNAL,AES,CIPHER,XPRTCPAC,ANY-MODE 
- For an exporter specify -ptype INTERNAL,AES,EXPORTER 
- For an importer specify -ptype INTERNAL,AES,IMPORTER

-key value
: where value is the name of the key to be written into the CKDS

-private value
: where value is the name of an asymmetric private/public key pair in the local system's PKDS

-public value
: where value is the name of an asymmetric public key from the remote system.

-party value
: where value is a seed value.   The length must be between 8 and 64 characters.

-debug value
: where value is an integer.  If value is greater than 0, it writes out diagnostic information.

## Generate the checksum for a key

You can calculate the checksum of a key to ensure the key is the same on multiple systems.

### Example JCL

```
//IBMKEYT JOB 'COLIN',CLASS=A,REGION=0M,COND=(4,LE) RESTART=ISTEST 
//JOBLIB JCLLIB ORDER=(COLIN.ICSF.C,CBC.SCCNPRC) 
//*COMPILE     EXEC  CCPROC,PROG=KEYTEST 
// SET KEY='-key DSKEY             ' 
//ISTEST   EXEC PGM=KEYTEST,REGION=0M,PARM='&KEY         ' 
//STEPLIB  DD DISP=SHR,DSN=COLIN.ICSFLOAD 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
```

The output is like

```
Keyname:DSKEY                                 
Hexadecimal Key check value ABE08DEB,18769C3E 
```