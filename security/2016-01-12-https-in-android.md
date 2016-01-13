# How does Https work?

This post comes from my share with my team. Many people may not understand HTTPS well, and why the https codes is like that? So I want to share it into Internet too.

## Cipher
Java 1.2 has imported a system called "JCE"(Java Cryptography Extension), which is in charge of key and certificates in Java.

We all know if we want to encrypt or decrypt some information, I have to have a key. It's like if you want to open a door, or lock a door, you have to have a key. 

The Key can be generated in Java with *KeyGenerator* or *KeyPairGenerator*. The former one is used to generate the symmetric key, and the latter one is used to generate the asymmetric key. 

* symmetric cryptography <br/>
    -- the encryption and decryption use a same key.

* asymmetric cryptography <br/>
   -- the encryption and decryption use different keys. Keys are usually is generated as a **public key** and a **private key**. <br/>
   The public key may be widely distributed, while the private key is known only to its proprietor. <br/>
   In a secure asymmetric key encryption scheme, when a message is encrypted with the public key, only the private key can decrypt it. So if a hacker get you public-key-encrypted message, he cannot decrypt it, because he has no the paired private key. So the transportation of this message is secure. <br/>


## Certificate

In a real world, if you want to buy a diamond and enter a diamond shop, how do you to tell whether the diamond is genuine. As a normal person, you may not understand the diamond knowledge. But if the diamond has a licence that is issued by the US government, you may trust this shop.

Certificate is the same. Certificate in the computer world, it may have some keys, another certificate (let's call it "B") . The key is what I need, and "B" is an license to prove this certificate is trustable. 

Another question: How can I be sure that "B" is trustable? Oh, that's a good question. 

Android has put nearly 150 CA root certificate in our phone. These 150 certificates are really trusted by the whole world, and they are like the chief justice in U.S.

"B" has a another certificate (let's call it "C") inside, so we will go check "C" to see if "C" is trustable.  ...  Going through this certificate chain, if we found the last or the root certificate is the same certificate in the 150 pre-put certificates in the phone, then we can trust the original certificate. 

p.s. : Certificate has many format. 
* "x.509"
: x.509 certificate is usually used to contain a public key.

* "PKCS12"
: PKCS12 certificate can also be used to contain a private key. Hence, the PKCS12 certificate need a password to open it. 


## Https
Finally, we now reach the "https" part. Https uses the above "cipher" and "certificate" part, that's why I talked them before.

Https(Http over SSL) is designed for secure communication over Internet. 

### 1. How to communicate safely?
First thing comes to my head is cipher. I encrypt my data, and pass the data and the key to the server. Then the server can use this key I passed to him to decrypte the encrypted messages.
![](/imgs/20160113_01.jpg)

But if some bad guys intercept this communication, then they can decrypt the message too. This is a problem?

### 2. How about Asymmetric Cryptography?
This is a great idea. The sever gives you its public key. You use the public Key to encrypt the messages.  Since the server is the only one who has the private key, which means only this server can decrypt your encrypted messages. Even if the hackers intercept the communication, hackers can do nothing because they have the corresponding private key.

But, asymmetric cryptography is a little longer than symmetric cryptography. For the user experience's sake, can we improve it based on this?

### 3. the final scheme
![](/imgs/20160113_02.jpg)
The picture above has clearly shown the process of HTTPS.
1. [Server] generate the key pair, with a public key and a private key. Let's call them as "KeyPub", "KeyPri". 
2. [Server] server pass the "KeyPub" to the client
3. [Client] generate a symmetric key (Let's call it "key2"), and encrypt the messages with "key2"
4. [Client] encrypt the "key2" with "KeyPub". So our "key2" is safe, since only the server has the "keyPri".
5. [Client] pass the encrypted data and encrypted key to the server
6. [Server] Decrypt the key with "KeyPri", and get the "key2"
7. [Server] Decrypt the data with "key2". Now the data arrieve in Server safely.


Since the symmetric cryptography is faster than asymmetric cryptography, https decides to use symmetric cryptography to encrypt the data, and use asymmetric cryptography to encrypt the symmetric key.  In this way, the encryption is fast and secure. 

