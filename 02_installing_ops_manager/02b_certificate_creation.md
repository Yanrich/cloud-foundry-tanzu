# Certificate creation

In this lab you will be:

- Create a certificate signed by a Certificate Authority

:bulb: It is recommended to do the certificate operations on your linux workstation

- Edit the ~/.env file and add the domain name you have

```bash
export ENV_NAME=vcapenv
export DOMAIN_NAME=example.com
source ~/.bashrc
```

- The output of this lab will be obtaining 2 files

```bash
vcapenv.example.com.crt
vcapenv.example.com.key
```

## Generate CA

- A CA-signed certificate is required for further operations.
- If the certificate is not properly configured, Tanzu Application Service cannot be deployed.
- An error like *"Error: 'mysql_monitor/xyz' is not running after update. Review logs for failed jobs: replication-canary"* will be raised during deployment. See [Tanzu Community support](https://community.pivotal.io/s/article/5004y00001qj2be1645018715441?language=en_US).
- create the root key

```bash
openssl genrsa -aes256 -out rootCA.key 4096
```

## Generate CSR & create the root certificate

```bash
cat > ./openssl-rootca.cnf <<-EOF
[ ca ]
default_ca = CA_default
[ CA_default ]
x509_extensions = v3_ca
copy_extensions = copy
default_days = 365
default_crl_days = 30
default_md = sha256
preserve = no
[ req ]
prompt = no
default_bits = 4096
default_keyfile = cakey.pem
distinguished_name = req_distinguished_name
x509_extensions = v3_ca
string_mask = utf8only
[ req_distinguished_name ]
C = CA
ST = QC
L = Montreal
O = Education Ltd
OU = Education Ltd Certificate Authority
CN = Education Ltd Root CA
[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical,CA:true
keyUsage = cRLSign,keyCertSign
EOF
```

- Use the following command

```bash
openssl req -config openssl-rootca.cnf \
    -key rootCA.key \
    -new -x509 -days 7300 -sha256 -extensions v3_ca \
    -out rootCA.crt
```

- Validate the root certificate

```bash
openssl x509 -in rootCA.crt -text -noout
openssl x509 -purpose -in rootCA.crt -inform PEM
```

## Generate Certificate Signing Request (CSR)

- Create the certificate key

```bash
openssl genrsa -out ${ENV_NAME}.${DOMAIN_NAME}.key 2048
```

- Create a Certificate Signing Request (CSR)

```bash
cat > ./${ENV_NAME}.${DOMAIN_NAME}.cnf <<-EOF
[ req ]
prompt = no
default_bits = 2048
default_keyfile = ${ENV_NAME}.${DOMAIN_NAME}.key
distinguished_name = server_distinguished_name
req_extensions = server_req_extensions
string_mask = utf8only
[ server_distinguished_name ]
C=CA
ST=QC
L=Montreal
O=Education Ltd
OU=Education Ltd Client Certificate
CN = ${ENV_NAME}.${DOMAIN_NAME}
[ server_req_extensions ]
subjectKeyIdentifier = hash
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
subjectAltName = @alternate_names
[ alternate_names ]
DNS.1 = ${ENV_NAME}.${DOMAIN_NAME}
DNS.2 = *.sys.${ENV_NAME}.${DOMAIN_NAME}
DNS.3 = *.apps.${ENV_NAME}.${DOMAIN_NAME}
DNS.4 = uaa.sys.${ENV_NAME}.${DOMAIN_NAME}
DNS.5 = *.uaa.sys.${ENV_NAME}.${DOMAIN_NAME}
DNS.6 = login.sys.${ENV_NAME}.${DOMAIN_NAME}
DNS.7 = *.login.sys.${ENV_NAME}.${DOMAIN_NAME}
EOF
```

- Use the following command

```bash
openssl req -config ${ENV_NAME}.${DOMAIN_NAME}.cnf \
    -key ${ENV_NAME}.${DOMAIN_NAME}.key \
    -new -sha256 \
    -out ${ENV_NAME}.${DOMAIN_NAME}.csr
```

- Validate the CSR

```bash
openssl req -text -noout -verify -in ${ENV_NAME}.${DOMAIN_NAME}.csr
```

## Generate Certificate

- Create signing config file

```bash
cat > ./openssl-rootca-signing.cnf <<-EOF
[ ca ]
default_ca = CA_default
[ CA_default ]
dir = .
certificate = \$dir/rootCA.crt
private_key = \$dir/rootCA.key
new_certs_dir = \$dir       # Location for new certs after signing
database = \$dir/index.txt  # Database index file
serial = \$dir/serial.txt   # The current serial number
unique_subject = no     
x509_extensions = v3_ca
copy_extensions = copy
default_days = 365
default_crl_days = 30
default_md = sha256
preserve = no
[ req ]
prompt = no
default_bits = 4096
default_keyfile = cakey.pem
distinguished_name = req_distinguished_name
x509_extensions = v3_ca
string_mask = utf8only
[ req_distinguished_name ]
C = CA
ST = QC
L = Montreal
O = Education Ltd
OU = Education Ltd Certificate Authority
CN = Education Ltd Root CA
[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical,CA:true
keyUsage = cRLSign,keyCertSign
[ signing_policy ]
countryName = optional
stateOrProvinceName = optional
localityName = optional
organizationName = optional
organizationalUnitName = optional
commonName = supplied
emailAddress = optional
[ signing_req ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
EOF
```

- Create index and serial files

```bash
touch index.txt
echo '01' > serial.txt
```

- Use the following command

```bash
openssl ca -config openssl-rootca-signing.cnf \
    -policy signing_policy \
    -extensions signing_req \
    -in ${ENV_NAME}.${DOMAIN_NAME}.csr \
    -out ${ENV_NAME}.${DOMAIN_NAME}.crt
```

- Validate the certificate

```bash
openssl x509 -in ${ENV_NAME}.${DOMAIN_NAME}.crt -text -noout
```

## References

- [How do you sign a Certificate Signing Request with your Certification Authority?](https://stackoverflow.com/questions/21297139/how-do-you-sign-a-certificate-signing-request-with-your-certification-authority)
- [OpenSSL Certificate Authority](https://jamielinux.com/docs/openssl-certificate-authority/)
- [Deploying Ops Manager on GCP Using Terraform](https://docs.pivotal.io/ops-manager/2-9/gcp/prepare-env-terraform.html)
