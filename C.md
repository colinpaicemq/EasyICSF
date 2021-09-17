#C interface

## C main programs

- EXPAES : export an AES key to a data set.
    - Parameters
        - -key name
        - -public pki public key
        - -dd  output data set,for example dd:CERT, where //CERT is in the JCL
        - -type Data |Cipher
    - It uses helper functions
        -    doExists("P",pPublic) to check the public key is available
        -    exportAES(pKey,pPublic,pType,&pData, &lData) to extract the key into pData, where the data is encrypted with the public key.
        -    writeKey(dd,pData,lData);  to write to the data set
    - The matching function is IMPAES     
- EXPPKI export a PKI public key
    - Parameters
        - -key name
        - -dd  output data set,for example dd:CERT, where //CERT is in the JCL
    - It uses helper functions
        - exportpki(pKey, &pExt, &lExt) to extract the key and returns it unencrupted in pExt
        - writeKey(dd,pExt,lExt) to write to the data set
    - The matching function is IMPPKI     
- GENAES  generate an AES key
    - Parameters
        - -key name
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added
     - It uses helper functions
        -  skeletonAES(& pToken,& lToken) which returns a skeleton in pToken
        -  GENAES2(pToken,&lToken) to take the skeleton and return the encrypted key
        -  addCKDS(pKey,pToken,lToken,pReplace) to pass in the key name, the encrypted key, and whether to replace or not if it already exists
- GENAESP This is a more advanced version of GENAESP, as it generates two keys. It returns the
doImportAES key, and returns an external key encrypted with the given public key.  The code has come #ifdef...#endif  to write the external key to a file and to import it into the current CKDS  
    - Parameters
        - -key name the name of the local key to be created
        - -key2 name - the name of the exported key (optional code to import it)
        - -public key for encruypting the exported key
        - -private the private key for decrypting the exernal key.
        - < -replace Y|N > If the key exists, then the old key is deleted and the new one added
        - -dd  output data set,for example dd:CERT, where //CERT is in the JCL
    - It uses helper functions
        - keyGENP(pPublic,&pLocal, &lLocal, &pExt,   &lExt);  to extract the key in pLocal, and the external version(encrypted with the public key) in pExt
        - addCKDS(pKey,pLocal,lLocal,pReplace) add the local data to the CKDS with name given key
        - < optional... 
        -  writeKey(dd,pExt,lExt) to write the external data to the specified dataset
        -  importPub(pPrivate,&pExt,&lExt) reads the external key, decrypts it, and returns the local version of it
        -  addCKDS(pKey2   , pExt,lExt,pReplace) add the local version of the external version to the CKDS with the name from pKEY2
        -  end optional>  
   
- GENDH generates an AES key using public and private keys so you have the same AES key on two systems without sending sensitive information between the two systems.
    - Parameters
        - -key name
        - -public name the public key of the remote end
        - -private name the private key of the local end
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added
     - It uses helper functions
        -  doExists("P",pPrivate) to check the private key exists
        -  doExists("P",pPublic) to check the public key exists
        -  skeletonAES(& pToken,& lToken) which returns a skeleton in pToken
        -  GENDH(pPrivate, pPublic,& pData,& lData);  to take the skeleton, private and public and return the encrypted key
        -  addCKDS(pKey,pToken,lToken,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

- GENPKI generates a PKI private and public pair of keys. 
    - Parameters
        - -key name
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added
     - It uses helper functions
        -  GENPKISKEL(pKey,&pData,&lData) passes in the key name (for storing in the key) and returns the skeleton
        -  GENPKIGEN(&pData, &lData);   to take the skeleton, private and and return the encrypted key.
        -  addPKDS(pKey,pToken,lToken,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

- IMPAES  Read a file, and import an AES key.  It uses the private key to decrypt it.
    - Parameters
        - -key name
        - -private name the name of the private key
        - -type the type of the key D for Data key (fixed format), or C for a Cipher(variable format)
        - < -replace Y|N > If the key exists, then the old key is deleted and the new one added
     - It uses helper functions
        - read(dd,&pData,&lData) read from the data set, return the data
        - doImportAES(pKey,pPrivate,pType,&pData,&lData) to take the name of the key, the private key name the type and the data read from the file decrypts it using the private key, encrypts the data with the master key,and returns the buffer.
        -  addCKDS(pKey,pData,lData,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

- IMPPKI import a public key
    - Parameters
        - -key name
        - -dd name of the data set for example dd:CERT maps to //CERT in the JCL
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added
     - It uses helper functions
        - read(dd,&pData,&lData) read from the data set, return the data
        - addPKDS(pKey,pData,lData,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

- LIST print out the CKDS and PKDS.   It lists the +KDS, displays the meta data, and displays detail information about some key types.
-     - No Parameters

- BINDOPTS these are the options for the binder( linkage editor)
- CCOPTS   these are the C compile options

## JCL for C programs
CC contains JCL to compile all the programs.   The other programs have JCL to compile and run the programs

- CC     : compile all of the program
- CCPROC : the procedure for compiling and binding the programs 
- EXPAES : program to export an AES key 
- EXPPKI : program to export an PKI public  
- GENAES : program to generate an AES key
- GENAESP
- GENDH  : program to export an AES key using DH, private and public keys 
- GENPKI : generate a PKI private and public key
- IMPAES : import an AES key
- IMPPKI : import a PKI public key 
- LIST   : lis the conents of the PKDS and CKDS


##  helper functions


### addCKDS(pKey,pToken,lToken,pReplace) 
to pass in the key name, the encrypted key, and whether to replace or not if it already exists.
It uses CSNBKRC2 to add the record, if the record exists, and pReplace = Y then use deleteCKDS to delete it, and retry the add.
### deleteCKDS(pKey) 
removes the record from the CKDS.  This uses CSNBKRD 
### addPKDS(pKey,pToken,lToken,pReplace) 
to pass in the key name, the encrypted key, and whether to replace or not
It uses CSNDKRC to add the record, if the record exists, and pReplace = Y then use deletePKDS to delete it, and retry the add.
### deletePKDS(pKey) 
removes the record from the CKDS.  This uses CSNDKRD 

### doExists("P",pPrivate)
 to check the private key exists
 This reads the keystore to see if the record exists.
 For PKDS it uses CSNDKRR
 For CKDS it uses CSNBKRR2

### doImportAES(pKey,pPrivate,pType,&pData,&lData)
 to take the name of the key, the private key name the type and the data read from the file and import it
 If type = D ( for Data) call importData
 else if type = C for Cipher call importCipher
### int importCipher(pKey, pPrivate, pTypeUnused, pData,lData) 
This uses rules  {"AES     ",  "PKOAEP2 "} and CSNDSYI2 to import the data.  It uses the private key to decrypt it, and stores it locally under the master key.

 
### importData int importData (pUnusedKey, pPrivate, pTypeUnused, pData, lData)
This uses rules {"AES     ",  "PKCSOAEP",  "SHA-256 "} and  CSNDSYI  to import the data.   It uses the private key to decrypt it, and stores it locally under the master key.

### exportAES(pKey,pPublic,pType,&pData, &lData)
 to extract the key into pData, where the data is encrypted with the public key.
 If type = Cipher then use rule =  {"AES     ", "PKOAEP2 ", "SHA-256 "} 
 If type = Data then use rule =  {"AES     ",  "PKCSOAEP",  "SHA-256 "}
 use CSNDSYX to export the key
### exportpki(pKey, &pExt, &lExt) 
to extract the key and returns it unencrupted in pExt
Use CSNDPKX
### GENAES2(pToken,&lToken)
 to take the skeleton and return the encrypted key. This is in member KEYAES.
 Rule rule[2]  = {"AES     ","OP      "}
 uses CSNBKGN2 with keyType1 = "TOKEN   " to say skeleton is passed in.
 
### GENDH(pPrivate, pPublic,& pData,& lData)
 to take the skeleton, private andl and return the encrypted key.  In member keyDH.
 Uses CSNDEDH( with
- rule =  = {"DERIV01 " ,"KEY-AES "}
- Party = "transaction-id"
- 
### GENPKIGEN(&pData, &lData)
 to take the skeleton, private and and return the encrypted key.  In Member KEYPKI. 
 Uses CSNDPKG with
 - rule  = {"MASTER  " }
### GENPKISKEL(pKey,&pData,&lData)
 passes in the key name (for storing in the key) and returns the skeleton.  In SKELPKI.
Uses CSNDPKB with
 rule  = {"RSA-CRT " ,"KEY-AES "}; 
### importPub(pPrivate,&pExt,&lExt) 
reads the external key, decrypts it, and returns the local version of it
Uses CSNDSYI with
- rule = {"AES     ", "PKCS-1.2", "OP      "}
### keyGENP(pPublic,&pLocal, &lLocal, &pExt, &lExt)
 to extract the key in pLocal, and the external version(encrypted with the public key) in pExt.
 Uses CSNDSYG with
- rule = {"AES     ", "PKCS-1.2","OP      "}; 
### read(dd,&pData,&lData)
 read from the data set, return the data.  No ICSF functions
### skeletonAES(& pToken,& lToken) 
which returns a skeleton in pToken
Uses CSNBKTB2 with
- rule ={ "INTERNAL","AES     ","CIPHER  ","XPRTCPAC","ANY-MODE" 

### writeKey(dd,pData,lData)
 to write to the data set.  This uses no ICSF functions.
