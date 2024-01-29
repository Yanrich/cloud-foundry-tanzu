# Installing ops manager

In this lab you will be:

- Install ops manager

## Use Installing ops manager

The Google Cloud Console web UI can be used to gain a more-detailed understanding of the infrastructure components that were provisioned.

- The VPC Networks section will show a new network with three subnets. One for management VMs, a second for the platform components, and a third subnet for services VMs.
- Also under VPC Networks, you will find new external IP addresses,firewall rules, and routes.
- Under Network Services > Load balancing, you will see a new load balancer.
- Under Network Services > Cloud DNS, you will also find a new zone for
your environment.

In GCP console, under Network Services > Cloud DNS, if you observe the
Cloud DNS Zone, there will be a zone **vcapenv-zone**. In this zone, check the NS type and note the 4 domain names associated (for instance ns-cloud-a1.googledomains.com., ns-cloud-a2.googledomains.com., ns-cloud-a3.googledomains.com. et ns-cloud-a4.googledomains.com)

At the same level of **vcapenv-zone**, create a new DNS zone **examplezone**

```bash
Zone type: Public
Zone name: examplezone
DNS name: example.com
DNSSEC: Off
Cloud Logging: Off
```

GCP will create NS record set for this zone. You have to update these name
servers in Godaddy site for your domain. In godaddy website, after logging in and selecting the domain, you have to select managedns and update the name servers there.

Example Type NS:

- ns-cloud-e1.googledomains.com
- ns-cloud-e2.googledomains.com
- ns-cloud-e3.googledomains.com
- ns-cloud-e4.googledomains.com

On Godaddy>DNS>Nameservers> Change Nameservers & use custom nameservers

- ns-cloud-e1.googledomains.com
- ns-cloud-e2.googledomains.com
- ns-cloud-e3.googledomains.com
- ns-cloud-e4.googledomains.com

On Google Cloud Console web UI, click on the zone you just created, click on **Add Standard**

```bash
DNS name: vcapenv.example.com
```

Select Resource record type as NS and writ the name servers in the  vcapenv-zone

```bash
ns-cloud-a1.googledomains.com.
ns-cloud-a2.googledomains.com.
ns-cloud-a3.googledomains.com.
ns-cloud-a4.googledomains.com.
```

- After this is done, wait for 5 minutes for name server propagation. Then you should be able to see ops manager Web UI at **pcf.vcapenv.example.com**
- Ops Manager has not yet been configured so the first screen you will prompt you to configure authentication.
- Select “internal Authentication”
- Give User name as admin and some password.
- Give any passphrase which has 20 characters
- It will take some time to configure and it will show a login screen.
- Login with your credentials.
- You will be able to observe home page of ops
manager.

:warning: If you delete your project, delete the DNS entries beforehand otherwise when Terraform is going to be used in another project, Google will not allow the resources to be created.
If you have already deleted your project, you can undelete the project, remove entries and then zone, re-delete project.

Congratulations! You have installed Ops Manager on GCP
