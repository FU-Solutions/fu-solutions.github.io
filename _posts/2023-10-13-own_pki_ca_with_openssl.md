---
title: Howto - Own CA for Homelab with OpenSSL
author: w3ich3rt
date: 2023-10-12 20:00:00 +0100
categories: [Linux]
tags: [openssl, ssl, linux, howto, ca, certificates, secure, security, certificate, homelab]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
---

# General

We all know that, we are trying out new products, software or something else in our little homelab. Since nowadays pretty much everything comes with a certificate, you are constantly running into the "Insecure" in the browser and have to set up the exception (on each client) for each new service.
On the Internet, using Let's Encrypt is a fine thing, but at home in the Homelab, it's not quite so simple. There are products like [LabCA](https://github.com/hakwerk/labca) or [step-ca](https://smallstep.com/docs/step-ca/), with which one could set up a local ACME variant, but that is also not always purposeful or simply goes beyond the scope :-)
I would like to show here the creation of a root CA infrastructure (for certificates) with OpenSSL, as it is perfectly sufficient for my Homelab. Maybe we will deal with a real PKI infrastructure later on, but I don't know yet.

> At this point I will spare us to explain how certificates work, there are tons of documents on the internet

## Step-by-Step

In summary, we need to proceed in six steps for a functioning certificate structure with root CA and certificates.

1. Create a private key for our own CA
1. Create a certificate for the CA
1. Add this certificate to the “Trusted Root Certificate Authorities” store of the clients so that it becomes trusted
1. Create a certificate for our webserver
1. Sign this certificate with our CA (which is trusted and therefore, also this new certificate becomes trusted)
1. Deploy the certificate

After performing these steps, the security notices for unsafe websites should be a thing of the past. 

## Creating privatekey for CA

To create the private key for the CA you should can use `openssl` with the following parameters. Because this key is very important and should be handled with some kind of security (anyone who get this key could create certificates for your root-ca), we'll encrypt the key with AES and set a strong password as well.

```shell
CA=MyName-RootCA
# generate aes encrypted private key
openssl genrsa -aes256 -out $CA.key 4096
```

After creating the CA-keyfile we can create the certificate of the CA

```shell
# create certificate, 1826 days = 5 years
# the following will ask for common name, country, ...
openssl req -x509 -new -nodes -key $CANAME.key -sha256 -days 1826 -out $CANAME.crt
# ... or you provide common name, country etc. via:
openssl req -x509 -new -nodes -key $CANAME.key -sha256 -days 1826 -out $CANAME.crt -subj '/CN=MyOrg Root CA/C=DE/ST=Syke/L=Lower-Saxony/O=MyOrg'
```

Of course, you can set the validity period of the certificate longer than 5 years. Certainly feasible in the Homelab ;-)

The created root-CA certificate can now be installed on all the clients (Windows, Linux or Mobilephone), so that all certificates signed with this CA certificate are classified as trusted.

## Create a certificate for a webserver

```shell
MYCERT=servername.home.lab
openssl req -new -nodes -out $MYCERT.csr -newkey rsa:4096 -keyout $MYCERT.key -subj '/CN=SomeCommonName/C=DE/ST=Lower-Saxony/L=Syke/O=FU-Solutions'
# create a v3 ext file for SAN properties
cat > $MYCERT.v3.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = servername.home.lab
DNS.2 = servername1.home.lab
IP.1 = 192.168.1.1
IP.2 = 192.168.2.1
EOF
```

The v3.ext file contains the properties of the v3 extension of certificates. In particular, this includes the SAN (subject alternative names), which contains the information about DNS or IP that the browser needs to trust the certificate (you must somehow make sure that mysite.local uses the certificate issued for mysite.local). Okay now we can sign our certificate:

```shell
openssl x509 -req -in $MYCERT.csr -CA $CANAME.crt -CAkey $CANAME.key -CAcreateserial -out $MYCERT.crt -days 730 -sha256 -extfile $MYCERT.v3.ext
```

It's done... finally we got our certificate and can deploy it.
