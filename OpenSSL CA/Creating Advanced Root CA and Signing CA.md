
# Root CA

## Create directories, index and serial files

mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private

cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl

## Create CA request

openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key

## Create CA certificate

openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key

## Create initial CRL 

openssl ca -gencrl \
    -config etc/root-ca.conf \
    -out crl/root-ca.crl

# TLS Signing CA

## Create directories, serial and index files

mkdir -p ca/tls-ca/private ca/tls-ca/db crl certs
chmod 700 ca/tls-ca/private

cp /dev/null ca/tls-ca/db/tls-ca.db
cp /dev/null ca/tls-ca/db/tls-ca.db.attr
echo 01 > ca/tls-ca/db/tls-ca.crt.srl
echo 01 > ca/tls-ca/db/tls-ca.crl.srl

## Create CA request

openssl req -new \
    -config etc/tls-ca.conf \
    -out ca/tls-ca.csr \
    -keyout ca/tls-ca/private/tls-ca.key

## Create CA certificate

openssl ca \
    -config etc/root-ca.conf \
    -in ca/tls-ca.csr \
    -out ca/tls-ca.crt \
    -extensions signing_ca_ext

## Create initial CRL

openssl ca -gencrl \
    -config etc/tls-ca.conf \
    -out crl/tls-ca.crl

## Create PEM bundle

cat ca/tls-ca.crt ca/root-ca.crt > \
    ca/tls-ca-chain.pem

# Operate TLS CA

## Create TLS server request

SAN=DNS:green.no,DNS:www.green.no \
openssl req -new \
    -config etc/server.conf \
    -out certs/green.no.csr \
    -keyout certs/green.no.key

## Create TLS server certificate

openssl ca \
    -config etc/tls-ca.conf \
    -in certs/green.no.csr \
    -out certs/green.no.crt \
    -extensions server_ext
## Create PKCS#12 bundle

openssl pkcs12 -export \
    -name "green.no (Network Component)" \
    -caname "Green TLS CA" \
    -caname "Green Root CA" \
    -inkey certs/green.no.key \
    -in certs/green.no.crt \
    -certfile ca/tls-ca-chain.pem \
    -out certs/green.no.p12

## Create TLS client request

openssl req -new \
    -config etc/client.conf \
    -out certs/barney.csr \
    -keyout certs/barney.key

## Create TLS client certificate

openssl ca \
    -config etc/tls-ca.conf \
    -in certs/barney.csr \
    -out certs/barney.crt \
    -policy extern_pol \
    -extensions client_ext

## Create PKCS#12 bundle

openssl pkcs12 -export \
    -name "Barney Rubble (Network Access)" \
    -caname "Green TLS CA" \
    -caname "Green Root CA" \
    -inkey certs/barney.key \
    -in certs/barney.crt \
    -certfile ca/tls-ca-chain.pem \
    -out certs/barney.p12

## Revoke certificate

openssl ca \
    -config etc/tls-ca.conf \
    -revoke ca/tls-ca/02.pem \
    -crl_reason affiliationChanged

## Create CRL

openssl ca -gencrl \
    -config etc/tls-ca.conf \
    -out crl/tls-ca.crl
