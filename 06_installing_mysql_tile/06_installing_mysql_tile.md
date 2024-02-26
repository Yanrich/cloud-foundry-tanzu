# Installing mysql tile

In this lab you will be:

- installing mysql tile

## Steps to install mysql tile

In network.pivotal.io, search for “VMware SQL with MySQL for Tanzu Application Service”

In that page, you can get the pivnet command to download the pivotal file for mysql tile and corresponding stemcells

Download using below commands:

```bash
  pivnet download-product-files --product-slug='pivotal-mysql' --release-version='3.1.3' --product-file-id=1709772
  
  pivnet download-product-files 
  --product-slug='stemcells-ubuntu-jammy' --release-version='1.379' --product-file-id=1737616
```

Follow the documentation page and do the next steps to complete installation