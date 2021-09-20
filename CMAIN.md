
## High level C programs

- [EXPAES   export an AES key to a data set](#EXPAES)
- [EXPPKI   export a PKI public key](#EXPPKI)
- [GAESXDH  generate a key using Diffi-Hellman](#GAESXDH)
- [GENAES   generate an AES key for a cipher key]({#GENAES)
- [GENAESP  a more advanced version of GENAES, as it generates two keys](#GENAESP)
- [GENDH    generates an AES key using public and private keys](#GENDH)
- [GENPKI   generates a PKI private and public pair of keys](#GENPKI)
- [GENPKI   generates a PKI private and public pair of keys](#GENPKI)
- [IMPAES   Read a file, and import an AES key](#IMPAES)
- [IMPPKI   import a public key](#IMPPKI)
- [LIST     print out the CKDS and PKDS](#LIST)

Other members of the C source PDS.

- [BINDOPTS the options for the binder(linkage editor)](#binder)
- [CCOPTS   the C compile options ](#ccopts)


#### EXPAES : export an AES key to a data set. {#EXPAES}

    - Parameters
        - -key name
        - -public pki public key
        - -dd  output data set,for example _-dd dd:CERT_, where //CERT is in the JCL
        - -type Data |Cipher
    - It uses helper functions:
        -    doExists("P",pPublic) to check the public key is available.
        -    exportAES(pKey,pPublic,pType,&pData, &lData) to extract the key into pData, where the data is encrypted with the public key.
        -    writeKey(dd,pData,lData);  to write to the data set.
    - The matching function is IMPAES to import the certificate from a file to the key store.     
#### EXPPKI export a PKI public key. {#EXPPKI}
    - Parameters:
        - -key name
        - -dd  output data set,for example dd:CERT, where //CERT is in the JCL
    - It uses helper functions:
        - exportpki(pKey, &pExt, &lExt) to extract the key and returns it unencrypted in pExt
        - writeKey(dd,pExt,lExt) to write to the data set.
    - The matching function is IMPPKI.

#### GAESXDH generate a key using Diffi-Hellman. {#GAESXDH}
This generates an Exporter, Importer or Cipher key using Diffi-Hellman.  It uses a private and public key PKI key.

     - Parameters
         - -key name
         - <-replace Y|N> If the key exists and Y is specified, then the old key is deleted and the new one added
         - -type E|I|C for key type to generate
         - -private name,  the name of the private key in the PKDS 
         - -public name, the name of the public key in the PKDS.
         - -party string, the common secret between the two systems.
     -  It uses helper functions
         -  skeletonAES(pType,&pData, &lData) passing the type of key to generate and gets back the skeleton key.
         -  GENDH(pPrivate, pPublic,& pData,& lData,pParty) passing the name of the private key, the name of the public key, and the data from the skeletonAES.
         -  addCKDS(pKey,pData,lData,pReplace) passing the key name to add to the CKDS, the key ( with length) and whether to replace it if it already exists.



#### GENAES  generate an AES key for a cipher key. {#GENAES}
    - Parameters:
        - -key name
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added.
     - It uses helper functions:
        -  skeletonAES("C",& pToken,& lToken) which asks for a Cipher key and returns a skeleton in pToken.
        -  GENAES2(pToken,&lToken) to take the skeleton and return the encrypted key.
        -  addCKDS(pKey,pToken,lToken,pReplace) to pass in the key name, the encrypted key, and whether to replace or not if it already exists.

#### GENAESP A more advanced version of GENAES, as it generates two keys. {#GENAESP}
It returns the key encrypted by the local master key, and returns an external key encrypted with the given public key, so it can be sent to a remote site.

The code has come #ifdef...#endif  to write the external key to a file and to import it into the current CKDS.
I think it would be better operationally to generate the certificate in one step, and export it when needed.   This way you can be sure you have the correct certificate.

    - Parameters:
        - -key name the name of the local key to be created.
        - -key2 name - the name of the exported key (optional code to import it).
        - -public key for encruypting the exported key.
        - -private the private key for decrypting the exernal key.
        - < -replace Y|N > If the key exists, then the old key is deleted and the new one added.
        - -dd  output data set,for example dd:CERT, where //CERT is in the JCL.
    - It uses helper functions:
        - keyGENP(pPublic,&pLocal, &lLocal, &pExt,   &lExt);  to extract the key in pLocal, and the external version(encrypted with the public key) in pExt
        - addCKDS(pKey,pLocal,lLocal,pReplace) add the local data to the CKDS with name given key
        - < optional... 
        -  writeKey(dd,pExt,lExt) to write the external data to the specified dataset
        -  importPub(pPrivate,&pExt,&lExt) reads the external key, decrypts it, and returns the local version of it
        -  addCKDS(pKey2   , pExt,lExt,pReplace) add the local version of the external version to the CKDS with the name from pKEY2
        -  end optional>  
   
####GENDH generates an AES key using public and private keys. {#GENDH}
This allows you tohave the same AES key on two systems without sending sensitive information between the two systems.  You run the same job at each end, but swap the private and public keys.

    - Parameters:
        - -key name, the name to store the key inthe PKDS.
        - -public name, the public key of the remote end
        - -private name, the private key of the local end
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added
        - -party string, the common secret between the two systems.
     - It uses helper functions
        -  doExists("P",pPrivate) to check the private key exists
        -  doExists("P",pPublic) to check the public key exists
        -  skeletonAES(& pToken,& lToken) which returns a skeleton in pToken
        -  GENDH(pPrivate, pPublic,& pData,& lData,pParty);  to take the skeleton, private and public and return the encrypted key
        -  addCKDS(pKey,pToken,lToken,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

#### GENPKI generates a PKI private and public pair of keys. {#GENPKI}
    - Parameters
        - -key name
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added
     - It uses helper functions
        -  GENPKISKEL(pKey,&pData,&lData) passes in the key name (for storing in the key) and returns the skeleton
        -  GENPKIGEN(&pData, &lData);   to take the skeleton, private and and return the encrypted key.
        -  addPKDS(pKey,pToken,lToken,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

#### IMPAES Read a file, and import an AES key. {#IMPAES} 
It uses the private key to decrypt it.

    - Parameters
        - -key name
        - -private name the name of the private key
        - -type the type of the key D for Data key (fixed format), or C for a Cipher(variable format)
        - < -replace Y|N > If the key exists, then the old key is deleted and the new one added
     - It uses helper functions
        - read(dd,&pData,&lData) read from the data set, return the data
        - doImportAES(pKey,pPrivate,pType,&pData,&lData) to take the name of the key, the private key name the type and the data read from the file decrypts it using the private key, encrypts the data with the master key,and returns the buffer.
        -  addCKDS(pKey,pData,lData,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

#### IMPPKI import a public key {#IMPPKI}
    - Parameters
        - -key name
        - -dd name of the data set for example dd:CERT maps to //CERT in the JCL
        - < -replace Y|N > If the key exists and Y is specified, then the old key is deleted and the new one added
     - It uses helper functions
        - read(dd,&pData,&lData) read from the data set, return the data
        - addPKDS(pKey,pData,lData,pReplace) to pass in the key name, the encrypted key, and whether to replace or not

#### LIST print out the CKDS and PKDS. {#LIST}   
It lists the +KDS, displays the meta data, and displays detail information about some key types.
       - No Parameters

#### BINDOPTS the options for the binder(linkage editor) {#binder}
#### CCOPTS   the C compile options {#ccopts}
