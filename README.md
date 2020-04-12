# Certificates
Public Certificates, configuration, and direction

## Root CA Certificate

Based on https://help.f-secure.com/product.html?business/threatshield/latest/en/task_50407934989D4923AB76367EA0E627CA-threatshield-latest-en

Generate Root Certificate & Key
```
openssl req -x509 -sha256 -days 3650 -newkey rsa:3072 \
    -config root-csr.conf -keyout private/rootCA_key.crt \
    -out rootCA.crt
```

Verify Certificate
```
openssl x509 -in rootCA.crt -text -noout
```

## Intermediate CA Certificate

Based on https://help.f-secure.com/product.html#business/threatshield/latest/en/task_9EF132D90B9241268DF4EC8CA5FADBBF-threatshield-latest-en

Generate Intermediate CSR & Key
```
openssl req -new -config CA-csr.conf -out CA.csr \
        -keyout private/CA_key.crt
```

Sign Intermediate CSR & Generate Intermediate Certificate
```
openssl ca -config rootCA.conf -days 365 -create_serial \
    -in CA.csr -out CA.crt -extensions ca_ext -notext
```

Certificate Chain into Linked File
```
cat CA.crt rootCA.crt >CA.pem
```

Verify Certificate
```
openssl x509 -in CA.crt -text -noout
```

## Self Signed Certificate

Based on https://help.f-secure.com/product.html#business/threatshield/latest/en/task_D81B8959CD3643C5A9E8DD0E2A4EF32E-threatshield-latest-en

Generate CSR & Key
```
openssl req -new -config san-csr.conf -out san.csr \
        -keyout private/san_key.crt
```

Sign CSR & Generate Certificate
```
openssl ca -config CA-san.conf -days 365 -create_serial \
    -in san.csr -out san.crt -extensions leaf_ext -notext
```

### SAN Domains Explained
The SAN DNS entries are based on the [HiJack Tracking](../hijack-tracking) repository.

Certificate Chain into Linked File
```
cat san.crt CA.pem >san.pem
```

Verify Certificate
```
openssl x509 -in san.crt -text -noout
```

# Troubleshooting

## Configuration file issue

`4403822188:error:0DFFF097:asn1 encoding routines:CRYPTO_internal:string too long:/AppleInternal/BuildRoot/Library/Caches/com.apple.xbs/Sources/libressl/libressl-47.100.4/libressl-2.8/crypto/asn1/a_mbstr.c:156:maxsize=2`

This was the result of the initial `req` for the Root CA Certificate because the configuration file states `prompt=0` while still having `_default = [default]` in the configuration.
