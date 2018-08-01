# OpenSSL Certificate Authority

Create the Root CA and the Intermediate CA

## Table of Contents

- [Create the Root CA](#create-the-root-ca)
- [Create the Intermediate CA](#create-the-intermediate-ca)
- [Create the keystore PKCS12](#create-the-keystore-pkcs12)
- [License](#license)



### Prerequisites

You need to install  openssl

```
$ openssl version -a

```

## Create the Root CA

### Create the directory

```
$ mkdir ca
$ cd ca
$ mkdir certs crl newcerts private pfx
$ chmod 700 private
$ touch index.txt
$ echo 1000 > serial
```
### Configuration OpenSSL file

Copy the Root CA configuration file to [YOUR_PATH]/ca/openssl.cnf. You must change the directory and file locations.
```
# Directory and file locations.
dir               = [YOUR_PATH]/ca
```
* [Download](ca/openssl.cnf) - Standar configuration file

### Create the Root Key
```
$ openssl genrsa -aes256 -out private/ca.key.pem 4096
$ chmod 400 private/ca.key.pem
```

### Create the Root Certificate
```
$ openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem
$ chmod 444 certs/ca.cert.pem
```

### Check Root certificate
```
$ openssl x509 -noout -text -in certs/ca.cert.pem
```


## Create the Intermediate CA

### Create the directory

```
$ mkdir ca/intermediate
$ cd ca/intermediate
$ mkdir certs crl newcerts private pfx
$ chmod 700 private
$ touch index.txt
$ echo 1000 > serial
```

### Configuration OpenSSL file

Copy the Intermediate CA configuration file to [YOUR_PATH]/ca/intermediate/openssl.cnf. You must change the directory and file locations.
```
# Directory and file locations.
dir               = [YOUR_PATH]/ca/intermediate
```
* [Download](ca/intermediate/openssl.cnf) - Standar configuration file


### Create the Intermediate Key
```
$ openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096
$ chmod 400 intermediate/private/ca.key.pem
```

### Create the Intermediate Certificate

We use the root CA with the v3_intermediate_ca extension to sign the intermediate CSR (other extension usr_cert or server_cert)

```
$ openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem
$ openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem
$ chmod 444 intermediate/certs/intermediate.cert.pem
```

### Check Intermediate certificate
```
$ openssl verify -CAfile certs/ca.cert.pem intermediate/certs/intermediate.cert.pem
```

### Create the certificate chain file
```
$ cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
$ chmod 444 intermediate/certs/ca-chain.cert.pem
```

## Create the keystore PKCS12
```
$ openssl pkcs12 -inkey intermediate/private/intermediate.key.pem -in intermediate/certs/ca-chain.cert.pem -export -name [YOUR_ALIAS] -out intermediate/pfx/intermediate.pfx.pem
```

### Convert the keystore (PKCS12) to Java Keystore (jks)

```
$ keytool -importkeystore -destkeystore keystore.jks -srckeystore intermediate/pfx/intermediate.pfx.pem -srcstoretype pkcs12 -alias [YOUR_ALIAS]

```

## License

This project is licensed under the GNU GPLv3 - see the [LICENSE.md](LICENSE.md) file for details

