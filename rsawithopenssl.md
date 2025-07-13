# 0. Introduction/Setup.
RSA is one of the oldest asymmetric-key cryptography algorithms. It was created by Ron Rivest, Adi Shamir, and Leonard Adleman in 1977. Symmetric encryption algorithms work by applying a single key to encrypt and decrypt data. However, asymmetric algorithms use a pair of two keys. One private key which must be kept secret, and a public key which is derived from the private key and may be shared with anyone and everyone. Either key can be used to encrypt a message, with the other key being used to decrypt said message. Oftentimes the private key is used to "sign" a message, allowing anybody with the public key to verify that the message was sent by the holder of the private key and hasn't been modified since. Usually this is accomplished via hashing the message and encrypting the hash. If you don't know what cryptographic hashing is, there are plenty of online resources that can explain it. Another use of RSA is using the public key to encrypt a message, and then sending it to the holder of the private key. This way, only the holder of the private key can read the message. Most often this is used to encrypt and transfer a symmetric key so that the two parties can communicate without anybody being able to eavesdrop. RSA is still considered to be a secure algorithm, however quantum computers will likely be able to break it in the future.

Note that this tutorial assumes you're working in a Linux/Unix environment. All commands featured in this tutorial are to be run in one's terminal application. One will need openssl to be installed, though it is likely to be installed by default in most Linux distributions.

# 1. Generating a private key.
The following command will generate and output an RSA private key. It is very important that this key is kept in a secure environment with the fewest possible people being allowed to access it. More information is given below about how to store the key in a more secure fashion. Note that the "4096" in the following command represents the key size in bits. A key size of 2048 bits may also be used if performance is an issue, but NIST recommends this only until the year 2030. A key size of 1024 bits has been standard previously but is no longer recommended or even supported by some applications such as web browsers. I would recommend using a keysize of 4096 in order to be "future proof." RSA keys are often used in SSL certificates to secure web traffic, and Certificate Authorities typically support up to 4096 bit keys. 3072 bit keys are also possible if one is so inclined, and provide security beyond the year 2030. Larger key sizes such as 7680 and 15360 bits aren't as likely to be supported and have an even greater performance impact.

>openssl genrsa -out private.pem 4096

To prevent other users in a Linux environment (aside from the root user and/or your user) from reading the private key, you may run the following command:

>chmod 600 private.pem

It is also possible to encrypt the private key using openssl when one generates it. You will be prompted for a passphrase, and the key will be outputted encrypted. Most tutorials use triple DES for this, but I strongly recommend not doing that and using a secure algorithm such as AES as shown below. Triple DES has been deprecated by NIST and replaced with AES. I use AES256, but other key sizes are supported and may also be used such as AES128 or AES192. Running "openssl --help" will reveal which algorithms and key sizes are supported by your openssl instance.

>openssl genrsa -aes256 -out private.pem 4096

# 2. Generating a public key.
The following command will generate a public key which is paired with the private key. It takes the private key as an input because the public key is mathematically derived from the private key. There is no need to hide or secure the public key, as there shouldn't be any harm in someone being able to read it. 

>openssl rsa -in private.pem -pubout -out public.pem

# 3. Creating a self-signed x509 certificate.
x509 certificates are a standard type of digital certificate which are used by websites in order to establish an encrypted connection with your web browser, among other things. These certificates provide the public key of the entity you're communicating with, and are often signed by a CA (Certificate Authority) which are trusted organizations that help to ensure certificates are only issued to the actual website owner. This way, an attacker cannot simply present their own public key and impersonate a website as part of a Man-In-The-Middle attack. For this tutorial, the certificate will not be signed by a CA but rather will be a self-signed certificate. One's own private key will be used to sign the certificate. Therefore, this certificate won't be trusted by web browsers, they will display a warning which will probably scare away many users. The certificate will still function and encrypt web traffic if the user proceeds, however, making it not wholly useless. I might make another tutorial later on showing how to install a self signed certificate on a webserver in order to actually use it.

Note that the certificate expires in 199 days. The standard for a certificate's maximum lifetime is 398 days currently, after which browsers and software oftentimes consider a certificate invalid even if one has set a higher expiry time. On March 15th, 2026 the maximum lifetime will be lowered to 200 days, however. On March 15th, 2027 it will be lowered to 100 days, and on March 15th, 2029 it will be lowered all the way down to 47 days. Since March 15th, 2026 is less than a year away as of the time this tutorial is being written, a certificate set to expire in about a year may be considered invalid before then so setting it in accordance with the 200 day standard is safer.

>openssl req -key private.pem -new -x509 -days 199 -out mycert.crt

# Sources.
https://en.wikipedia.org/wiki/RSA_cryptosystem

https://docs.openssl.org/3.0/man1/openssl-rsa/

https://community.rsa.com/s/article/RSA-cryptography-and-NIST-guidance

https://stackoverflow.com/questions/589834/what-rsa-key-length-should-i-use-for-my-ssl-certificates

https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs

https://www.ssl.com/guide/ssl-certificate-expiration-guide/

https://www.digicert.com/blog/tls-certificate-lifetimes-will-officially-reduce-to-47-days
