##  Helper functions

The functions include code which takes the name of a null terminated string and converts it to a 64 character string.

There is a list of my interpretation of ICSF reason codes [here](https://colinpaice.blog/2021/08/26/icsf-return-codes-not-for-humans).

- pKey and pLabel are null terminated strings for the name of the key in the PKDS or CKDS
- pToken and pData are pointers to a block of storage containing the token (blob) with the key in it
- lToken and lData are the lengths of the above block. 
- pKek is the name of the key used to encrypt another key.   This is also know as a Key Encrypting Key (KEK) or transportation key.

## Key management

- [Add a record to the CKDS](#addCKDS)
- [Add a record to the PKDS](#addPKDS)
- [Delete a record from the CKDS](#deleteCKDS)
- [Delete a record from the PKDS](#deletePKDS)
- [Check to see if a key exists in the CKDS](#existC)
- [Check to see if a key exists in the PMDS](#exisP)
- [Get a record from the PKDS or CKDS](#findKey)

## File management
- [Read a record from a file and return it as a blob](#read)
- [Write a blob of data to a file](#writekey)

## Generate a key
- [Generate an AES skeleton for Cipher, Imported, or Exporter](#skeltonAES)
- [Generate an AES key using a skeleton](#GENAES2)
- 
- [Generate a symmetric key using Diffi-Hellman (which can be done on two sites to be able to get matching keys at each end](#GENDH)
- [Generate a PKI skeleton](#GENPKISKEL)
- [Generate a PKI from a skeleton](#GENPKIGEN)
- [(Advanced) Generate an AES symmetric key, save locally and generate one encrypted with KEK](#keyGENP)

## Export a key

- [Export a symmetric (AES) key using a public key to encrypt it](#exportAES)
- [Export a PKI key](#exportpki)

## Import a key
- [Import a blob of type Data encrypted with a KEK transport key to a blob encrypted with the local master key](#importAES)
- [Import a CIPHER, EXPORTER, or IMPORTER  blob of data encrypted with a KEK transport key to a blob encrypted with the local master key](#importCipher)
- [Import a DATA a blob of data encrypted with a KEK transport key to a blob encrypted with the local master key](#importData)-
- [Import a PKI public key](#importPub)

## Utility

- [Lookup an ICSF reason code to get a description](#csfgetrc)
- [From a blob return information about the blob, eg PKI, ECC, key size](#keyType)
- [Print the contents of AES key blob](#printAES)
- [Print the contents of a PKI key blob](#printPKI)
- [Create a 64 character variable from a null terminated string, suitable for passing as a key name to ICSF routines](#do64) 




### addCKDS
[addCKDS(pKey,pToken,lToken,pReplace)](CHelpers/ADDCKDS)  
Pass in the key name, the encrypted key, and whether to replace or not if it already exists.
It uses CSNBKRC2 to add the record, if the record exists, and pReplace = Y then use deleteCKDS to delete it, and retry the add.

 
### addPKDS(pKey,pToken,lToken,pReplace) 
[addPKDS(pKey,pToken,lToken,pReplace) ](Chelpers/ADDPKDS)
Pass in the key name, the encrypted key, and whether to replace or not.
It uses CSNDKRC to add the record, if the record exists, and pReplace = Y then use deletePKDS to delete it, and retry the add.

### csfgetrc 
[csfgetrc(return code,reason)](CHelpers/CSFGETRC)
Takes the reason code, and returns a string describing the reason code(for this problems I experienced).

### deleteCKDS
[deleteCKDS(pKey)](CHelpers/DELCKDS)
Removes the record from the CKDS.  This uses CSNBKRD.

### deletePKDS
[deletePKDS(pKey)](CHelpers/DELPKDS)
Removes the record from the CKDS.  This uses CSNDKRD.

### existsP
[existsP(pName)](CHelpers/EXISTS)
Check the key name exists in the PKDS exists using CSNDKRR.
This reads the PKDS keystore to see if the record exists.

### existsC
[existsC(pName)](CHelpers/EXISTS)
Check the key name exists in the CKDS exists using CSNBKRR2
This reads the CKDS keystore to see if the record exists.

### findKey
[findKey(type,pKey, &pData,&lData)](CHelpers/FINDKEY)
Return the key from the CKDS or PKDS.  If type is 

- "P" look only in the PKDS
- "C" look only in the CKDS
- " " look in both starting with the CKDS


### importAES
[importAES(pKey,pKek,&pData,&lData](CHelpers/IMPAESH)
Take the name of the key, the private key name, and the blob of data read from the file and import it.
It looks at the blob of data and deduces what type it is, 

- data (fixed format)
- RSA  PKI
- Importer (varible format)

it uses the transportation (KEK) key in pKEK to decrypt it, and the master key to encrypt it locally, before passing the data back.
 
### int importCipher
[importCipher(pKey, pPrivate, pTypeUnused, &pData,&lData)](CHelpers/IMPCIP)
This uses rules  {"AES     ",  "PKOAEP2 "} and CSNDSYI2 to import the data.  It uses the private key to decrypt it, and the local master key to encrypt it, and passes it back to the caller.
It takes the pKey name, and stores it as part of the certificate.

 
### importData  
[importData (pUnused_Key, pPrivate, pTypeUnused, pData, lData)](CHelpers/IMPDAT)
This uses rules {"AES     ",  "PKCSOAEP",  "SHA-256 "} and  CSNDSYI  to import the data.   It uses the private key to decrypt it, and stores it locally under the master key.

### exportAES
[exportAES(pKey,pPublic,&pData, &lData)](CHelpers/EXPAESK)
 to extract the key into pData, where the data is encrypted with the public key.

- If type = Cipher then use rule =  {"AES     ", "PKOAEP2 ", "SHA-256 "} 
- If type = Data then use rule =  {"AES     ",  "PKCSOAEP",  "SHA-256 "}

Use CSNDSYX to export the key

### exportpki
[exportpki(pKey, &pData, &lData)](CHelpers/EXPPKI)
Extract the key and returns it unencrypted in pData.  It use CSNDPKX.

### GENAES2 
[GENAES2(pToken,&lToken) ](CHelpers/KEYAES)
Take the skeleton and return the encrypted key. This is in member KEYAES.
 Rule rule[2]  = {"AES     ","OP      "}.  
 It uses CSNBKGN2 with keyType1 = "TOKEN   " to say skeleton is passed in.
 
### GENDH
[GENDH(pPrivate, pPublic,& pData,& lData,pParty)](CHelpers/GENH)
Take the private name, public key name, and the skeleton and return the encrypted key. The skeleton is created member keyDH.
pParty is a string to use as a randomiser, known as a party secret.
 Uses CSNDEDH( with

- rule =  = {"DERIV01 " ,"KEY-AES "}
- Party = "transaction-id"

### GENPKIGEN 
[GENPKIGEN(&pData, &lData)](CHelpers/KEYPKI)
Take the skeleton, private and and return the encrypted key. This is in member KEYPKI. 
 Uses CSNDPKG with

- rule  = {"MASTER  " }
### GENPKISKEL
[GENPKISKEL(pKey,&pData,&lData)](CHelpers/SKELPKI)
Generates a PKI skeleton. Pass in the key name (for storing in the key) and returns the skeleton. This  is in member SKELPKI.
Uses CSNDPKB with

- rule  = {"RSA-CRT " ,"KEY-AES "}; 
### importPub
[importPub(pPrivate,&pExt,&lExt)](CHelpers/IMPPUB)
Takes the external blob, decrypts it using the private key, and returns the local version of it, encrypted with the local master key.
It uses CSNDSYI with
- rule = {"AES     ", "PKCS-1.2", "OP      "}

### keyGENP 
[keyGENP(pPublic,&pLocal, &lLocal, &pExt, &lExt)](CHelpers/KEYGENP)
This generates a 2 part key. It creates a local key in pLocal, and the external version(encrypted with the specified public key) in pExt.
 Uses CSNDSYG with the rule
 
- rule = {"AES     ", "PKCS-1.2","OP      "}; 

### keyType
[keyType (pData,lData, &output,&lOutput)](CHelpers/KEYTYPE)
Pass in a blob, and return a string descriptor.
For example
```
    INTERNAL PKA      ECCPRIV  BP512   
```
Which says

- This key is encrypted with the local(Master) key
- It is a PKI key
- It is an ECC Private key
- It is of type Brain Pool with key length 512

and
```
    INTERNAL SYMMETRI EXPORTER CANAES      
```    
Which says

- This key is encrypted with the local(Master) key
- It is an AES (Symmetric)key
- It is an Exporter key
- It can be used for AES processing.

### read
[read(dd,&pData,&lData)](CHelpers/READ)
Read from the data set, return the data. dd is a string like "dd:CERT" used in fopen.   This maps to a data set in JCL with //CERT..  It has No ICSF functions

### skeletonAES
[skeletonAES(pType,& pToken,& lToken)](CHelpers/SKELAES)
Return an AES skeleton in pToken.  Ptype can be E|I|C for Exporter, Importer, Cipher key.
This uses CSNBKTB2 with

- Cipher    "INTERNAL","AES     ","CIPHER  ", "XPRTCPAC","ANY-MODE" 
- Importer  "INTERNAL","AES     ","IMPORTER"
- Exporter  "INTERNAL","AES     ","EXPORTER"

### writeKey 
[writeKey(dd,pData,lData)](CHelpers/WRITEKEY)
 to write to the data set. dd is a string like "dd:CERT" used in fopen.  This uses no ICSF functions.

### printAES
[printAES(pData, lData, File)](CHelpers/PRINTAES)
Formats the AES data blob to the File handle, such as stdout.

### printPKI
[printPKI(Data, lData, File)](CHelpers/PRINTPKI)
Formats the PKI data blob to the File handle, such as stdout.

### do64
[do64(Name,pKey)](CHelpers/DO64)
Creates a variable Name, from the null terminated string pKey - it takes the first 64 characters. 

