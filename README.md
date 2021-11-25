# cromwell_google_setup

I mostly followed [this github page](https://github.com/atgu/cromwell_google_setup).

## Create VM and google bucket specific for cromwell server.

Here `long-covid-hg-cromwell` and gs://long-covid-hg-cromwell 

```
gcloud compute ssh --ssh-flag="-X" long-covid-hg-cromwell --zone us-central1-b
```

## Install needed tools

1. Update system: `sudo apt update` (after `sudo su`)
2. Install docker by following instructions at https://docs.docker.com/install/linux/docker-ce/ubuntu/. (Specifically sections "Install using the repository" and "INSTALL DOCKER ENGINE - COMMUNITY". Ubuntu = amd64)
3. Install java: `sudo apt-get install default-jre`
4. Install docker-compose:

```
sudo curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Install cromwell server

5. Clone cromwell
```
git clone https://github.com/broadinstitute/cromwell.git
```
6. Create a directory to hold Google service account credentials which are used to run the cromwell jobs and change ownership to your account.
```
sudo mkdir credentialdir
sudo chown [username] credentialdir
```
7. Create credential json file called default_credentials.json
```
gcloud iam service-accounts keys create credentialdir/default_credentials.json --iam-account [service account name]
```
The instruction how to create `[service account name]` is [here](https://cloud.google.com/iam/docs/creating-managing-service-accounts)

8. Change directories via: `cd cromwell/scripts/docker-compose-mysql`
Copy configuration template from git config/application.conf to `compose/cromwell/app-config/`

9.Modify following rows to appropriate values. Bucket should be a regional bucket in the same region so the cromwell server is a) the most efficient and b) does not incur data transfer charges!. Zones control the default zone of the VM's cromwell is spinning up. Zones should be the same zone where the cromwell root is to avoid egress charges and for fast data transfer. These can be changed in wdl runtime attributes per task.

These will be specific to your tasks. Examples as follows:

```
application-name = "Name of application"

call-caching {
  enabled = true
}
project = "long-covid-hg"
root = "gs://long-covid-hg-cromwell/"

zones: ["us-central1-b"]
```

10. Add yourself to docker group to be able to issue docker commands

```
sudo usermod -a -G docker $USER
```


## Start up Cromwell

While logged into the Cromwell VM, Build the docker compose image:

`docker-compose build`

Start the Cromwell server docker services

`docker-compose up -d`
