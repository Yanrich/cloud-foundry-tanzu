# Installing bosh director

In this lab you will be:

- Installing bosh director through ops manager

Once logged in to Ops Manager, you will see an orange-colored tile labelled
Bosh Director for GCP.

This tile represents the BOSH director that Ops Manager will use to perform all further work.

The orange color indicates that the tile is not yet configured.
Click on that tile

## Steps to install bosh director

### Google config

This section is where you tell the BOSH director how to communicate with the
underlying IaaS (in this case GCP).

Terraform has set up another service account for Ops Manager to use, so this
will tell Ops Manager to use that.

Observe what is the name of this serviceaccount in GCP console by going to IAM&Admin > Service Accounts.

Select Google Config. Complete the following fields:

- Project ID: Enter the value of project from your terraform.tfvars file
- Default Deployment Tag: Enter the value of env_name from your terraform.tfvars file
- Select the radio button for “The ops manager VM Service account”
- Click Save

### Director Config

- In the NTP Servers (comma delimited) field, enter 169.254.169.254
- Select the Enable VM Resurrector Plugin checkbox to enable the BOSH Resurrector functionality and increase your runtime availability.
- Enable Post Deploy Scripts.
- Leave all other settings unchanged.
- Click Save

### Create availability zones

- Configure the BOSH director to allocate jobs to multiple Availability Zones
(AZs). By balancing jobs across multiple AZs, BOSH increases the resiliency of services.
- Use the same list of availability zones already specified in your
terraform.tfvars file
- In our case, azs should be zones = ["us-east1-b", "us-east1-c", "us-east1-d"]
- Click on Add 3 times and give the above values

```bash
Google Availability Zone: us-east1-b
Google Availability Zone: us-east1-c
Google Availability Zone: us-east1-d
```

### Create networks

- This section is where you configure the networks that are available to the BOSH director. Because the BOSH director fulfills the role of a network administrator (i.e. it assigns VMs to subnets and gives them IP addresses) it needs to know what networks and subnets are available at the IaaS level. Previously, when you ran ***terraform apply***, several GCP subnets were created for you.
- You can see those by running **gcloud compute networks subnets list**

- You will see three subnets named with the following convention:

```bash
- ENV_NAME-infrastructure-subnet
- ENV_NAME-pas-subnet
- ENV_NAME-services-subnet
```

- You will also see that each subnet has a slice of IP address space

- Ops Manager "networks" map directly to subnets in GCP
- In the Ops Manager, leave the Enable ICMP checks global setting unchecked then, for each of the names above, perform the following steps:
- Use the Add Network button to create an Ops Manager network. Click on Add Network button 3 tomes to create 3 networks. Give the names of networks as
  - infrastructure
  - pas
  - services

The Google Network Name is critical in establishing the mapping from the
BOSH director down to the IaaS. The format looks like "NETWORK/SUBNET/REGION"
Following can be the values for 3 networks:

```bash
vcapenv-pcf-network/vcapenv-infrastructure-subnet/us-east1
vcapenv-pcf-network/vcapenv-pas-subnet/us-east1
vcapenv-pcf-network/vcapenv-services-subnet/us-east1
```

- Set the CIDR field to match the corresponding GCP subnet's CIDR range (infrastructure network only). Reserve the first ten IP addresses. This address range (expressed as 10.0.0.1-10.0.0.10) needs to be set aside to provide a contiguous space for non-BOSH VMs on the infrastructure network (e.g. jumpboxes and backup VMs).
- Set the DNS setting for all three Ops Manager's networks to 169.254.169.254
- Set the Gateway field to the first IP address in the corresponding CIDR range. For example, for the CIDR range 10.0.0.0/24, the first address in the range is 10.0.0.1.
- For each of the Ops Manager networks, select all of the AZ's you set up previously.
- Leave all other settings unchanged and click Save

```bash
    infrastructure
        vcapenv-pcf-network/vcapenv-infrastructure-subnet/us-east1
        10.0.0.0/26
        10.0.0.1-10.0.0.10
        169.254.169.254
        10.0.0.1
        check the 3 availability zones
    pas        
        vcapenv-pcf-network/vcapenv-pas-subnet/us-east1
        10.0.4.0/24
        169.254.169.254
        10.0.4.1
        check the 3 availability zones
    services        
        vcapenv-pcf-network/vcapenv-services-subnet/us-east1
        10.0.8.0/24
        169.254.169.254
        10.0.8.1
        check the 3 availability zones   
```

### Assign AZs and networks

This section is where you tell Ops Manager where to put singleton jobs - i.e. jobs that are only comprised of a single VM. One job in particular, the BOSH director, is a single VM and needs an AZ to go to. Normally, you would want to deploy the BOSH director to a different AZ than the Ops Manager VM so choose any one of the other zones. ops-manager is deployed on us-east1-c so

```bash
Singleton Availability Zone 
us-east1-d
```

### Network

Select the "infrastructure" network for your BOSH Director.

### Security

Copy the root CA public key in Trusted Certificates
Be sure to select the checkbox Include OpsManager Root CA in Trusted Certs

### Resource config

Increase the value of Main Compilation Job to eight. A higher value will increase parallelization and provide faster completion of bosh deployments that you will be performing in subsequent labs.

- Suggestions:

```bash
Bosh Director: e2-highmem-2 cpu:2 ram:16 disk: 64
Main Compilation Job: e2-highmem-4 cpu:4 ram:32 disk:128
```

### Apply changes

- At this point, the BOSH director will be fully configured and ready to deploy.
- Go back to the installation dashboard and click Review Pending Changes followed by Apply Changes.
- This will take about 10 minutes to complete.
- Once it does complete, run gcloud compute instances list and see that another VM exists.

Congratulations!! You have installed Bosh Director
