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

Enter pass phrase for ca.key.pem: password
Verifying - Enter pass phrase for ca.key.pem: password

chmod 400 /root/ca/private/ca.key.pem
```
Create the root CA certificate.
```
openssl req -config /root/ca/openssl.cnf \
      -key /root/ca/private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out /root/ca/certs/ca.cert.pem

Enter pass phrase for /root/ca/private/ca.key.pem: password
You are about to be asked to enter information that will be incorporated
into your certificate request.
-----
Country Name (2 letter code) [US]:
State or Province Name [North Dakota]:
Locality Name [West Fargo]:
Organization Name [Lab Org]:
Organizational Unit Name [Lab]:
Common Name []:Lab OpenSSL Root CA
Email Address [lab@lab.local]:

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
Create OpenSSL intermediate configuration file. Copy the contents of [this](https://github.com/Bodayngo/openssl/blob/main/example-intermediate-ca-config) example configuration file. As before, you can optionally edit the defaults beneath [req_distinguished_name].
```
nano /root/ca/intermediate/openssl.cnf
```
Generate the intermediate private key.
```
openssl genrsa -aes256 \
      -out /root/ca/intermediate/private/intermediate.key.pem 4096

Enter pass phrase for intermediate.key.pem: password
Verifying - Enter pass phrase for intermediate.key.pem: password

chmod 400 /root/ca/intermediate/private/intermediate.key.pem
```
Create the intermediate CSR.
```
openssl req -config /root/ca/intermediate/openssl.cnf -new -sha256 \
      -key /root/ca/intermediate/private/intermediate.key.pem \
      -out /root/ca/intermediate/csr/intermediate.csr.pem

Enter pass phrase for intermediate.key.pem: password
You are about to be asked to enter information that will be incorporated
into your certificate request.
-----
Country Name (2 letter code) [US]:
State or Province Name [North Dakota]:
Locality Name [West Fargo]:
Organization Name [Lab Org]:
Organizational Unit Name [Lab]:
Common Name []:Lab OpenSSL Intermediate CA
Email Address [lab@lab.local]:
```
Sign the intermediate CSR to create the intermediate certificate.
```
openssl ca -config /root/ca/openssl.cnf -extensions v3_intermediate_ca \
      -days 3650 -notext -md sha256 \
      -in /root/ca/intermediate/csr/intermediate.csr.pem \
      -out /root/ca/intermediate/certs/intermediate.cert.pem

Enter pass phrase for /root/ca/private/ca.key.pem: password
Sign the certificate? [y/n]: y
1 out of 1 certificate requests certified, commit? [y/n]y

chmod 444 /root/ca/intermediate/certs/intermediate.cert.pem
```
Validate the contents of the intermediate certificate and and verify it against the root certificate.
```
openssl x509 -noout -text \
      -in /root/ca/intermediate/certs/intermediate.cert.pem

openssl verify -CAfile /root/ca/certs/ca.cert.pem \
      /root/ca/intermediate/certs/intermediate.cert.pem

intermediate.cert.pem: OK
```
Create the certificate chain.
```
cat /root/ca/intermediate/certs/intermediate.cert.pem \
      /root/ca/certs/ca.cert.pem > \
      /root/ca/intermediate/certs/ca-chain.cert.pem

chmod 444 /root/ca/intermediate/certs/ca-chain.cert.pem
```
## Create the Server/Client Certificates
Generate the server/client private key.
```
openssl genrsa -aes256 \
      -out /root/ca/intermediate/private/hostname.lab.local.key.pem 4096

chmod 400 /root/ca/intermediate/private/hostname.lab.local.key.pem
```
Generate the server/client CSR.
```
openssl req -config /root/ca/intermediate/openssl.cnf \
      -key /root/ca/intermediate/private/hostname.lab.local.key.pem \
      -new -sha256 -out /root/ca/intermediate/csr/hostname.lab.local.csr.pem

Enter pass phrase for hostname.lab.local.key.pem: password
You are about to be asked to enter information that will be incorporated
into your certificate request.
-----
Country Name (2 letter code) [US]:
State or Province Name [North Dakota]:
Locality Name [West Fargo]:
Organization Name [Lab Org]:
Organizational Unit Name [Lab]:
Common Name []:hostname.lab.local
Email Address [lab@lab.local]:
```
Sign the RADIUS server CSR with the intermediate CA.
* Use the server_cert extenstion when signing a server certificate
* Use the usr_cert extension when signing a client certificate
```
openssl ca -config /root/ca/intermediate/openssl.cnf \
      -extensions server_cert -days 375 -notext -md sha256 \
      -in /root/ca/intermediate/csr/hostname.lab.local.csr.pem \
      -out /root/ca/intermediate/certs/hostname.lab.local.cert.pem

Enter pass phrase for /root/ca/intermediate/private/intermediate.key.pem: password
Sign the certificate? [y/n]: y
1 out of 1 certificate requests certified, commit? [y/n]y

chmod 444 /root/ca/intermediate/certs/hostname.lab.local.cert.pem
```
Validate the contents of the RADIUS server certificate and verify it against the previously created CA certificate chain.
```
openssl x509 -noout -text \
      -in /root/ca/intermediate/certs/hostname.lab.local.cert.pem

openssl verify -CAfile /root/ca/intermediate/certs/ca-chain.cert.pem \
      /root/ca/intermediate/certs/hostname.lab.local.cert.pem

hostname.lab.local.cert.pem: OK
```
