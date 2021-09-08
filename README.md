# EasyICSF
Rexx programs and C programs to make programming of z/OS ICSF encryption APIs easier to use.
## JCL for rexx programs

- GAESEXJ : Generate AES keys for Export, Import and Cipher using DH and private and public keys. Parameters -KEY name -PRIVATE private_key label -PUBLIC public_key_label -TYPE E|I|C <-REPLACE>
- GPKIJ   : Generate an ECC PKI private.Public key pair.  Parameters -KEY  keyname <-REPLACE>
- EAESDHJ : Export an AES key using DH symmetric key. Parameters -KEY keyname  -PUBLIC exporterKeyName.  This puts the encrypted data in //EXPORT  with DCB=(LRECL=800,RECFM=VB,BLKSIZE=3200) 
- EPKIJ   : Export a PKI public key.  Parameters -KEY keyname.  This puts the encrypted data in //EXPORT  with DCB=(LRECL=800,RECFM=VB,BLKSIZE=3200)  
- IAESDHJ : Import an AES key created by EAESDH. Parameters -KEY outputKeyname  -PRIVATE Importer <-REPLACE>.  This reads from //IMPORT
- IPKIJ    : Import a PKI public key. Parameters -KEY outputKeyname.  This reads from //IMPORT 

## Rexx main functions 
- GAESEX : Generate AES keys for Export, Import and Cipher using DH and private and public keys
- GPKI   : Generate an ECC PKI private.Public key pair
- EAESDH : Export an AES key using DH symmetric key  
- EPKI   : Export a public key 
- IAESDH : Import an AES key created by EAESDH
- IPKI    : Import a PKI public key 
## Rexx helper functions.
These are called from the Rexx main execs.   They do one ICSF function.
When passing key data around it is passed - coverted to Hex. This prevents problems if there were characters which would cause parseing problems, such as blank  or comma.  I refer to this as the blob of data.
- ADDCKDS - Add a record to the CKDS.  
    - Parameters label,blob,replace where replace='Y' if the record is to be replaced if it exists already.  
    - Returns rc rs error_string 
- ADDPKDS - Add a record to the PKDS.  
    - Parameters label,blob,replace where replace='Y' if the record is to be replaced if it exists already.   
    - Returns rc rs error_string  
- CPRS  - Reason code to string program. 
    - Parameters code and reason code to get the string.  eg CPRS(8,.2016) returns "The rule_array parameter contents are incorrect".  
    - Returns the string (could be null string)
- DELCKDS  - Delete a key from the CKDS. 
    - Parameters label.  
    - Returns rc rs error_string 
- DELPKDS  - Delete a key from the PKDS. 
    - Parameters label.  
    - Returns rc rs error_string 
- EXISTSCK - Does a key exist in the CKDS. 
    - Parameters label.  
    - Returns rc rs error_string  
- EXISTSPK - Does a key exist in the PKDS. 
    - Parameters label.  
    - Returns rc rs error_string 
- EXPAES   - Export an AES key using PKI public key.  
    - Parameters label, public key name
    - Returns rc rs blob of encrypted key
- EXPAESE  - Export an AES key using Exported key.  
    - Parameters label, public key name
    - Returns rc rs blob of encrypted key
- EXPPKI   - Export an PKI public. 
    - Parameter label. 
    - Returns rc rs blob of key
- GENAES - Generate an AES key.  
    - Parameters label, blob of key from skeleton.  
    - Returns rc rs blob
- GENDH - Generate an AES key using DH. 
    - Parameters  name of private key, name of public key at the remote end, blob created by skeleton.   
    - Returns rc rs blob of key
- IMPAES   - Import an AES key encrypted with public key.  
    - Parameters label to put into key,name of private key, blob of input data. 
    - Returns rc rs blob of data encrypted with local key.
- IMPAESE  - Import an AES key encryped with Exporter key.  
    - Parameters label to put into key,name of importer key, blob of input data. 
    - Returns rc rs blob of data encrypted with local key.
- PKIGEN  - Generate a PKI key.  
     - Parameters, label, blob of data from skeleton.  
     - Returns rc rc blob.
- PKISKEL - Create a PKI skeleton for passing into PKIGEN.  
    - Parameters label.  It uses 'ECC-PAIR' 'KEY-MGMT' 
    - Returns rc rs blob.
- PKISKELQ- Create a PKI skeleton uing QSA.  Could not use it.  
    - Parameters label. Uses 'QSA-PAIR' 'U-DIGSIG'.  
    - Returns rc rs blob.
- QSAPRINT- Print out information from a QSA key. 
    - Parameters blob containing the key.  
    - Returns rc rs
- READCKDS- Read a record from the CKDS.  
    - Parameter label.  
    - Returns rc rs blob
- READPKS - Read a record from the PKDS.  
    - Parameter label.  
    - Returns rc rs blob
- SKELAES - Create a skeleton AES key for a CIPHER.  
    - Parameters none.  
    - Returns rc rs blob
- SKELAESE- Create a skeleton AES key for Exporter, Importer or Cipher.   
    - Parameters type (E|I|C).  
    - Returns rc rs blob.  The blob can be passed to GENAESE
