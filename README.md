## Contents:
* [Create the Root CA Certificate](https://github.com/Bodayngo/openssl/blob/main/README.md#create-the-root-ca-certificate)
* [Create the Intermediate Certificate](https://github.com/Bodayngo/openssl/blob/main/README.md#create-the-intermediate-ca-certificate)
* Creating the Server/Client Certificates

## Create the Root CA Certificate
Create a root directory in which you will store your CA private key and certificate. We will also create various other sub-directories needed for this process as well as the index.txt and serial files which used to keep track of signed certificates.
```
mkdir /root/ca
cd /root/ca
mkdir certs crl newcerts private
chmod 700 /root/ca/private
touch /root/ca/index.txt
echo 1000 > /root/ca/serial
```
Create OpenSSL CA configuration file. Copy the contents of [this](https://github.com/Bodayngo/openssl/blob/main/example-root-ca-config) example configuration file. Optionally, edit the defaults beneath [req_distinguished_name].
```
nano /root/ca/openssl.cnf
```
Create the root CA private key.
```
openssl genrsa -aes256 -out /root/ca/private/ca.key.pem 4096
```
```
chmod 400 /root/ca/private/ca.key.pem
```
Create the root CA certificate.
```
openssl req -config /root/ca/openssl.cnf \
      -key /root/ca/private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out /root/ca/certs/ca.cert.pem
```
```
chmod 444 /root/ca/certs/ca.cert.pem
```
Validate the contents of the root CA certificate file.
```
openssl x509 -noout -text -in /root/ca/certs/ca.cert.pem
```
## Create the Intermediate CA Certificate
Create a directory in which you will store the rest of your private keys and certificates. Like before, we will also create various other sub-directories needed for this process as well as the index.txt and serial files which used to keep track of signed certificates
```
mkdir /root/ca/intermediate
cd /root/ca/intermediate
mkdir certs crl csr newcerts private
chmod 700 /root/ca/intermediate/private
touch /root/ca/intermediate/index.txt
echo 1000 > /root/ca/intermediate/serial
echo 1000 > /root/ca/intermediate/crlnumber
```
Create OpenSSL intermediate configuration file. Copy the contents of [this]() example configuration file. As before, you can optionally edit the defaults beneath [req_distinguished_name].
