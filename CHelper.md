##  Helper functions

The functions include code which takes the name of a null terminated string and converts it to a 64 character string.

There is a list of my interpretation of ICSF reason codes [here](https://colinpaice.blog/2021/08/26/icsf-return-codes-not-for-humans).

- pKey and pLabel are null terminated strings for the name of the key in the PKDS or CKDS
- pToken and pData are pointers to a block of storage containing the token (blob)with the key in it
- lToken and lData are the lengths of the above block. 
- pKek is the name of the key used to encrypt another key.   This is also know as a Key Encrypting Key (KEK) 

### addCKDS(pKey,pToken,lToken,pReplace) 
Pass in the key name, the encrypted key, and whether to replace or not if it already exists.
It uses CSNBKRC2 to add the record, if the record exists, and pReplace = Y then use deleteCKDS to delete it, and retry the add.

### deleteCKDS(pKey) 
Removes the record from the CKDS.  This uses CSNBKRD.

### addPKDS(pKey,pToken,lToken,pReplace) 
Pass in the key name, the encrypted key, and whether to replace or not.
It uses CSNDKRC to add the record, if the record exists, and pReplace = Y then use deletePKDS to delete it, and retry the add.

### csfgetrc(return code,reason)
Takes the reason code, and returns a string describing the reason code(for this problems I experienced).

### deletePKDS(pKey) 
Removes the record from the CKDS.  This uses CSNDKRD.

### existsP(pName)
Check the key name exists in the PKDS exists using CSNDKRR.
This reads the PKDS keystore to see if the record exists.

### existsC(pName)
Check the key name exists in the CKDS exists using CSNBKRR2
This reads the CKDS keystore to see if the record exists.

### importAES(pKey,pKek,&pData,&lData)
Take the name of the key, the private key name, and the blob of data read from the file and import it.
It looks at the blob of data and deduces what type it is, 

- data (fixed format)
- RSA  PKI
- Importer (varible format)

it uses the transportation (KEK) key in pKEK to decrypt it, and the master key to encrypt it locally, before passing the data back.
 
### int importCipher(pKey, pPrivate, pTypeUnused, &pData,&lData) 
This uses rules  {"AES     ",  "PKOAEP2 "} and CSNDSYI2 to import the data.  It uses the private key to decrypt it, and the local master key to encrypt it, and passes it back to the caller.
It takes the pKey name, and stores it as part of the certificate.

 
### importData int importData (pUnused_Key, pPrivate, pTypeUnused, pData, lData)
This uses rules {"AES     ",  "PKCSOAEP",  "SHA-256 "} and  CSNDSYI  to import the data.   It uses the private key to decrypt it, and stores it locally under the master key.

### exportAES(pKey,pPublic,pType,&pData, &lData)
 to extract the key into pData, where the data is encrypted with the public key.
 If type = Cipher then use rule =  {"AES     ", "PKOAEP2 ", "SHA-256 "} 
 If type = Data then use rule =  {"AES     ",  "PKCSOAEP",  "SHA-256 "}
 use CSNDSYX to export the key

### exportpki(pKey, &pData, &lData) 
Extract the key and returns it unencrypted in pData.  It use CSNDPKX.

### GENAES2(pToken,&lToken)
Take the skeleton and return the encrypted key. This is in member KEYAES.
 Rule rule[2]  = {"AES     ","OP      "}.  
 It uses CSNBKGN2 with keyType1 = "TOKEN   " to say skeleton is passed in.
 
### GENDH(pPrivate, pPublic,& pData,& lData)
Take the private name, public key name, and the skeleton and return the encrypted key. This is in member keyDH.
 Uses CSNDEDH( with

- rule =  = {"DERIV01 " ,"KEY-AES "}
- Party = "transaction-id"

### GENPKIGEN(&pData, &lData)
Take the skeleton, private and and return the encrypted key. This is in member KEYPKI. 
 Uses CSNDPKG with

- rule  = {"MASTER  " }
### GENPKISKEL(pKey,&pData,&lData)
Generates a PKI skeleton. Pass in the key name (for storing in the key) and returns the skeleton. This  is in member SKELPKI.
Uses CSNDPKB with

- rule  = {"RSA-CRT " ,"KEY-AES "}; 
### importPub(pPrivate,&pExt,&lExt) 
Takes the external blob, decrypts it using the private key, and returns the local version of it, encrypted with the local master key.
It uses CSNDSYI with
- rule = {"AES     ", "PKCS-1.2", "OP      "}
### keyGENP(pPublic,&pLocal, &lLocal, &pExt, &lExt)
This generates a 2 part key. It creates a local key in pLocal, and the external version(encrypted with the specified public key) in pExt.
 Uses CSNDSYG with the rule
 
- rule = {"AES     ", "PKCS-1.2","OP      "}; 
### read(dd,&pData,&lData)
Read from the data set, return the data. dd is a string like "dd:CERT" used in fopen.   This maps to a data set in JCL with //CERT..  It has No ICSF functions
### skeletonAES(pType,& pToken,& lToken) 
Return an AES skeleton in pToken.  Ptype can be E|I|C for Exporter, Importer, Cipher key.
This uses CSNBKTB2 with

- Cipher    "INTERNAL","AES     ","CIPHER  ", "XPRTCPAC","ANY-MODE" 
- Importer  "INTERNAL","AES     ","IMPORTER"
- Exporter  "INTERNAL","AES     ","EXPORTER"


### writeKey(dd,pData,lData)
 to write to the data set. dd is a string like "dd:CERT" used in fopen.  This uses no ICSF functions.
