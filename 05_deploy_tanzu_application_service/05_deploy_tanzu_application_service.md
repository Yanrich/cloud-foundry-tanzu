# Installing Tanzu Application service

In this lab you will be:

- Installing Tanzu application Service

## Steps to install Tanzu application Service

### Om installation

See [om tool](https://github.com/pivotal-cf/om)

```bash
wget https://github.com/pivotal-cf/om/releases/download/7.9.0/om-linux-amd64-7.9.0
chmod +x om-linux-amd64-7.9.0
sudo mv om-linux-amd64-7.9.0 /usr/local/bin/om
```

Define environment variables that pre-configure your target om url and
credentials.
Add the following lines to your ~/.env file:

```bash
export OM_USERNAME="admin"
export OM_PASSWORD="******" # replace with your ops manager password
```

### Import tile and stemcell to ops manager

Use the om CLI to upload both the TAS tile and its stemcell to your Ops
Manager.

```bash
om upload-stemcell -s light-bosh-stemcell-621.543-google-kvm-ubuntu-xenial-go_agent.tgz
om upload-product -p srt-2.11.39-build.2.pivotal
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
Locate the file you generated earlier which ends .cert and copy its contents
into the Ops Manager text field labelled Certificate PEM
Locate the file you generated earlier which ends .key and copy the contents
into the Ops Manager text field labelled Private Key PEM
Choose Disable for HAProxy forwards requests to Router over TLS
Click Save.

#### App Developer Controls

Default app memory quota per org: 20480

#### App Security groups

Type an X in the box

#### UAA

As per the Networking instructions above, find your certificate and private key and paste them in to the SAML Service Provider Credentials fields

#### Credhub

Add an encryption key in Internal encryption provider keys
Name the key default.
Choose a secret Key that is at least 20 characters long. There is currently no validation for this in Ops Manager, but the deployment will fail and waste your time.
Check Primary
Click Save

#### Internal MySQL

email: <admin@example.com>

#### Errands

Select Off for the following errands:
Notifications Errand
Notifications UI Errand
App Autoscaler Errand
App Autoscaler Smoke Test Errand
NFS Broker Errand

#### Resource Config

- Control (diego brain)
  - tcp:vcapenv-cf-ssh
- Router (load balancers)
  - tcp:vcapenv-cf-ws,http:vcapenv-httpslb

:warning: VM names differ for the Small Footprint Runtime product versus the full TAS tile. In Small Footprint Runtime, Diego Cells are called Compute, and the Diego Brain is called Control. See [small-footprint](https://docs.pivotal.io/application-service/2-10/operating/small-footprint.html) for a full description of how the two tiles differ.

Once your TAS install is complete, you can use the **cf login** command to
connect to your own instance as you did before. However, this time you will
need to use your environment-specific API endpoint and the admin user. Be
aware, this user is not the same admin user you used previously to access the
Ops Manager.
This new UAA admin account was created by the TAS tile .
Click on the TAS tile in ops manager. Click on Credentials tab. Now go to UAA
section and see the credentials of Admin
You can use this credentials to login to apps manager at **apps.vcapenv.example.com**.

Congratulations!! You have deployed Tanzu Application Service
