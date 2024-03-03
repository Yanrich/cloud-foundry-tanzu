# Installing mysql tile

In this lab you will be:

- installing mysql tile

## Steps to upload mysql tile

In network.pivotal.io, search for “VMware SQL with MySQL for Tanzu Application Service”

In that page, you can get the pivnet command to download the pivotal file for mysql tile and corresponding stemcells

Download using below commands:

```bash
pivnet download-product-files --product-slug='pivotal-mysql' --release-version='3.1.3' --product-file-id=1709772
  
pivnet download-product-files --product-slug='stemcells-ubuntu-jammy' --release-version='1.340' --product-file-id=1702440
```

Upload the products

```bash
om upload-product -p pivotal-mysql-3.1.3.pivotal

om upload-stemcell -s light-bosh-stemcell-1.340-google-kvm-ubuntu-jammy-go_agent.tgz
```

You can use **om products** to list the available products

```bash
+----------------------+---------------+-------------------+-------------------+
|         NAME         |   AVAILABLE   |      STAGED       |     DEPLOYED      |
+----------------------+---------------+-------------------+-------------------+
| cf                   | 4.0.17        | 4.0.17            | 4.0.17            |
|                      | 2.11.50       |                   |                   |
| p-bosh               |               | 3.0.19-build.1050 | 3.0.19-build.1050 |
| pivotal-mysql        | 3.1.3-build.1 |                   |                   |
| pivotal-telemetry-om | 2.1.3-build.1 |                   |                   |
+----------------------+---------------+-------------------+-------------------+
```

To stage the product, use the following command:

```bash
om stage-product --product-name pivotal-mysql --product-version 3.1.3-build.1
```

## Manually configure MySQL tile

In the Ops Manager interface, configure the tile using the [MySQL 3.1 configuration](https://docs.vmware.com/en/VMware-SQL-with-MySQL-for-Tanzu-Application-Service/3.1/mysql-for-tas/install-config.html):

### Assign AZs and networks

```bash
  Place singleton jobs in AZ
    us-east1-b
  Balance other jobs in AZs
    Select three zones
    us-east1-b
    us-east1-c
    us-east1-d
  Network
    pas
  Service Network
    services
```

### Plan 1

```bash
  Plan
    Single Node
  MySQL default Version
    MySQL 5.7
  Service Plan Access
    Enable
  Plan Name
    db-small   
  Paid Plan
    leave unselected
  MySQL Availability Zone(s)
    Select three zones
    us-east1-b
    us-east1-c
    us-east1-d    
```

### Plan 2 and 3

```bash
  Plan
    Inactive
```

### Settings

```bash
  E-mail address
    admin@example.com
```

### Backups

There is no easy way to turn off :-)

```bash
  Backup configuration
    SCP
      Username
        nobody
      Private Key
        use the private key in the jumpbox (cat ~/.ssh/ops_manager)
      Hostname
        localhost
      Destination Directory
        /tmp/backups
      SCP Port  
        22
      Cron Schedule
        0 0 1 1 * 
```

### Errands

Select Off for the following errands:

- Validate no IP-bases bindings in use before upgrade-all-service-instances
- Upgrade all On-demand MySQL Service Instances
- Delete All Service Instances and Deregister Broker

## Apply changes

 Click on "Apply changes"

## Post MySQL Install

You can verify that the MySQL tile is present on the command line or using the apps manager console > Marketplace

```bash
  cf marketplace
```

You can create an instance

```bash
  cf create-service --help
  cf create-service p.mysql db-small my-db
```

Bosh takes care of the deployment

```bash
  bosh deployments
```

Congratulations!! You have installed Mysql tile
