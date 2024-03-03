# Creating orgs, spaces, users and quotas

In this lab you will see:

- concepts of orgs, spaces and roles
- creating orgs, spaces, users and quotas

## Planning orgs and spaces in Cloud Foundry

See the documentation [Planning orgs and spaces in Cloud Foundry](https://docs.cloudfoundry.org/concepts/orgs-and-spaces.html) and [Orgs, spaces, roles, and permissions in Cloud Foundry](hhttps://docs.vmware.com/en/VMware-Tanzu-Application-Service/4.0/tas-for-vms/roles.html)

### Orgs

An org is a development account that an individual or multiple collaborators can own and use. All collaborators access an org with user accounts, which have roles such as Org Manager, Org Auditor, and Org Billing Manager. Collaborators in an org share a resource quota plan, apps, services availability, and custom domains.

### Spaces

A space provides users with access to a shared location for app development, deployment, and maintenance. An org can contain multiple spaces. Every app, service, and route is scoped to a space. Roles provide access control for these resources and each space role applies only to a particular space.

### User roles

A user account represents an individual person within the context of a TAS for VMs foundation. A user can have one or more roles. These roles define the user’s permissions in orgs and spaces.

Roles can be assigned different scopes of User Account and Authentication (UAA) privileges.

The following describes each type of user role in TAS for VMs:

- Admin: Perform operational actions on all orgs and spaces using the Cloud Controller API
- Admin Read-Only: Read-only access to all Cloud Controller API resources
- Global Auditor: Read-only access to all Cloud Controller API resources except for secrets, such as environment variables
- Org Managers: Administer the org
- Org Auditors: Read-only access to user information and org quota usage information
- Org Users: Read-only access to the list of other org users and their roles
- Space Managers: Administer a space within an org
- Space Developers: Manage apps, services, and space-scoped service brokers in a space
- Space Auditors: Read-only access to a space
- Space Supporters: Troubleshoot and debug apps and service bindings in a space

### User role permissions

See [User role permissions](https://docs.vmware.com/en/VMware-Tanzu-Application-Service/4.0/tas-for-vms/roles.html#user-role-permissions-3)


## Planning orgs and spaces in Cloud Foundry



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

Clone the repo [cloudfoundry-samples/test-app](https://github.com/cloudfoundry-samples/test-app).

```bash
  git clone https://github.com/cloudfoundry-samples/test-app
  cd test-app/
  cf push
```

The app is deployed.

If you try to do the same thing with the **myorgquota** quota in my-space2, an error occured *Routes quota exceeded for organization 'my-org2'*.

```bash
  cf create-org my-org2 -q myorgquota
  cf target -o "my-org2"
  cf create-space my-space2
  cf target -o "my-org2" -s "my-space2"
```
