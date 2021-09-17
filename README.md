# EasyICSF
Rexx programs and C programs to make programming of z/OS ICSF encryption APIs easier to use.

## For example REXX 
run the Rexx exe from TSO 
```
%GAES -KEY REXXAES  -REPLACE 
```
This creates an AES key with name REXXAES.
The GAES exec has 
```
/* Rexx */ 
/*********************************************************************/ 
/* High level rexx to generate an AES key                            */ 
/* Syntax GAES -KEY name <-REPLACE>                                  */ 
/*********************************************************************/ 
signal on novalue 
parse upper arg   . "-KEY" label . 
                                                                         
parse upper arg args 
if ( wordpos("-REPLACE",args) > 0) then replace="Y" 
else replace="NO" 
say " generate an AES key " 
trace e 
if ( label =  "") then 
do 
   say "syntax is  -KEY key <-Replace>          " 
   return 8 
end 
/*********************************************************************/ 
/* now the ICSF code                                                 */ 
/*********************************************************************/ 
/* Generate an AES using a skeleton                                  */ 
/*********************************************************************/ 
rc = SKELAES() 
parse var rc myrc myrs skel 
if myrc <> 0 then return rc 
                                                                             
/*********************************************************************/ 
/* Generate the key. Pass the skeleton to the create routine         */ 
/*********************************************************************/ 
rc= GENAES(label,skel) 
parse var rc myrc myrs Data 
if myrc <> 0 then return myrc 
/*********************************************************************/ 
/* Add it to the CKDS                                                */ 
/*********************************************************************/ 
rc = addckds(label,Data,replace) 
return 0 

```
Where SKELEAS(), GENAES(), addckds() are helper rexx execs which issue the ICSF functions. 


See [REXX](Rexx.md) for more details

##Example of C code

```
 rc =exportAES (pKey,pKek,&pData, &lData); 
 if (rc > 0 ) return rc; 
 rc = writeKey("dd:CERT",pData,lData); 
```
You pass in the null terminated C string pKey, and the label of the key to be used for 
encryption (the Key Encryption Key).  It returns a pointer to a blob of data pData,
or length lData).
It writes the data to the data set //CERT .... 

The exportAES routine has logic to detect if the KEK is a PKI key, or an AES Exporter key,
and chooses the appropriate options.

See [C](C.md) for examples in C.

There are helper routines for example

- ADDCKDS to add blob of data to the CKDS with the given key name
- DELCKDS to delete a key entry from the CKDS 
- CSFGETRC to get the error string for some reason codes (the ones I experienced).
- GENDH generate a Diffie- Hellman symmetric key, passing the private and public key names.   
