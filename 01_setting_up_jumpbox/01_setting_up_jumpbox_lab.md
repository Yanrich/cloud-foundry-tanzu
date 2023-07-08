# Setting up Jumpbox

In this lab, you will be setting a jumpbox on GCP which we will be using for doing all the admin tasks.

It is a good practice to do everything from a jumpbox instead of your local
machine. You can avoid downloading and uploading all the files required for
TAS installation to your workstation and use Google's network instead.

## Prepare the environment on your linux workstation

- Install [google-cloud-cli](https://www.golinuxcloud.com/install-gcloud-on-linux/)
- Install [direnv package](https://direnv.net/). See [direnv setup](https://direnv.net/docs/hook.html)

## Commands on your linux workstation

- create a working directory and a .envrc file

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



## Commands on your jumpbox

- Go to [Bosh CLI](https://bosh.io/docs/cli-v2-install/) and navigate to the [Bosh CLI Github release page](https://github.com/cloudfoundry/bosh-cli/releases)

```bash
wget https://github.com/cloudfoundry/bosh-cli/releases/download/v7.2.4/bosh-cli-7.2.4-linux-amd64 &&
chmod +x bosh-cli-7.2.4-linux-amd64 &&
sudo mv bosh-cli-7.2.4-linux-amd64 /usr/local/bin/bosh
```