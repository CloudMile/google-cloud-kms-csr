# CSR with Google Cloud KMS

This project forks from https://github.com/mattes/google-cloud-kms-csr

Quick utility tool that creates a CSR cert and signs it with a private key coming from Google Cloud KMS or HSM.
The private key never leaves Google, everyone is happy. The CSR can then be used to get cert from CA.

This project use SignatureAlgorithm: `x509.SHA256WithRSA`

* https://cloud.google.com/kms/docs/algorithms
* https://cloud.google.com/kms/docs/reference/rest/v1/projects.locations.keyRings.cryptoKeys#CryptoKeyVersionTemplate

Recommend version: go 1.12+

## Prepare the HSM key

Create a HSM key on Cloud KMS: https://cloud.google.com/kms/docs/hsm

Set the variables

```
export KMS_LOCATION="asia-east1"
export KMS_KEY_RING="test-hsm-ring-${KMS_LOCATION}"
export KMS_KEY="test-hsm-key"
```

Create the key rings

```
gcloud kms keyrings create "${KMS_KEY_RING}" \
  --location "${KMS_LOCATION}"
```

Create the key

```
gcloud kms keys create "${KMS_KEY}" \
  --location "${KMS_LOCATION}" \
  --keyring "${KMS_KEY_RING}" \
  --purpose asymmetric-signing \
  --default-algorithm rsa-sign-pkcs1-2048-sha256 \
  --protection-level hsm
```

Get the key resource ID

```
export KMS_KEY_RESOURCE_ID=$(gcloud alpha kms keys versions list \
  --location "${KMS_LOCATION}" \
  --keyring "${KMS_KEY_RING}" \
  --key "${KMS_KEY}" \
  --sort-by ~name \
  --limit 1 \
  --format="value(name)")
```

```
echo $KMS_KEY_RESOURCE_ID
```

Key Resource ID has the following format:

```
projects/xxx/locations/xxx/keyRings/xxx/cryptoKeys/xxx/cryptoKeyVersions/xxx
```

## Usage

Build binary.

```
go build -o csr
```

Execute command to generate example csr:

```
./csr \
  --key $KMS_KEY_RESOURCE_ID \
  --out my.csr
```

Show all configuraion options:

```
./csr --help
```


Execute command to generate csr with full configuration(Example):

```
./csr \
  --key $KMS_KEY_RESOURCE_ID \
  --out my.csr \
  --common-name super.example.com \
  --org "Big Oraganization" \
  --org-unit "Super Unit" \
  --country TW \
  --province Taiwan \
  --locality Taipei
```

You can verify `my.csr` with:

```
openssl req -text -noout -verify -in my.csr
```

Or with this website: https://ssltools.digicert.com/checker/views/csrCheck.jsp

## Troubleshooting

Google's application credentials are used for authenticating with the Google API.
If you haven't done so already, you can set the application default credentials locally with:

```
gcloud auth application-default login
```

## Destroy Key

[Destroying and restoring key versions](https://cloud.google.com/kms/docs/destroy-restore)

## Document

  * https://cloud.google.com/kms/docs/how-tos
  * https://en.wikipedia.org/wiki/Certificate_signing_request
