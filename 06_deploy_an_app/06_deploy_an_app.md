# Deploy an app

In this lab you will be:

- deploy an app

## Retrieve the UAA credentials

You can use the **cf login** command to connect to your own instance as you did before.

However, this time you will need to use your environment-specific API endpoint and the admin user. Be aware, this user is not the same admin user you used previously to access the Ops Manager.

This new UAA admin account was created by the TAS tile.

Click on the TAS tile in ops manager. Click on Credentials tab. Now go to UAA section and see the credentials of Admin

## Connect to Apps Manager

You can use this credentials to login to apps manager at **apps.sys.vcapenv.example.com**.

```bash
  https://apps.sys.vcapenv.example.com
```

## Login to cloud foundry

Login to cloud foundry using admin credentials

```bash
  cf login -a api.sys.vcapenv.example.com --skip-ssl-validation
```

After logging in, list the orgs:

```bash
  cf orgs
```

You will see that there is only one org with name **system**

## Create an org and space

Now create 1 org using

```bash
  cf create-org my-org
  cf target -o "my-org"
  cf create-space my-space
  cf target -o "my-org" -s "my-space"
```

## Deploy test-app

Clone the repo [cloudfoundry-samples/test-app](https://github.com/cloudfoundry-samples/test-app).

```bash
  git clone https://github.com/cloudfoundry-samples/test-app
  cd test-app/
  cf push
```

The app is deployed

In the apps manager console > Marketplace, you will see that no service is available. In the next lab, we will install MySQL tile.

Congratulations!! You have deployed an application
