# Bosh Deploy Release Lab

## Prepare the environment on the jumpbox

### Install Bosh CLI

- Go to [Bosh CLI](https://bosh.io/docs/cli-v2-install/) and navigate to the [Bosh CLI Github release page](https://github.com/cloudfoundry/bosh-cli/releases)

```bash
wget https://github.com/cloudfoundry/bosh-cli/releases/download/v7.8.6/bosh-cli-7.8.6-linux-amd64 &&
chmod +x bosh-cli-7.8.6-linux-amd64 &&
sudo mv bosh-cli-7.8.6-linux-amd64 /usr/local/bin/bosh
```

### Install OM

- Go to [OM](https://github.com/pivotal-cf/om) and navigate to the [OM Github release page](https://github.com/pivotal-cf/om/releases)

```bash
wget https://github.com/pivotal-cf/om/releases/download/7.14.0/om-linux-amd64-7.14.0 &&
chmod +x om-linux-amd64-7.14.0 &&
sudo mv om-linux-amd64-7.14.0 /usr/local/bin/om
```

### Add env variables

- Add in the .env file

```bash
cat >> ./.env <<-EOF
export OM_TARGET="https://pcf.${ENV_NAME}.${DOMAIN_NAME}"
export OM_SKIP_SSL_VALIDATION="true"
export OM_USERNAME="admin"
export OM_PASSWORD="your_ops_manager_password"
EOF
```

- source environnement

```bash
source ~/.bashrc
```

### Set Bosh env

```bash
cd ~/pivotal-cf-terraforming-gcp-f4aab02/terraforming-pas
terraform output ops_manager_ssh_private_key > ~/.ssh/ops_manager
terraform output ops_manager_ssh_public_key > ~/.ssh/ops_manager.pub
chmod 600 ~/.ssh/ops_manager
om bosh-env #get the env variables
om bosh-env -i ~/.ssh/ops_manager
echo 'eval "$(om bosh-env -i ~/.ssh/ops_manager)"' >> ~/.env
source ~/.bashrc
bosh env
```

- For information, you can type

```bash
env | grep BOSH
```

- it gives you

```bash
BOSH_CLIENT_SECRET
BOSH_CLIENT
BOSH_CA_CERT
BOSH_ALL_PROXY
BOSH_ENVIRONMENT
```

## Deploy nginx

### Upload stemcell

- Upload [stemcell](https://bosh.io/stemcells/)

```bash
bosh upload-stemcell --sha1 35138db48751027e6f71e28511755d9e4b583c66 \
  https://bosh.io/d/stemcells/bosh-google-kvm-ubuntu-jammy-go_agent?v=1.301
```

### Upload nginx release

- Go to [nginx release](https://bosh.io/releases/github.com/cloudfoundry-community/nginx-release?all=1)

- Upload the release to your director with the upload-release command

```bash
bosh upload-release --sha1 59dbc1e8dd5f4c85cca18dce1d5b70f11f9ddfcd \
  https://bosh.io/d/github.com/cloudfoundry-community/nginx-release?v=1.21.6
```

### Practise some commands

- You can test the follwing commands:

```bash
bosh tasks
bosh task <number>
bosh task -r
bosh task -r -- cpi
bosh vms
bosh deployments
bosh stemcells
bosh releases | grep nginx
```

### Create the nginx manifest deploy

- In the [nginx-release GitHub repo](https://https://github.com/cloudfoundry-community/nginx-release), there is a a folder **manifest**
- Download the nginx_ubuntu_centos.yml file

```bash
curl -sLO https://raw.githubusercontent.com/cloudfoundry-community/nginx-release/master/manifests/nginx_ubuntu_centos.yml && mv nginx_ubuntu_centos.yml nginx_cloudfoundry.yml
```

- Remove the section related to centos
- To have information about vm_type, you can check the bosh config

```bash
bosh cloud-config | less
```

- the final version of the nginx file is available in the manifest folder in this section
- Create a folder manifests on our jumpobx and put the nginx.yml file with the [cat command](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/#4)

```bash
cat > nginx.yml
```

### Deploy nging using the yaml file

- Deploy nginx

```bash
bosh deploy -d nginx nginx.yml
```

- Retrieve logs

```bash
bosh -d nginx logs
tar -tzvf nginx-20231119-044404-334908227.tgz
```

## Inside VM

- access to the VM

```bash
bosh -d nginx ssh nginx/0
```

- to validate is nginx is running

```bash
nginx/d7be3cfb-1a38-4690-9ed0-4cfc37c78477:~$ ps -ef | grep nginx
```

- to see the nginx index.html

```bash
nginx/d7be3cfb-1a38-4690-9ed0-4cfc37c78477:~$ curl localhost
<html><body><p>Hello world</p></body></html>
```

- take a look at /var/cvcap

```bash
nginx/d7be3cfb-1a38-4690-9ed0-4cfc37c78477:/var/vcap$ ls -al
total 44
drwxr-xr-x 11 root root 4096 Jul  1 20:47 .
drwxr-xr-x 12 root root 4096 Jun 29 16:12 ..
drwx------  6 root root 4096 Jul  1 20:40 bosh
drwxr-xr-x  3 root root 4096 Jul  2 01:27 bosh_ssh
drwxr-xr-x 16 root root 4096 Jul  1 20:41 data
drwxr-xr-x  3 root root 4096 Jul  1 20:40 instance
drwxr-x---  2 root vcap 4096 Jul  1 20:41 jobs
drwxr-xr-x  3 root root 4096 Jun 29 16:27 micro_bosh
drwxr-xr-x  5 root root 4096 Jul  1 20:41 monit
drwxr-xr-x  2 root vcap 4096 Jul  1 20:41 packages
drwxr-xr-x  3 root root 4096 Jul  1 20:41 store
lrwxrwxrwx  1 root root   18 Jul  1 20:40 sys -> /var/vcap/data/sys
```

- Go to [Configuration Locations](https://bosh.io/docs/vm-config/) with **vcap** equal **vmware cloud application platform**

| Mount | Description |
|---|---|
| /var/vcap/bosh | Core Bosh software, e.g bosh-agent,monit |
| /var/vcap/jobs | Configuration files, control scripts and programs |
| /var/vcap/packages | deployed software packages that will be run on the VM |
| /var/vcap/sys/log | log files |
| /var/vcap/data | Ephemeral data (will not be preserved if VM is destroyed) |
| /var/vcap/store | Persitent data |

- Understand files

```bash
nginx/d7be3cfb...77:/var/vcap$ sudo -s
nginx/d7be3cfb...77:~# cd /var/vcap/

nginx/d7be3cfb...77:~# pstree

nginx/d7be3cfb...77:/var/vcap# find . -name nginx.conf
./data/jobs/nginx/d7be3cfb...77/etc/nginx.conf
./data/packages/nginx/d7be3cfb...77/conf/nginx.conf

nginx/d7be3cfb...77:/var/vcap# cat /var/vcap/store/nginx/index.html 
<html><body><p>Hello world</p></body></html>

nginx/d7be3cfb...77:/var/vcap# find . -name ctl
./data/jobs/loggr-system-metrics-agent/03c70319c790ddc032ec6b002d96a41bd0ec80fa/bin/ctl
./data/jobs/nginx/1299f392e1ec6c1eefcdbb6ae76ee8615dbc654c/bin/ctl

nginx/d7be3cfb...77:/var/vcap# find . -name monit
./bosh/bin/monit
./monit
./data/jobs/bosh-dns/4a8b5b99b27b0f24c6b0449dcc370db58f3902ca/monit
./data/jobs/loggr-system-metrics-agent/03c70319c790ddc032ec6b002d96a41bd0ec80fa/monit
./data/jobs/nginx/1299f392e1ec6c1eefcdbb6ae76ee8615dbc654c/monit

nginx/d7be3cfb...77:/var/vcap# cat jobs/nginx/monit 
check process nginx
  with pidfile /var/vcap/sys/run/nginx/nginx.pid
  start program "/var/vcap/jobs/nginx/bin/ctl start"
  stop program "/var/vcap/jobs/nginx/bin/ctl stop"
  group vcap
```

- Stop process with monit -> the process is restarted by monit

```bash
/var/vcap/jobs/nginx/bin/ctl stop && watch monit status
```

- bosh agent

```bash
nginx/d7be3cfb-1a38-4690-9ed0-4cfc37c78477:/var/vcap# tail -f /var/vcap/bosh/log/current 
```

- bosh agent and monit

```bash
grep --after-context 4 'monit status' /var/vcap/bosh/log/current 
```

- stop monit to see the log and restart it after

```bash
nginx/d7be3cfb-1a38-4690-9ed0-4cfc37c78477:/var/vcap# sv stop monit
```

- stop bosh-agent

```bash
killall bosh-agent && sleep 10 &&
grep grep --after-context 500 'Starting agent' /var/vcap/bosh/log/current | \
grep DEBUG | grep --invert-match 'Cmd Runner\|File System\|ConcreteUdevDevice'
```

```bash
grep grep --after-context 3 "Sending hm message 'heartbeat'" \
    /var/vcap/bosh/log/current
killall nginx    
grep grep --after-context 3 "Sending hm message 'alert'" \
    /var/vcap/bosh/log/current    
```

### Persitent storage

- Modify index.html and stop bosh agent to replace the vm -> health_monitor will recreate the VM

```bash
cat /var/vcap/store/nginx/index.html
sudo -s vi /var/vcap/store/nginx/index.html # Hello Universe
curl localhost
sudo sv stop agent
while true; do sleep 5; date; done
```

- Bosh detect that the VM is unresponsive and recreate a new one

```bash
~. # close the SSH connection and retourn to your jumpbox command line terminal 
watch -n 5 bosh tasks
bosh -d nginx ssh nginx/0 -c "curl localhost"
#stdout | <html><body><p>Hello world</p></body></html>
```

- Deploy nginx_with_persistent_disk.yml to add a persitent disk

```bash
bosh deploy -d nginx nginx_with_persistent_disk.yml
```

- Modification on index.html -> Hello Universe is kept

```bash
df -h # /var/vcap/store is now a separate mount point
sudo -s vi /var/vcap/store/nginx/index.html
curl localhost
<html><body><p>Hello Universe</p></body></html>
sudo sv stop agent
while true; do sleep 5; date; done
~. # close the SSH connection and retourn to your jumpbox command line terminal 
watch -n 5 bosh tasks
bosh -d nginx ssh nginx/0 -c "curl localhost"
#stdout | <html><body><p>Hello world</p></body></html>
```

- Cleanup to remove releases and stemcells no longer used

```bash
bosh clean-up --all
```

- Delete nginx deployment -> disk of 1 GB is still there, automatically destroy after 5 days unless we use clean-up --all

```bash
bosh delete-deployment -d nginx
bosh disks --orphaned
```
