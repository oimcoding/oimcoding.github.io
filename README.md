
### **SU-CPSC 5710 25FQ Notes** [Canvas Link](https://seattleu.instructure.com/courses/1623416)

***
**Lecture 1: Intro**
> Understand basic terms and CIA Triad.
>
C (Confidentially) -I (Integrity) -A (Avaliability)  
Vulnerability, threat, attack, control


***
**Lecture 2: Authentication, Access Control**


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
            \- chomd u/g/o - r/w/x file/folder\_name (SPACE IS IMPORTANT)
            
        *   chgrp: change file/folder ownership to new group (chgrp new\_group file/folder)
            
        *   chown: change group ownership to new user (chown new\_user group)
            
        *   whoami: current user(name)
            
        *   groups: all groups which I belong
            

***

**Lecture 3: Cryptography**

> Understand several methods of symmetric key cryptography, public key cryptography on a high (math-free) level. Focus on the pros & cons of each method. (ex. Why this mechanism is brute-force free? Why we won't adapt this brute-force free approach?)

Definition:

 *   symmetric key: same key is used for encryption & decryption.
     
 *   asymmetric/public key (opposite of symmetric): separate keys for encryption & decryption. Anyone with the public key can encrypt, only the holder of the private key can decrypt.
     

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
    
    > "What the mathematics behind Diffie-Hellman does, is allow the protocol to send messages where you can't extract this (which mixed in the shared public value) private variable and that's exactly what elliptic curves do, they just do in a slightly different way." - [Dr Mike Pound in the Computerphile video](https://www.youtube.com/watch?v=NF1pwjL9-DE&list=PLzH6n4zXuckpoaxDKOOV26yhgoY2S-xYg&index=4)


**Questions:**
*  _Why does simple encryption scheme/symmetric key not secure?_

*   _How does RSA work?_
    
*   _Why does RSA secure?_
    
*   _Why does Elliptic Curve secure (Brute-force free)?_
    
*   _Why does Elliptic Curve more efficient (than RSA)?_
    
*   _RSA in practice: session keys_

**Quick summaries (from ChatGPT)**

>Below are short, non-mathematical summaries and simple diagrams for each topic from Lecture 3. 

- Substitution & Transposition

    What: simple letter/position transforms (Caesar, substitution, columnar transposition).

    Diagram (very simple):

    Plain:  HELLO WORLD  
    Subst:  IFMMP XPSME   (each letter shifted)  
    Transp: HLOEL OLWRD   (columns read differently)  

    Attacker: frequency analysis, word patterns, known-plaintext.

- Block Ciphers 

    What: encrypt fixed-size blocks independently. Keys are the mappings in the table.

    Diagram (conceptual):

    [plaintext block] --(key)--> [cipher block]

    Attacker: brute-force (if key small), (key won't be big b/c it will comes with a huge table).  

- DES / DES-CBC (Chained DES with HEAD)

    What: older symmetric ciphers and modes. DES has short keys (56 bits + 8 bits not for encryption).  
    Lots for shuffle around, doesn't make the message larger.  
    
    Attacker: DES — brute force the 56-bit keys.
     
    DES‑CBC attack types:
    Brute force, padding‑oracle: if an application reveals whether padding was correct during decryption, an attacker can recover plaintext block‑by‑block.  
    IV misuse: reusing the same IV for multiple messages or using a predictable IV leaks relationships between plaintexts (first‑block XOR patterns).  
    If you still used DES‑CBC, you’d be exposed to both the weak key size and the above practical attacks. CBC can be safe only when used with a strong block cipher, a fresh random IV per message, and authenticated encryption (or a separate MAC).

- AES 

    What: modern standard block cipher (128/192/256-bit keys).

    Attacker: no practical cryptanalytic break for correct AES; main risks are side-channel leaks, weak keys, or bad modes.
    Practical outcome: secure and recommended when used with AEAD modes (GCM/CCM) and good randomness.

- RSA 

    What: public-key system based on multiplying large primes. Public key encrypts; private key decrypts.

    Diagram (conceptual):

    Sender: message --(encrypt with recipient's public key)--> ciphertext --> Recipient: decrypt with private key

    Attacker: factor the large modulus to recover the private key; exploit poor padding or small exponents; side-channel attacks.
    Practical outcome: secure with large keys (2048+) and modern padding; slow for bulk data — usually used to protect session keys.

- Elliptic Curve 

    What: public-key methods using elliptic curve point math; security from the difficulty of reversing scalar multiplication.

    Diagram (conceptual):

    P (base point) + k -> Q (public point), attacker sees P,Q but can't easily find k.

    Attacker: generic discrete-log attacks (Pollard-rho), weak curves, and implementation side-channels.
    Practical outcome: very strong for smaller key sizes; efficient on constrained devices; choose well-known secure curves.

-- End of ChatGPT summaries --