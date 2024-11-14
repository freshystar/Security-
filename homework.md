# Security and data encryption

   Information security is a comprehensive approach to safeguaring all types of information assets. Securing data with encryption is of very much importance in many aspects of today’s system administration. Moreover, when it comes to accessing systems remotely. Encryption is the basic building block of data security.   
   #### Data encrytion converts data from readable plaintext into an unreable encoded format: ciphertext.
   There are two types of data encryption:
   - Symmetric data encryption and
   - Asymmetric data encryption
   ## Symmetric and Asymetric data encryption
   In the old times and even now, many people used symmetric encryption.
   We had a brief view of what this was all about with Carrie Anne. In simpler words, with symmetric encryption, symmetric keys were used: both parties had a shared password. The data were encrypted with that password and then decrypted using the same password. 
 Asymmetric data encryption was a better approach to data security, since it had to do with the notion of `key pairs`.When we talk of key pairs, we are talking of `public` and `private` keys. 
 Whatever I lock with a private key can only be unlocked with a public key and vice-versa.
   ## SSH Keys
   SSH(Security SHell) is a network communication protocol that enables two computers to communicate and share data. SSH provides a secure encrypted connection between two hosts over an insecure network. It does this via `key pairs`. 
   The connection done via SSH can also be used for terminal access, file transfers and tunneling other applications. SSH works in `authenticating` host and `secure` traffic.
   For more security, the public keys makes use of `fingerprints`.A `fingerprint` is a unique identifier for the server's public key, ensuring the client communicates with the intended server and not a malicious impersonator. Every fingerprint has a `randomart` image. This is simply a support to the public key's fingerprint to compare the public keys `visually`.
   Lets's have an example:
   ```sh
   ssh <ip>
   ```
   

