
## Creating directories, index and serial files

mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private


cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl

# Creating CA request

openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key

# Creating CA certificate

openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca.csr \
    -out ca/root-ca.crt \
    -extensions root_ca_ext


# Creating Signing CA

# Creating directories, index and serial files

mkdir -p ca/signing-ca/private ca/signing-ca/db crl certs
chmod 700 ca/signing-ca/private

cp /dev/null ca/signing-ca/db/signing-ca.db
cp /dev/null ca/signing-ca/db/signing-ca.db.attr
echo 01 > ca/signing-ca/db/signing-ca.crt.srl
echo 01 > ca/signing-ca/db/signing-ca.crl.srl

# Creating CA request

openssl req -new \
    -config etc/signing-ca.conf \
    -out ca/signing-ca.csr \
    -keyout ca/signing-ca/private/signing-ca.key

# Creating CA certificate

openssl ca \
    -config etc/root-ca.conf \
    -in ca/signing-ca.csr \
    -out ca/signing-ca.crt \
    -extensions signing_ca_ext

# Create TLS server request

SAN=DNS:www.simple.org \
openssl req -new \
    -config etc/server.conf \
    -out certs/simple.org.csr \
    -keyout certs/simple.org.key

# Create TLS server certificate 

openssl ca \
    -config etc/signing-ca.conf \
    -in certs/simple.org.csr \
    -out certs/simple.org.crt \
    -extensions server_ext

# Revoking a certificate

openssl ca \
    -config etc/signing-ca.conf \
    -revoke ca/signing-ca/01.pem \
    -crl_reason superseded

# Creating CRL

openssl ca -gencrl \
    -config etc/signing-ca.conf \
    -out crl/signing-ca.crl

# Create DER certificate

openssl x509 \
    -in certs/fred.crt \
    -out certs/fred.cer \
    -outform der

# Create DER CRL

openssl crl \
    -in crl/signing-ca.crl \
    -out crl/signing-ca.crl \
    -outform der

# Create PKCS#7 bundle 

openssl crl2pkcs7 -nocrl \
    -certfile ca/signing-ca.crt \
    -certfile ca/root-ca.crt \
    -out ca/signing-ca-chain.p7c \
    -outform der

# Create PKCS#12 bundle

openssl pkcs12 -export \
    -name "Fred Flintstone" \
    -inkey certs/fred.key \
    -in certs/fred.crt \
    -out certs/fred.p12

# Create PEM bundle

cat ca/signing-ca.crt ca/root-ca.crt > \
    ca/signing-ca-chain.pem

cat certs/fred.key certs/fred.crt > \
    certs/fred.pem

# Viewing commands

## Request 

openssl req \
    -in certs/fred.csr \
    -noout \
    -text

## Certificate

openssl x509 \
    -in certs/fred.crt \
    -noout \
    -text

## CRL

openssl crl \
    -in crl/signing-ca.crl \
    -inform der \
    -noout \
    -text

## PKCS#7 bundle

openssl pkcs7 \
    -in ca/signing-ca-chain.p7c \
    -inform der \
    -noout \
    -text \
    -print_certs

# PKCS#12 bundle

openssl pkcs12 \
    -in certs/fred.p12 \
    -nodes \
    -info

