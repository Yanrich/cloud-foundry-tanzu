# Installing Tanzu Application service

In this lab you will be:

- Installing Tanzu application Service

## Steps to install Tanzu application Service

### Import tile and stemcell to ops manager

Use the om CLI to upload both the TAS tile and its stemcell to your Ops
Manager.

```bash
om upload-stemcell -s light-bosh-stemcell-1.423-google-kvm-ubuntu-jammy-go_agent.tgz
om upload-product -p srt-4.0.21-build.4.pivotal
```

### Manually configure TAS tile

From the ui of Ops manager, stage the TAS tile

When you refresh your Ops Manager dashboard, you will see the staged tile
labeled Small Footprint Tanzu Application Service. The orange stripe at the
bottom of the tile indicates that it requires further configuration before it can be deployed.

Click the tile to configure it.

#### Assign AZs and networks

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
```

#### Domains

```bash
  sys.vcapenv.example.com
  apps.vcapenv.example.com
```

#### Networking

On the Ops Manager, locate the configuration section called Certificates and private keys for the Gorouter and HAProxy.
Click the Add button.
Set the Name to default.
Locate the file you generated earlier which ends .crt and copy its contents
into the Ops Manager text field labelled Certificate PEM
Locate the file you generated earlier which ends .key and copy the contents
into the Ops Manager text field labelled Private Key PEM.
Click Save.

#### App Developer Controls

Default app memory quota per org: 20480

#### App Security groups

Type an X in the box

#### UAA

As per the Networking instructions above, find your certificate and private key and paste them in to the SAML Service Provider Credentials fields

#### Credhub

Add an encryption key in Internal encryption provider keys.
Name the key default.
Choose a secret Key that is at least 20 characters long. There is currently no validation for this in Ops Manager, but the deployment will fail and waste your time.
Check Primary
Click Save

#### Internal MySQL

```bash
  Email address
    admin@example.com
```

#### Errands

Select Off for the following errands:

- Notifications Errand
- Notifications UI Errand
- App Autoscaler Errand
- App Autoscaler Smoke Test Errand
- NFS Broker Errand

#### Resource Config

- Control (diego brain)
  - tcp:vcapenv-cf-ssh
- Router (load balancers)
  - tcp:vcapenv-cf-ws,http:vcapenv-httpslb

:warning: VM names differ for the Small Footprint Runtime product versus the full TAS tile. In Small Footprint Runtime, Diego Cells are called Compute, and the Diego Brain is called Control. See [small-footprint](https://docs.vmware.com/en/VMware-Tanzu-Application-Service/4.0/tas-for-vms/small-footprint.html) for a full description of how the two tiles differ.

- [optional]: In the row labeled Compute, set the number of instances to 3 and the vm type to:

```bash
e2-highmem-2 cpu:2 ram:16 disk: 64
```

### Apply changes

 Click on "Apply changes"

### Post TAS Install with errands

On the jumpbox, you can see the errands available

```bash
  bosh -d `bosh deployments --column=name | grep cf` errands
```

You can run a second time Smoke Test Errand if you want

```bash
  bosh -d `bosh deployments --column=name | grep cf` run-errand smoke_tests
```

### Post TAS Install

Once your TAS install is complete, you can export the configuration

```bash
  om products
  om staged-config --include-credentials -p cf > cf.yml
  om staged-director-config --no-redact > director.yml
```

Congratulations!! You have deployed Tanzu Application Service
