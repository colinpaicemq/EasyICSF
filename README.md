# EasyICSF
Rexx programs and C programs to make programming of z/OS ICSF encryption APIs easier to use.

## For example 
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


See [REXX](Rexx.md) for more details and [C](C.md) for examples in C.