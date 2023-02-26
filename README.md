# openssl

##Create Root Certificate
###Prepare root CA directory structure
"""
mkdir /root/ca
cd /root/ca
mkdir certs crl newcerts private
chmod 700 /root/ca/private
touch /root/ca/index.txt
echo 1000 > /root/ca/serial
"""
