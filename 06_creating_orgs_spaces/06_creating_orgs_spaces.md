# Creating orgs, spaces, users and quotas

In this lab you will be:

- creating orgs, spaces, users and quotas

Login to cloud foundry using admin credentials

```bash
  cf login -a api.sys.vcapenv.example.com --skip-ssl-validation
```

After logging in, list the orgs:

```bash
  cf orgs
```

You will see that there is only one org with name **system**

Before we create an org, create a quota with 2G memory:

```bash
  cf create-org-quota myorgquota -m 2G -i 1G -s 1
```

You can list the quotas

```bash
  cf org-quotas
  name         total memory   instance memory   routes   service instances   paid service plans   app instances   route ports   log volume per second
  default      20G            unlimited         1000     100                 allowed              unlimited       0             unlimited
  runaway      100G           unlimited         1000     unlimited           allowed              unlimited       0             unlimited
  myorgquota   2G             1G                0        1                   disallowed           unlimited       0             unlimited  
```

Now create 1 org using

```bash
  cf create-org my-org
  cf target -o "my-org"
  cf create-space my-space
  cf target -o "my-org" -s "my-space"
```

Clone the repo [cloudfoundry-samples/test-app](https://github.com/cloudfoundry-samples/test-app). The app is deployed.

```bash
  git clone https://github.com/cloudfoundry-samples/test-app
  cd test-app/
  cf push
```

If you try to do the same thing with the quota in my-space2, an error occured *Routes quota exceeded for organization 'my-org2'*.

```bash
  cf create-org my-org2 -q myorgquota
  cf target -o "my-org2"
  cf create-space my-space2
  cf target -o "my-org2" -s "my-space2"
```
