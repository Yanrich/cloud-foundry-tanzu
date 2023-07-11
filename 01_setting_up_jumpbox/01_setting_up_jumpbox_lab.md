# Setting up Jumpbox

In this lab, you will be setting a jumpbox on GCP which we will be using for doing all the admin tasks.

It is a good practice to do everything from a jumpbox instead of your local
machine. You can avoid downloading and uploading all the files required for
TAS installation to your workstation and use Google's network instead.

## Prepare the environment on your linux workstation

- Install [google-cloud-cli](https://www.golinuxcloud.com/install-gcloud-on-linux/)
- Install [direnv package](https://direnv.net/). See [direnv setup](https://direnv.net/docs/hook.html)

## Commands on your linux workstation

- Create a working directory and a .envrc file

```bash
cd ~/workspace
cat > ./.envrc <<-EOF
export CLOUDSDK_CONFIG=$PWD/gcloud-config
export CLOUDSDK_CORE_PROJECT=vcaptasproject
export CLOUDSDK_COMPUTE_REGION=us-east1
export CLOUDSDK_COMPUTE_ZONE=us-east1-b
export CLOUDSDK_IAM_EMAIL="director@${CLOUDSDK_CORE_PROJECT}.iam.gserviceaccount.com"
EOF
```

- Associate the project with the billing account

```bash
gcloud projects create $CLOUDSDK_CORE_PROJECT
gcloud alpha billing accounts list --filter="open:true"
```

- It returns account_id

```bash
ACCOUNT_ID
017E7E-C29DB6-xxxxxx
```

- You can type

```bash
gcloud alpha billing projects link $CLOUDSDK_CORE_PROJECT --billing-account=017E7E-C29DB6-xxxxxx
```

- Check if it points to the right project and that the account is active

```bash
gcloud config list
gcloud config configurations list
```

- Enable APIs

```bash
gcloud services enable compute.googleapis.com && \
gcloud services enable iam.googleapis.com && \
gcloud services enable cloudresourcemanager.googleapis.com && \
gcloud services enable dns.googleapis.com && \
gcloud services enable sqladmin.googleapis.com
```

- Installing TAS for VMs requires more than the project's default 8 IP addresses.
- In the Google Cloud console, go to IAM&Admin > Quotas
- In the filter, put:

```bash
 Compute Engine API and Limit name: In-use-addresses-per-project-region
```

- Select region **us-east1** and edit quotas to change 8 to 25
- An email will be sent by <cloudquota@google.com> to confirm the change

- Check the list of available images on [Compute Images](https://cloud.google.com/compute/docs/images)

```bash
 gcloud compute images list
```

- Create the jumpbox in the project

```bash
 gcloud compute instances create "jumpbox" \
 --image-family "ubuntu-2204-lts" \
 --image-project "ubuntu-os-cloud" \
 --boot-disk-size "200" \
 --zone $CLOUDSDK_COMPUTE_ZONE
```

- Connect to the VM with ssh by entering a passphrase

```bash
gcloud compute ssh ubuntu@jumpbox --zone=$CLOUDSDK_COMPUTE_ZONE 
```

## Commands on your jumpbox

- Once on the jumpbox, type the following command

```bash
gcloud config list
```

- The account is a default service account. To perform operations as your Google account and have the required permissions, you must do:

```bash
gcloud auth login
```

- Run the gcloud config list command again to validate that the account on the jumpbox is the same as that of your Linux workstation.

- In the home directory, create an empty .env file

```bash
cat > ~/.env <<EOF
EOF
```

- Run the following commands

```bash
source ~/.env
echo "source ~/.env" >> ~/.bashrc
sudo apt update && sudo apt install unzip jq git -y
```

Congratulations! You have setup a jumpbox on GCP.
