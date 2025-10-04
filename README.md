### **SU-CPSC 5710 25FQ Notes**

Canvas Link: [Granted access is needed for visiting this site](https://seattleu.instructure.com/courses/1623416)

![Person holding a humorous IT sticker outdoors, perfect for tech enthusiasts.](https://images.pexels.com/photos/11035465/pexels-photo-11035465.jpeg)

\---

**Lecture 3: Cryptography**

> Understand several methods of symmetric key cryptography, public key (asymmetric) key cryptography on a high (math-free) level. Focus on the pros & cons of each method. (ex. Why this mechanism is brute-force free? Why we won't adapt this brute-force free approach?)

RSA

> One important characteristic here is that there are separate keys for encryption & decryption for RSA. -GeeksForGeeks website

Diffie-Hellman (not mentioned in the lecture)

> In Diffie-hellman, a secret key isn't shared. Instead, a set of public values is used. The public values are then combined with private, hidden variables to create an identical shared key.

Elliptic Curve

> "What the mathematics behind Diffie-Hellman does, is allow the protocol to send messages where you can't extract this (which mixed in the shared public value) private variable and that's exactly what elliptic curves do, they just do in a slightly different way." - Dr Mike Pound in the Computerphile video

[![Elliptic Curves - Computerphile](https://i.ytimg.com/vi/NF1pwjL9-DE/hqdefault.jpg)](https://www.youtube.com/watch?v=NF1pwjL9-DE&list=PLzH6n4zXuckpoaxDKOOV26yhgoY2S-xYg&index=4)

*   _Why is Elliptic Curve secure (Brute-force free)?_
    
*   _Why is Elliptic Curve efficient?_