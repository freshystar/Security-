# Security and data encryption

   Information security is a comprehensive approach to safeguaring all types of information assets. Securing data with encryption is of very much importance in many aspects of today’s system administration. Moreover, when it comes to accessing systems remotely. Encryption is the basic building block of data security.   
   **Data encrytion converts data from readable plaintext into an unreable encoded format: ciphertext**.
   There are two types of data encryption:
   - Symmetric data encryption and
   - Asymetric data encryption
   ## Symmetric and Asymetric data encryption
   In the old times and even now, many people used symmetric encryption.
   We had a brief view of what this was all about with Carrie Anne. In simpler words:
   - **symmetric** encryption, symmetric keys were used: both parties had a shared password. The data were encrypted with that password and then decrypted using the same password. 
 - **Asymetric** encryption was a better approach to data security, since it had to do with the notion of `key pairs`.When we talk of key pairs, we are talking of `public` and `private` keys. 
 Whatever I lock with a private key can only be unlocked with a public key and vice-versa.
   ## SSH Keys
   SSH(Security SHell) is a network communication protocol that enables two computers to communicate and share data. SSH provides a secure encrypted connection between two hosts over an insecure network. It does this via `key pairs`. 
   The connection done via SSH can also be used for terminal access, file transfers and tunneling other applications. SSH works in `authenticating` host and `secure` traffic.
   For more security, the public keys makes use of `fingerprints`.A `fingerprint` is a unique identifier for the server's public key, ensuring the client communicates with the intended server and not a malicious impersonator. Every fingerprint has a `random art` image. This is simply a support to the public key's fingerprint to compare the public keys `visually`.

   #### Known host keys

   One of the unique features of SSH is that by default, it trusts and remembers the host's key when first connecting to it. The memorized host keys are called **known host keys** and they are stored in a file called **known_hosts** in OpenSSH. As long as host keys don't change, this approach is very easy to use and provides fairly good security. The `known_hosts` file as the name suggests, is a file in the `~/.ssh/` directory that contains legite servers, their keys and fingerprint.
   - We have the `/etc/ssh` in which we can find the  configuration files for the client, that of the server and four other `key pairs` for each supported algorithm. These keys are created when *OpenSSH* server is installed. As already noted, the server uses these host keys to identify itself to clients as required.

   If it happens that I edit the file `known_hosts` and change a given server's fingerprint, I won't be able to access that server again. An error identification message will be displayed. To fix that, I will have to remove this server from my `known_hosts` file.
   ```sh
   -> ~ ssh-keygen -R 192.168.77.8
   ```
   Asssuming that our server in this case is `192.168.77.8`.
   Then I can now do:
   ```sh
   -> ~ ssh <user>@192.168.77.8
   ```
   To reconnect to this server and get a new key and fingerprint.

   ####  Creating your own key pairs
   To create/generate your own key pairs, you can do:
   ```sh
   -> ~ ssh-keygen
   ```
   This will display an interactive output that will guide me to creating my key pairs. By default, `rsa` key pairs will be generared. `rsa` is an algorithm to generate keys. Some others include:
   - dsa
   - ecdsa
   - ecdsa-sk
   - ed25519
   - ed25519-sk
   Back to the previous command, you will notice that `ssh-keygen`creates a private `id_rsa` key, a public `id_rsa.pub` key, shows us the fingerprint and corresponding `random art image`. 
  Let us create  `ed25519` key pairs.
  ```sh
  -> ~ ssh-keygen -t ed25519
  ```
  This will create a private `id_ed25519` key, public `id_ed25519.pub`key, a fingerprint and corresponding random art image. This will be generated in the `~/.ssh/` directory.
  The system keys for connection(ssh server) are located in the `/etc/ssh/` directory

  - `-t` in this case is the switch to specify the algorithm that will be used to generate the key pairs. If absent, `rsa` will be taken as default algorithm to generate the keys.
  **Never share your private key.You can only share your public key**
  - When creating the key pair,you can pass `ssh_keygen` with the `-b` option to specify key size in bits (e.g `ssh_keygen -t ecdsa -b 521`).
  - The permissions on the files containing the *private keys* are `0600` that is *read,write* only for the owner(root), while that of a public key file is `0644`.
  - You can view fingerprints of the keys by pasing `-l` switch plus `-f` to specify the key file path. The `-v` will enabke you to see the corresponding random art image.

  #### Password less login
  One of the biggest usages of private keys in ssh is in **password less login**. 
The ssh server can be configured to check your identity using your keys. It's enough to save your public-key on the server's user account and tell the server to check for key-based logins too. In this case, your ssh client will provide your private key to the ssh server as a proof of identity and you will be able to enter the user without providing passwords.
You can manually copy your public key and login into the server.Add it to `~/.ssh/authorized_keys` file, while making sure that in `/etc/ssh/sshd_config`, `PubKeyAuthentication` is `Yes`. Now log out and on the next login you should be able to login without password.
`PubKeyAuthentication Yes` means that you can login into an ssh account using your public key rather than a password. You could `cat` out the content of your public key and `pipe` it to `ssh` like so:
```sh
-> ~ cat id_ed25519.pub | ssh <user>@192.168.77.8 'cat >> .ssh/authorized_keys'
```
The configuration file for ssh is found in the `/etc/ssh/sshd_config`.

#### ssh-agent
The **ssh-agent** works like a key manager for your ssh keys. It keeps your private keys in memory and provides them when needed. For example if you have set a password on your key, you have to type the password every single time you are going to use that key. To make this easier using ssh-agent, you have to rush a shell with ssh-agent and add all your key to the agent:
```sh
-> ~ ssh-agent /bin/bash
-> ~ ssh-add
```
Now, the agent knows about your keys and wont ask for the passwords anymore, because all the keys are in memory.
This is the main usage of the `ssh-agent`. The ssh-agent can also be used to `tranfer keys`.

#### ssh tunnels
As earlier said, ssh can also be used for `tunnelling`. Just as the name suggests,ssh tunelling `tunnels` the data between machines. For example:
```sh
-> ~ ssh -L 7000:github.com:80 desmond@10.39.78.208
```
Here, I create a local tunnel `(-L)` from my machine towards the `github.com` port `80` using the *SSH process* running on `desmond@10.39.78.208`. Now if I connect to `localhost:7000` on my machine, the request will be tunnels through `10.39.78.208` toward `github.com` port `80`. You can try it with curl `localhost:7000`.
That is, the entrance of the tunnel is `:7000` (on my computer) and exit is `:80`. This is called **Local port tunnelling**.
We also have the **Remote Port tunnelling**. In remote port tunnelling (or reverse port forwarding) the traffic coming on a port on the remote server is forwarded to the SSH process running on your local host, and from there to the specified port on the destination server (which may also be your local machine).Let's have an example:
```sh
-> ~ ssh -R 7575:localhost:80 -Nf desmond@192.168.0.171
```
Here, I'm telling *ssh* to create a **Remote**tunnel. After this, any request on port `:7575` from `Desmond`'s computer will reach to local host (my machine) on port `:80`. This is useful when you want to let someone from outside your network access a service running on your local host machine.
**-N**: No login, just do the port forwarding(tunnel).
**f**: run *ssh* in the background.

Now that we know about local and remote port tunnelling, it will be easier to talk about **X11 Tunnels**. Through an X11 tunnel, the X Window System on the remote host is forwarded to your local machine. To forward the X, its enough to add a `-X` to your ssh. Please note that the `X11Forwarding yes` configuration should be present in your **sshd_config** file.

```sh
-> ~ ssh -X 192.168.0.171
```
From this, you can run graphical programs (say firefox) on the remote machine and see the graphical part on your own X server!.
- You can also use `-x` to disable X forwarding.

## Encrypting and signing data with **GPG**.
GPG is a free version of PGP. It can be used to *create keys*, *encrypt data*, *sign messages* etc. You can use GPG like SSH, just that with *GPG*, everything has to be done manually as opposed to *SSH* where most things are done automatically.
 There is an implementation of this method called `gpg` which can be used on Linux (and other machines) to perform these tasks. First you need to generate a key:
 ```sh
 -> ~ gpg --gen-key
 ```
 Now the key is created in the `~/.gnupg` directory and we can share our public key with others.
 Let us explain the use of each file generated:
 - `opengp-revocs.d`: The revocation certificate that was created along with the key pair is kept here. The permissions on this directory are quite restrictive as anyone who has access to the certificate could revoke the key.
- `private-keys-v1.d`: This is the directory that keeps your private keys, therefore permissions are restrictive.
- `pubring.kbx`: This is your public keyring. It stores your own as well as any other imported public keys.
- `trustdb.gpg`: he trust database. This has to do with the concept of Web of Trust(for more details, google is our friend ;)).
 You can now view your public keys:
 ```sh
 -> ~ gpg --list-keys
 ```
 This will display the `public keyring` content.
 The exported key is in *binary* format, add `-a` to armor it in ACSII.
 - You can check your fingerprint with the command `gpg --fingerprint USER-ID`.
 #### Key distribution and revocation.
 Now you have to distribute this key to others. You can upload it somewhere, email it, put it on your website or use some of the public key stores to share it with other:
 ```sh
 -> ~ gpg --export maxwell > maxwell.pub.key
 ```
 This will permit you to export your key to the file `maxwell.pub.key` in other to make it easier to share.
 - Adding a `-a` option after the `--export` will create ASCII armored output which can be safely emailed. 

 If anyone receives it, he/she can import it into his/her key store using the following command:
 ```sh
 -> ~ gpg --import maxwell.pub.key
 ```
 - You could also do:
 ```sh
 -> ~ scp maxwell.pub.key desmond@10.39.77.8
 ```
 *Desmond* is now in the possession of `maxwell.pub.key`. He will use it to encrypt a file and send it to `maxwell` in the next section.
 If it happens that my private key gets compromised, you have to create a **revoke** file and share it with others to tell them that your private key is not valid anymore:
```sh
-> ~ gpg --output maxwell.revoke.asc --gen-revoke maxwellele.@gmail.com
```
This tells gpg to create a *revoke file* called `maxwell.revoke.asc` for the identity *maxwellele@gmail.com*. If *maxwellele@gmail.com* needs to invalidate his private key, he has to publish this file to the Internet or key servers and others will know that the previous public key is not valid anymore since the associated private key was compromised.

#### Encrypting and Decrypting files.
Since *desmond* has already imported *Maxwell*'s key, *desmond* can now send private data to *Maxwell*:
```sh
-> ~ echo "Take this serious" > secret.txt
-> ~ gpg --output secret.txt.enc --recipient maxwellele@gmail.com -a --encrypt secret.txt
```
*Maxwell* can now open the encrypted file since he has the **private** key:
```sh
-> ~ gpg --out out.txt --decrypt file.txt.enc
```
- `--out`:creates a file.

#### Signing and Verifying files.
Desmond used Maxwell's Public key to encrypt and then Maxwell used his own Private Key to decrypt the data. But what happens if Maxwell uses his Private key on his side and lets others use his Public key to open the data? That is called **signing*

Please remember that only Maxwell has access to his private key. So if Maxwell applies it to a file, every one:

   - will be able to open the file using his Public Key
  - Will be sure that Maxwell has signed this because he is the only one having access to the private key of the public key they used to open the file.

Let's sign a message:
```sh
-> ~ echo "Freedom is a choice" > message.txt
-> ~ gpg --output message.sig --sign message.txt
```
**NB**: Using `--sign` the document is compressed and then signed. The output is in binary format.

Now Maxwell can share the  `message.sig` file to the internet or to `Desmond`.
If they want to make sure that it is truely comming from Maxwell, it's enough for them to check his signature (obviously after importing his public key):
```sh
-> ~ gpg --verify message.sig
```
 If they also needed to decrypt the message they could do as follow:
 ```sh
 -> ~ gpg --ouput output.txt --decrypt --clearsign message.sig
 ```
 - The `--clearsign` option will create a file ending in `.asc` which containt the original clear-text message alongside the signature.

 #### gpg-agent
 Just like the **ssh-agent**, **gpg-agent** is a tool(daemon) which acts like a password manager for your gpg keys. It keeps the keys in the memory so you won't need to provide password on every single use. 
 - To view a summary of the most useful options, run `gpg-agent --help` or `gpg-agent -h`.

This marks the end of **Security and Data Encryption**










  


   

