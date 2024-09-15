---
title: "OpenSSL Cheat Sheet"
date: 2024-09-15T17:41:12+01:00
slug: 2024-09-15-openssl
type: posts
draft: true
categories:
  - openssl
tags:
  - certificates
---
Get things done with OpenSSL

## Private Key
Generate (old devices don't like keys longer than 2048)

`openssl genrsa -aes128 -out example.key 2048`

Remove password protection

`openssl rsa -in example.key -out example_open.key`

## Certificate Generation
CSR template file (public CAs don't accept IP alternative names)

```
[ req ]
prompt = no
distinguished_name = dn
req_extensions = req_ext

[ dn ]
CN = example.com
emailAddress = me@example.com
O = Corp.
OU = Geek
L = City
ST = State
C = Country

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = example.com
IP.1 = 10.11.12.13
```
 
Generate the CSR from the template

`openssl req -new -out example.csr -key example.key -config example.conf`

### Self signed (Root CA)
Generate the CSR

`openssl req -x509 -nodes -sha256 -days 730 -newkey rsa:2048 -keyout my.key -out my.pem`

Sign the cerficate with the same key as the CSR

`openssl x509 -in test.csr -out test.pem -req -signkey test.key -days 1000 -extensions 'req_ext'`
 
## Format Conversions

PEM to PKCS12 

`openssl pkcs12 -export -out example.pfx -inkey example.key -in example.pem -certfile issuer.pem -certfile root.pem`

PKCS7 to PEM

`openssl pkcs7 -print_certs -in primary_chain.p7b -out  primary_chain.cer`

DER to PEM

`openssl x509 -inform der -in cert.cer -out cert.pem`

PKCS12 to PEM

`openssl pkcs12 -in test.p12 -out test.pem -nodes`

## Server Analysis
Server certificate

`openssl s_client -serverinfo -connect www.example.com:443 < /dev/null | openssl x509 -noout -text`

Fetch server PEM certiticate

`openssl s_client -connect www.example.com:443 < /dev/null 2>/dev/null| sed -n '/-----BEGIN/,/-----END/p'`

Get full certificate chain

`openssl s_client -connect www.example.com:443 -showcerts < /dev/null 2>/dev/null`

## Certificate Validation
Get OCSP URI

`openssl s_client -connect www.example.com:443 < /dev/null 2>/dev/null| sed -n '/-----BEGIN/,/-----END/p' | openssl x509 -noout -ocsp_uri`

OCSP check

`openssl ocsp -issuer issuer.pem -cert cert.pem -text -url http://OCSP_URI`

Check against CRL file

`openssl crl -inform DER -noout -text -in mycrl.crl  | less`
 
Check against online CRL

`curl http://CRL_URL | openssl crl -inform DER -noout -text | less`


## File Encryption

Arguments:
* `-aes-256-cbc` : cipher
* `-pbkdf2` : Use PBKDF2 algorithm with default iteration count
* `-base64` : encode encryption result base64

Encrypt

`openssl enc -aes-256-cbc -pbkdf2 -base64 -in original_file.txt -out encrypted_file.txt`

Decrypt

`openssl enc -d -aes-256-cbc -pbkdf2 -base64 -in encrypted_file.txt -out decrypted_file.txt`
