# Create service account and use Terraform

In this lab you will be:

- Use terraform to create all required resources on GCP

## Create service account

- Edit the ~/workspace/.envrc file

```bash
...
export PROJECT_ID=$(gcloud config get-value core/project)
```

- In the console

```bash
ACCOUNT_NAME=terraform-sa
gcloud iam service-accounts create ${ACCOUNT_NAME}

gcloud iam service-accounts keys create "terraform.key.json" --iam-account "${ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"

gcloud projects add-iam-policy-binding ${PROJECT_ID} --member "serviceAccount:${ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role 'roles/owner'
```

## Use Terraform

- Install Terraform

```bash
wget -O terraform.zip https://releases.hashicorp.com/terraform/0.11.15/terraform_0.11.15_linux_amd64.zip && \
unzip terraform.zip && \
sudo mv terraform /usr/local/bin 
```

- Download the Terraform template in TAS 2.9.5 after accepting EULA (GCP Terraform Templates 0.98.0)

```bash
pivnet download-product-files --product-slug='elastic-runtime' --release-version='2.9.5' --product-file-id=697856
```

:bulb: For more information, please see [Deploying Ops Manager on GCP Using Terraform](https://docs.pivotal.io/ops-manager/2-9/gcp/prepare-env-terraform.html) and [Install Ops Manager using paving in GitHub](https://github.com/pivotal/paving)

- Unzip the Terraform template

```bash
unzip terraforming-gcp-0.98.0.zip
cd pivotal-cf-terraforming-gcp-f4aab02/terraforming-pas
```

- Create a terraform.tfvars file with by modifying the Opsman URL to match the version noted above (2.10.61-build.1615)

```bash
cat > ./terraform.tfvars <<-EOF
env_name = "$ENV_NAME"
project = "$PROJECT_ID"
region = "us-east1"
zones = ["us-east1-b", "us-east1-c", "us-east1-d"]
dns_suffix = "$DOMAIN_NAME"
opsman_image_url = "https://storage.googleapis.com/ops-manager-us/pcf-gcp-3.0.18-build.1029.tar.gz"
create_gcs_buckets = "false"
external_database = 0
ssl_cert = <<SSL_CERT
REPLACE_ME_WITH_YOUR_SSL_CERT
SSL_CERT
ssl_private_key = <<SSL_KEY
REPLACE_ME_WITH_YOUR_PRIVATE_KEY
SSL_KEY
service_account_key = <<SERVICE_ACCOUNT_KEY
$(cat ~/terraform.key.json)
SERVICE_ACCOUNT_KEY
EOF
```

- Replace SSL_CERT and SSL_KEY with the values obtained above

- Use the terraform commands

```bash
terraform init
terraform plan
terraform apply
```
