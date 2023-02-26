## Create Root CA Certificate
Create a root directory in which you will store your CA private keys and certificates. We will also create various other sub-directories needed for this process as well as the index.txt and serial files which used to keep track of signed certificates.
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
