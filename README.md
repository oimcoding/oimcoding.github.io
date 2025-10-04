### **SU-CPSC 5710 25FQ Notes**

Canvas Link: [Granted access is needed for visiting this site](https://seattleu.instructure.com/courses/1623416)

![Person holding a humorous IT sticker outdoors, perfect for tech enthusiasts.](https://images.pexels.com/photos/11035465/pexels-photo-11035465.jpeg)

**Exercise 1: Access Control**

*   Task:
    
*   Cheatsheet:
    
    _linux access control_
    
    *   _find your default permissions with umask_
        
        umask output: 022 | maximum in octal(8): 777 
        
        default permissions: 777-22  = 755
        
        755 in binary representation: 111 101 101
        
        755 in symbolic notation: rwx r-x r-x
        
    *   _Commands_
        
        *   chmod: change user/group/others access of file/folder
            
            \- chomd new\_octal\_notation file/folder\_name
            
            \- chomd u/g/o + r/w/x file/folder\_name
            
        *   chgrp: change file/folder ownership to new group (chgrp new\_group file/folder)
            
        *   chown: change group ownership to new user (chown new\_user group)
            
        *   whoami: current user(name)
            
        *   groups: all groups which I belong
            

**Lecture 3: Cryptography**

> Understand several methods of symmetric key cryptography, public key cryptography on a high (math-free) level. Focus on the pros & cons of each method. (ex. Why this mechanism is brute-force free? Why we won't adapt this brute-force free approach?)

Definition:

> *   symmetric key: same key is used for encryption & decryption.
>     
> *   asymmetric/public key (opposite of symmetric): separate keys for encryption & decryption. Anyone with the public key can encrypt, only the holder of the private key can decrypt.
>     

Simple encryption scheme:

*   Substitution and Transposition (Permutation)
    
*   Block Cipher
    

Symmetric key encryption scheme:

*   DES, Chained DES, Chained DES with Initialization Vectors
    
*   A(Advanced)ES
    

Public key encryption scheme:

*   RSA
    
*   Elliptic Curve
    
    _Background knowledge: Diffie-Hellman Key Exchange Algorithm (not a encryption algorithm)_
    
    > In Diffie-hellman, a secret key isn't shared. Instead, a set of public values is used. The public values are then combined with private, hidden variables to create an identical shared key.
    
    > "What the mathematics behind Diffie-Hellman does, is allow the protocol to send messages where you can't extract this (which mixed in the shared public value) private variable and that's exactly what elliptic curves do, they just do in a slightly different way." - Dr Mike Pound in the Computerphile video
    

[![Elliptic Curves - Computerphile](https://i.ytimg.com/vi/NF1pwjL9-DE/hqdefault.jpg)](https://www.youtube.com/watch?v=NF1pwjL9-DE&list=PLzH6n4zXuckpoaxDKOOV26yhgoY2S-xYg&index=4)

**Questions:**

*   _How does RSA work?_
    
*   _Why does RSA secure?_
    
*   _Why does Elliptic Curve secure (Brute-force free)?_
    
*   _Why does Elliptic Curve more efficient (than RSA)?_
    
*   _RSA in practice: session keys_