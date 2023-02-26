# openssl

## Create Root CA Certificate
### Prepare root CA directory structure
Create a root directory in which you will store your CA private keys and certificates. We will also create various other sub-directories needed for this process as well as the index.txt and serial files which used to keep track of signed certificates.
```
mkdir /root/ca
cd /root/ca
mkdir certs crl newcerts private
chmod 700 /root/ca/private
touch /root/ca/index.txt
echo 1000 > /root/ca/serial
```
### Create root CA OpenSSL configuration file
Create OpenSSL CA configuration file. Copy the contents of this example configuration file. Optionally, edit the defaults beneath [req_distinguished_name].
```
nano /root/ca/openssl.cnf
```
