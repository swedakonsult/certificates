# Certificate Administration

Source: https://pki-tutorial.readthedocs.io/en/latest/advanced/

## Before First Run

Create File Structure
```
mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private
```

Initialize Required Files
```
cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl
```

## Root CA Certificate Generation

Generate the CSR & Key
```
openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key.crt
```

Generate the CSR With Existing Key
```
openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -key ca/root-ca/private/root-ca.key.crt
```

Sign the CA Certificate
```
openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca.csr \
    -out ca/root-ca.crt \
    -extensions root_ca_ext \
    -enddate 20301231235958Z
```

Create a Revocation List
```
openssl ca -gencrl \
    -config etc/root-ca.conf \
    -out crl/root-ca.crl
```

Validate the Revocation List
```
openssl crl -in crl/root-ca.crl -noout -text
```

## TLS Intermediate CA Certificate Generation

```
mkdir -p ca/tls-ca/private ca/tls-ca/db crl certs
chmod 700 ca/tls-ca/private
```

```
cp /dev/null ca/tls-ca/db/tls-ca.db
cp /dev/null ca/tls-ca/db/tls-ca.db.attr
echo 01 > ca/tls-ca/db/tls-ca.crt.srl
echo 01 > ca/tls-ca/db/tls-ca.crl.srl
```

Generate the CSR & Key
```
openssl req -new \
    -config etc/tls-ca.conf \
    -out ca/tls-ca.csr \
    -keyout ca/tls-ca/private/tls-ca.key.crt
```

Generate the CSR With Existing Key
```
openssl req -new \
    -config etc/tls-ca.conf \
    -out ca/tls-ca.csr \
    -key ca/tls-ca/private/tls-ca.key.crt
```

```
openssl ca \
    -config etc/root-ca.conf \
    -in ca/tls-ca.csr \
    -out ca/tls-ca.crt \
    -extensions signing_ca_ext
```

```
openssl ca -gencrl \
    -config etc/tls-ca.conf \
    -out crl/tls-ca.crl
```


