# Veolia UKI IoT PKI Configuration

## OpenSSL Config Files

| Type | Config | File name |
| --- | --- | --- |
| Root | Root CA Config File | etc/root-ca.conf |
| Issuer | `<issuer-name>` CA Config File | etc/`<issuer-name>`-ca.conf |
| Certificate | `<issuer-name>` Certificate Config File | etc/`<issuer-name>`.conf |

**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

## Initial Setup

If needed install Git on the EC2 instance to be used to host the PKI.
```
yum install git
```

Clone repo from GitLab to PKI Server
```
git clone https://github.com/VeoliaUKI/veolia-uki-iot-pki
cd veolia-uki-iot-pki
```

## Root CA Configuration Steps

Perform the following steps to create the Veolia UKI IoT Root CA.

### Create Root CA Directories
Execute the following commands to create the Root CA directories.
```
mkdir -p ca/root-ca/private ca/root-ca/db crl/root-ca certs/root-ca
chmod 700 ca/root-ca/private
```

The `ca` directory holds the CA resources, the `crl` directory holds the CRLs (not currently used) and the `certs` directory holds the device certificates.

### Create Root CA Certificate Database
Execute the following commands to create the Root CA database.
```
cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl
```

### Create Root CA Certificate Request
Execute the following command to create the Root CA certificate request.
```
openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key
```
When prompted, enter a PEM pass phrase, and when prompted to verify the pass phrase, enter it again.

With the `openssl req -new` command we create a private key and a Certificate Signing Request (CSR) for the root CA. The configuration is taken from the `[req]` section of the root CA configuration file.

### Create Root CA Certificate
Execute the following command to create the Root CA certificate.
```
openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca/root-ca.csr \
    -out ca/root-ca/root-ca.crt
```
When prompted, enter a PEM pass phrase, when asked to sign the certificate, press `y` and when asked to commit the certificate, press `y` to commit the entry to the database.

### Create Initial Root CA CRL
Execute the following command to create the Root CA certificate revocation list (CRL).
```
openssl ca -gencrl \
    -config etc/root-ca.conf \
    -out crl/root-ca/root-ca.crl
```
When prompted, enter the Root CA pass phrase and press `Enter`

## Issuer CA Configuration Steps

Perform the following steps each time there is a need to generate a new Issuer CA.

### Create Issuer CA Directories
Execute the following commands to create the Issuer CA directories.
```
mkdir -p ca/<issuer-name>-ca/private ca/<issuer-name>-ca/db crl/<issuer-name>-ca certs/<issuer-name>-ca
chmod 700 ca/<issuer-name>-ca/private
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

The `ca` directory holds the CA resources, the `crl` directory holds the CRLs (not curretly used) and the `certs` directory holds the device certificates.

### Create Issuer CA Database
Execute the following commands to create the Issuer CA database.
```
cp /dev/null ca/<issuer-name>-ca/db/<issuer-name>-ca.db
cp /dev/null ca/<issuer-name>-ca/db/<issuer-name>-ca.db.attr
echo 01 > ca/<issuer-name>-ca/db/<issuer-name>-ca.crt.srl
echo 01 > ca/<issuer-name>-ca/db/<issuer-name>-ca.crl.srl
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

### Create Issuer CA Configuration Files
Execute the following commands to create the Issuer CA configuration files.
```
cp etc/template.conf etc/<issuer-name>.conf
cp etc/template-ca.conf etc/<issuer-name>-ca.conf
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

### Edit the Issuer Configuration Files

Edit the config files replacing `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

This can be done by editing the files directly using a text editor such as `nano` or by running the commands below.
```
sed -i 's/<issuer-name>/new_issuer/g' etc/new_issuer.conf
sed -i 's/<issuer-name>/new_issuer/g' etc/new_issuer-ca.conf
```
**_Note:_** Replace the `new-issuer` text in the commands above with the name of the new issuer used in the preceeding steps.

### Create Issuer CA Certificate Request
Execute the following command to create the Issuer CA certificate request.
```
openssl req -new \
    -config etc/<issuer-name>-ca.conf \
    -out ca<issuer-name>-ca/<issuer-name>-ca.csr \
    -keyout ca/<issuer-name>-ca/private/<issuer-name>-ca.key
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

When prompted, enter a PEM pass phrase, and when prompted to verify the pass phrase, enter it again.

We create a private key and a CSR for the Issuer CA. The configuration is taken from the `[req]` section of the Issuer CA configuration file.

### Create Issuer CA Certificate
Execute the following commands to create the Issuer CA certificate.
```
openssl ca \
    -config etc/root-ca.conf \
    -in ca/<issuer-name>-ca/<issuer-name>-ca.csr \
    -out ca/<issuer-name>-ca/<issuer-name>-ca.crt \
    -extensions signing_ca_ext
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

This uses the root CA to issue the issuer CA certificate.

When prompted, enter a PEM pass phrase, when asked to sign the certificate, press y and when asked to commit the certificate, press y to commit the entry to the database.

### Create Initial Issuer CA CRL
Execute the following command to create the Issuer CA certificate revocation list (CRL).
```
openssl ca -gencrl \
    -config etc/<issuer-name>-ca.conf \
    -out crl/<issuer-name>-ca/<issuer-name>-ca.crl
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

### Create Issuer CA PEM Bundle
Execute the following command to create the Issue CA PEM bundle.
```
cat ca/<issuer-name>-ca/<issuer-name>-ca.crt ca/root-ca/root-ca.crt > \
    ca/<issuer-name>-ca/<issuer-name>-ca-chain.pem
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.

## Issuer CA Azure IoT Hub Proof of Ownership Verification
Perform the following steps to generate a new Proof of Ownership certificate for verifying the Issue CA within the Azure IoT Hub

### Create Proof of Ownerhship Certificate Request
Execute the following command to create a new certficate request for Azure Iot Hub proof of ownership certificate siging request (CSR).
```
openssl req -new \
    -config etc/<issuer-name>.conf \
    -out certs/<issuer-name>-ca/proof-of-ownership.csr \
    -keyout certs/<issuer-name>-ca/proof-of-ownership.key
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided, and `<verification-key>` with the proof of ownership verification key generated by the Azure IoT Hub.

You will be prompted to enter the Distinquished Name (DN) values for the certificate, where appropriate accept the default, but please ensure that the `Common Name` value is set to the proof of ownership verification key generated by the Azure IoT Hub.

### Create Proof of Ownerhship Certificate
Execute the following command to create a new Azure IoT Hub proof of ownership certificate.
```
openssl ca \
    -config etc/<issuer-name>-ca.conf \
    -in certs/<issuer-name>-ca/proof-of-ownership.csr \
    -out certs/<issuer-name>-ca/proof-of-ownership.crt
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided.


## Issuer CA Device Certificate Generation

Perform the following steps to generate a new IoT device certificate from the Issuer CA.

### Create Device Certificate Request
Execute the following command to create a new Azure IoT device certificate signing request (CSR).
```
openssl req -new \
    -config etc/<issuer-name>.conf \
    -out certs/<issuer-name>-ca/<device-id>.csr \
    -keyout certs/<issuer-name>-ca/<device-id>.key
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided, and `<device-id>` with the ID of the device created in the Azure IoT Hub.

You will be prompted to enter the Distinquished Name (DN) values for the certificate, where apprpriate accept the default, but please ensure that the `Common Name` value is set to the Device ID generated in the Azure IoT Hub.

### Create Device Certificate
Execute the following command to create a new Azure IoT device certificate.
```
openssl ca \
    -config etc/<issuer-name>-ca.conf \
    -in certs/<issuer-name>-ca/<device-id>.csr \
    -out certs/<issuer-name>-ca/<device-id>.crt
```
**_Note:_** Replace `<issuer-name>` with the name of the issuer to whom the certificates will be provided, and `<device-id>` with the ID of the device created in the Azure IoT Hub.
