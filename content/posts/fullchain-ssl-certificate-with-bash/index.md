---
authors:
  - Burak Berk
title: Creating Fullchain SSL Certificate With Bash
description: Become a local CA for development or self-hosted applications.
date: 2023-07-18
tags:
  - 'ssl'
  - 'ca'
  - 'self-signed'
categories:
  - 'DevOps'
  - 'System Administration'
---

## Overview

After you follow this guide, you will became a local certificate authority that signs certs for web applications.

You will follow this steps:

- Create a rootCA key and crt.
- Create a intermediateCA (subCA) key and crt that is signed by rootCA.
- Create a application key and crt that is signed by intermediateCA.
- After all, you will have root, sub and app (fullchain) cert and key that is ready to be used in nginx or other stuf.

## Before Starting

- Create a localCA folder in /etc/ssl

```bash
sudo mkdir -p /etc/ssl/localCA && \
cd /etc/ssl/localCA
```

- For non sudo commands, chown the directory.

```bash
sudo chown -R $USER:$USER /etc/ssl/localCA
```

---

## Create Root CA

- Create a folder inside `/etc/ssl/localCA` named `rootCA`

```bash
mkdir -p /etc/ssl/localCA/rootCA && \
cd /etc/ssl/localCA/rootCA
```

- Create rootCA.key file

```bash
openssl genrsa -aes256 -out rootCA.key 4096
```

- Create a cnf file named `rootCA.cnf`

```bash
vim rootCA.cnf
```

```bash
[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /etc/ssl/localCA/rootCA
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/rootCA.key
certificate       = $dir/rootCA.crt

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 3650
preserve          = no
policy            = policy_strict

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name 2 Letter
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = TR
stateOrProvinceName_default     = Istanbul
localityName_default            = Istanbul
0.organizationName_default      = Safderun
organizationalUnitName_default  = Safderun ROOT CA
commonName_default              = Safderun ROOT CA
emailAddress_default            = burakberkkeskin@gmail.com

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

> **Caution ⚠️**
>
> Change the values lines in the rootCA.cnf file above:
>
> - countryName_default
> - stateOrProvinceName_default
> - localityName_default
> - 0.organizationName_default
> - organizationalUnitName_default
> - commonName_default
> - emailAddress_default

- Create the rootCA.crt file

```bash
openssl req -config rootCA.cnf \
            -key rootCA.key \
            -new -x509 -days 7305 -sha256 -extensions v3_ca \
            -out rootCA.crt
```

- Create a `newcerts` directory

```bash
mkdir /etc/ssl/localCA/rootCA/newcerts
```

- Create a `index.txt` file

```bash
touch /etc/ssl/localCA/rootCA/index.txt && \
chmod 600 /etc/ssl/localCA/rootCA/index.txt
```

- Create a `serial` file

```bash
echo "1000" > /etc/ssl/localCA/rootCA/serial && \
chmod 600 /etc/ssl/localCA/rootCA/serial
```

---

## Create Intermediate CA

- Create a intermediateCA folder

```bash
mkdir -p /etc/ssl/localCA/intermediateCA && \
cd /etc/ssl/localCA/intermediateCA
```

- Create intermediateCA.key file

```bash
openssl genrsa -aes256 -out intermediateCA.key 4096
```

- Create a `intermediateCA.cnf` file

```bash
vim intermediateCA.cnf
```

```bash
[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /root/ca/intermediate
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_loose

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name 2 Letter
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = TR
stateOrProvinceName_default     = Istanbul
localityName_default            = Istanbul
0.organizationName_default      = Safderun
organizationalUnitName_default  = Safderun Intermediate CA
commonName_default              = Safderun Intermediate CA
emailAddress_default            = burakberkkeskin@gmail.com

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

> **Caution ⚠️**
>
> Change the values lines in the intermediateCA.cnf file above:
>
> - countryName_default
> - stateOrProvinceName_default
> - localityName_default
> - 0.organizationName_default
> - organizationalUnitName_default
> - commonName_default
> - emailAddress_default

- Create intermediateCA.csr file

```bash
openssl req -config intermediateCA.cnf \
			-new -sha256 \
			-keyout intermediateCA.key \
			-out intermediateCA.csr
```

- Sign the intermediateCA.csr with rootCA.key file

```bash
openssl ca -config ../rootCA/rootCA.cnf \
			-extensions v3_intermediate_ca \
			-days 3650 -notext -md sha256 \
			-in intermediateCA.csr \
			-out intermediateCA.crt
```

---

## Creating a SSL cert for your web app with domain

- Create a directory named your application

```bash
export appname=exampleApp && \
mkdir -p /etc/ssl/localCA/$appname && \
cd /etc/ssl/localCA/$appname
```

- Create a `exampleApp.key` file

```bash
openssl genpkey -algorithm RSA -out "${appname}.key"
```

- Create a `exampleApp.cnf` file

```bash
vim "${appname}.cnf"
```

```bash
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn

[ dn ]
countryName            = TR
stateOrProvinceName    = Istanbul
localityName           = Istanbul
organizationName       = Safderun
organizationalUnitName = Safderun Webapp
commonName             = example.com
emailAddress           = burakberkkeskin@gmail.com

[ v3_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = api.example.com
DNS.2 = app.example.com
```

> [!caution]
> Change the values lines in the exampleApp.cnf file above:
>
> - countryName_default
> - stateOrProvinceName_default
> - localityName_default
> - 0.organizationName_default
> - organizationalUnitName_default
> - commonName_default
> - emailAddress_default
> - DNS.1

- Create a `exampleApp.csr` file

```bash
openssl req -new -key "${appname}.key" \
                 -config "${appname}.cnf" \
                 -out "${appname}.csr"
```

- Sign the `exampleApp.csr` with `intermediateCA.key`

```bash
openssl x509 -req -in "${appname}.csr" \
			-CA ../intermediateCA/intermediateCA.crt \
			-CAkey ../intermediateCA/intermediateCA.key \
			-CAcreateserial \
			-days 365 -extensions v3_ext \
			-extfile "${appname}.cnf" \
			-out "${appname}.crt"
```

- Create Fullchain Cert

```bash
cat ./${appname}.crt >> fullchain.crt && \
cat ../intermediateCA/intermediateCA.crt >> fullchain.crt && \
cat ../rootCA/rootCA.crt >> fullchain.crt
```

---

## The Final

You have 3 folder. rootCA, intermediateCA and application directory. In the application directory, you have 5 files.

- exampleApp.cnf: cert conf file
- exampleApp.crt: public cert file
- exampleApp.csr: cert signing request file
- exampleApp.key: private key
- fullchain.crt: Contains rootCA, intermediateCA and app cert which is ready to use.

You can use the fullchain cert and key file in a nginx server.

You can keep creating new certs for new web applications. All you need to do is repeating [this step](#creating-a-ssl-cert-for-your-web-app-with-domain)
