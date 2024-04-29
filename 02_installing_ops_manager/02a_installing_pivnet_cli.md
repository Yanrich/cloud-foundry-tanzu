# Installing Ops Manager VM

In this section, you will be:

- Install pivnet and understand how to use it
- Create a certificate
- Use terraform to create all required resources on GCP
- Install ops manager

If this is not already the case, connect to the LAB 1 jumpbox

```bash
gcloud compute ssh ubuntu@jumpbox --zone=$CLOUDSDK_COMPUTE_ZONE
```

## Network.pivotal.io

Go to [network.pivotal.io](https://network.pivotal.io/) and create an account with the fictitious email address used in LAB 1

Once registered, go to your profile (top right of the page)

On the UAA API Token line, click on “Request New Refresh Token”

Copy this UAA API Token to put it in the .env file of LAB 1

```bash
PIVNET_TOKEN=xxxxxxxxx-r
```

Then, type the following commands

```bash
source ~/.bashrc
echo $PIVNET_TOKEN
```

## Installing pivnet-cli

Go to [pivnet-cli](https://github.com/pivotal-cf/pivnet-cli)

```bash
wget https://github.com/pivotal-cf/pivnet-cli/releases/download/v4.1.1/pivnet-linux-amd64-4.1.1
chmod +x pivnet-linux-amd64-4.1.1
sudo mv pivnet-linux-amd64-4.1.1 /usr/local/bin/pivnet
pivnet login --api-token=${PIVNET_TOKEN}
```

You can try these commands to get familiar with this utility

```bash
pivnet products --help
# list products:
pivnet products
# show product information:
pivnet product -p <product-name>
# list releases for a product:
pivnet releases -p <product-name>
# list files for a specific product release:
pivnet product-files -p <product-name> -r <product-version>
# accept the EULA for each product
pivnet accept-eula -p <product-name> -r <product-version>
# download specific product files
pivnet download-product-files -p <product-name> -r <product-version> -i <product-id>
```

:bulb: If you try to download a product, you must first accept the EULA from the web browser

[Network.pivotal.io](https://network.pivotal.io/) is essentially a catalog of software artifacts published by VMware, including the Tanzu Application Service (TAS), Operations Manager, TAS service tiles, Health watch, Harbor, Concourse for VMware Tanzu and partner products that are part of the Tanzu ecosystem.

In the search field, enter "VMware Tanzu Application Service for VMs"

On the product page, note "Releases" drop-down

For a given release (for instance 4.0.21+LTS-T), a product will consist of one or more downloadable software artifacts

There are 3 artifacts named “Small Footprint PAS”, ”CF CLI” and “Pivotal Application Service”

In the details (right column), we have the supported versions VMware Tanzu Operations Manager and the Pivotal Stemcells

In the "Small Footprint TAS" section, click on the "i" and copy the Pivnet CLI line

```bash
pivnet download-product-files --product-slug='elastic-runtime' --release-version='4.0.21+LTS-T' --product-file-id=1785154
```

In the "CF CLI" section, note the version supported and install cf CLI (see [Installers and compressed binaries](https://github.com/cloudfoundry/cli/wiki/V8-CLI-Installation-Guide#installers-and-compressed-binaries))

```bash
curl -L "https://packages.cloudfoundry.org/stable?release=linux64-binary&version=8.7.9&source=github" | tar -zx
sudo mv cf8 /usr/local/bin
sudo mv cf /usr/local/bin
# ...copy tab completion file on Ubuntu (takes affect after re-opening your shell)
sudo curl -o /usr/share/bash-completion/completions/cf8 https://raw.githubusercontent.com/cloudfoundry/cli-ci/master/ci/installers/completion/cf8
# ...and to confirm your cf CLI version
cf version
```

By clicking on Pivotal Stemcells (Ubuntu Xenial), do the same operation for Ubuntu Xenial Stemcell for Google Cloud Platform

```bash
pivnet download-product-files --product-slug='stemcells-ubuntu-xenial' --release-version='621.924' --product-file-id=1777045
pivnet download-product-files --product-slug='stemcells-ubuntu-jammy' --release-version='1.423' --product-file-id=1778542
```

For Tanzu Ops Manager, note the version and the build

```bash
Tanzu Ops Manager YAML for GCP - 3.0.25+LTS-T (3.0.25-build.1261)
```
