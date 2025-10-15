Certificate Authority
====================

## [Introduction](https://jamielinux.com/docs/openssl-certificate-authority/introduction.html)

OpenSSL is a free and open-source cryptographic library that provides several command-line tools for handling digital certificates. Some of these tools can be used to act as a certificate authority.

A certificate authority (CA) is an entity that signs digital certificates. Many websites need to let their customers know that the connection is secure, so they pay an internationally trusted CA (eg, VeriSign, DigiCert) to sign a certificate for their domain.

In some cases it may make more sense to act as your own CA, rather than paying a CA like DigiCert. Common cases include securing an intranet website, or for issuing certificates to clients to allow them to authenticate to a server (eg, Apache, OpenVPN).


## How-to

1. **Init**
* Create Root CA pair
* Create the intermediate pair
* Sign intermediate pair

```
# git clone --depth=1 git@github.com:quick-sort/CA-template.git
# cd CA-template
# ./init.sh
```

2. **Sign server and client certificates**

We will be signing certificates using our intermediate CA. You can use these signed certificates in a variety of situations, such as to secure connections to a web server or to authenticate clients connecting to a service.

2.1 Create a key

Our root and intermediate pairs are 4096 bits. Server and client certificates normally expire after one year, so we can safely use 2048 bits instead.

```
# openssl genrsa -aes256 -out intermediate/private/www.example.com.key.pem 2048
# chmod 400 intermediate/private/www.example.com.key.pem
```

2.2 Create a certificate

Use the private key to create a certificate signing request (CSR). The CSR details don’t need to match the intermediate CA. For server certificates, the Common Name must be a fully qualified domain name (eg, www.example.com), whereas for client certificates it can be any unique identifier (eg, an e-mail address). Note that the Common Name cannot be the same as either your root or intermediate certificate.

```
# openssl req -config intermediate/openssl.cnf \
      -key intermediate/private/www.example.com.key.pem \
      -new -sha256 -out intermediate/csr/www.example.com.csr.pem
```

To create a certificate, use the intermediate CA to sign the CSR. If the certificate is going to be used on a **server**, use the `server_cert` extension. If the certificate is going to be used for **user authentication**, use the `usr_cert` extension. Certificates are usually given a validity of one year, though a CA will typically give a few days extra for convenience.

```
# openssl ca -config intermediate/openssl.cnf \
      -extensions server_cert -days 375 -notext -md sha256 \
      -in intermediate/csr/www.example.com.csr.pem \
      -out intermediate/certs/www.example.com.cert.pem

# chmod 444 intermediate/certs/www.example.com.cert.pem
```

2.3 Verify the certificate

```
# openssl x509 -noout -text -in intermediate/certs/www.example.com.cert.pem
# openssl verify -CAfile intermediate/certs/ca-chain.cert.pem \
      intermediate/certs/www.example.com.cert.pem
```

2.4 Deploy the certificate

You can now either deploy your new certificate to a server, or distribute the certificate to a client. When deploying to a server application (eg, Apache), you need to make the following files available:
```
ca-chain.cert.pem
www.example.com.key.pem
www.example.com.cert.pem
```

