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

Go to [Broadcom support portal](https://profile.broadcom.com/web/registration) and create an account with the fictitious email address used in LAB 1

Once registered, select the Tanzu category (button on the left of your profile at the top right)

In My Dashboard, click on Tanzu API Token in the Quick Links square on the right below Critical Alerts

Click on Request New Refresh Token

Copy this "Refresh Token" to put it in the .env file of LAB 1

```bash
PIVNET_TOKEN=xxxxxxxxx-r
```

Then, type the following commands

```bash
source ~/.bashrc
echo $PIVNET_TOKEN
echo PIVNET_TOKEN=$PIVNET_TOKEN >> ~/.envv
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

[Broadcom support portal](https://support.broadcom.com/) is essentially a catalog of software artifacts published by Broadcom, including the Tanzu Application Service (TAS), Operations Manager, TAS service tiles, Health watch, Harbor, Concourse for VMware Tanzu and partner products that are part of the Tanzu ecosystem.

In the search field, enter "Tanzu Platform for Cloud Foundry"

For a given release (for instance 4.0.30+LTS-T), a product will consist of one or more downloadable software artifacts

There are 3 artifacts named “Small Footprint TAS”, ”CF CLI” and “Tanzu Application Service”

We want to download the "Small Footprint TAS" section

```bash
pivnet product-files -p elastic-runtime -r 4.0.30+LTS-T
pivnet accept-eula -p elastic-runtime -r 4.0.30+LTS-T
pivnet download-product-files --product-slug='elastic-runtime' --release-version='4.0.30+LTS-T' --product-file-id=199896
```

In the "CF CLI" section, note the version supported and install cf CLI (see [Installers and compressed binaries](https://github.com/cloudfoundry/cli/wiki/V8-CLI-Installation-Guide#installers-and-compressed-binaries))

```bash
curl -L "https://packages.cloudfoundry.org/stable?release=linux64-binary&version=8.8.2&source=github" | tar -zx
sudo mv cf8 /usr/local/bin
sudo mv cf /usr/local/bin
# ...copy tab completion file on Ubuntu (takes affect after re-opening your shell)
sudo curl -o /usr/share/bash-completion/completions/cf8 https://raw.githubusercontent.com/cloudfoundry/cli-ci/master/ci/installers/completion/cf8
# ...and to confirm your cf CLI version
cf version
```

By clicking on "Upgrade/Dependency Information", you will see the recommended version for Stemcells (Ubuntu Jammy) and VMware Tanzu Operations Manager. You have to download the products for Google Cloud Platform.

```bash
pivnet product-files -p stemcells-ubuntu-jammy -r 1.651
pivnet accept-eula -p stemcells-ubuntu-jammy -r 1.651
pivnet download-product-files --product-slug='stemcells-ubuntu-jammy' --release-version='1.651' --product-file-id=200031
```

For Tanzu Ops Manager, note the version and the build in the yml file

```bash
pivnet product-files -p ops-manager -r 3.0.35+LTS-T
pivnet accept-eula -p ops-manager -r 3.0.35+LTS-T
pivnet download-product-files -p ops-manager -r 3.0.35+LTS-T -i 200219 
```

Example: Tanzu Ops Manager YAML for GCP - 3.0.35+LTS-T (3.0.35-build.1608)