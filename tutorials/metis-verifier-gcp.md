# How to Run a Metis Verifier Node on GCP

The following document describes how to setup a Metis verifier node on GCP.  The primary steps include:

1. Create a GCP account
2. Setup a GCP Billing Account
3. Create a GCP Project
4. Setup a Compute Engine VM Instance
5. Setup VM Instance Firewall Rules
5. Install Git on the VM Instance
6. Download the Metis Verifier Repo
7. Find or Create an ETH RPC Instance
8. Edit Verifier Configuration
9. Start Verifier Docker Containers

Note: Metis verifiers must be approved by the official MetisDAO team before receiving verifier rewards.  For more information, send an email verifier@metis.io.

## Create a GCP Account

Navigate to the [Google Cloud Platform](https://cloud.google.com/) and login with your gmail account.  If you do not have a gmail account, you will need to create one.

Click the Console link to arrive at the cloud platform dashboard.

![Console Button](/tutorials/images/"Screen Shot 2022-06-26 at 4.47.02 PM.jpg")

## Setup a Billing Account

## Create a GCP Project

Google Cloud Platform projects are simply ways to organize any cloud computing tasks required to complete a certain goal; in this case - running a Metis Verifier node.

Click the dropdown arrow at the top toolbar on the GCP dashboard homepage and select "New Project".

Choose a name for the project and select "Create".

## Setup a Compute Engine VM Instance

A Compute Engine VM Instance is a virtual machine hosted by Google.  These machines can be customized with hardware and software specifications as needed.  These machines are simply physical hardware that can be accessed remotely.

Inside the newly created project dashboard, select "Create a VM".  You will be asked to enable the compute engine API.  Select "enable".

Choose the following hardware/software combination for the VM Instance.

### Basic Information 
- Name: Any name is acceptable
- Labels: Skip
- Region & Zone: Any region or zone is acceptable

### Machine Configuration
- Machine Family: General Purpose
- Series: E2
- Machine Type: Custom
- Machine Cores: 6 cores
- Machine Memory: 12 GB
- Display Device: Optional
- Confidential VM Service: Optional
- Deploy Container: Skip

### Boot Disk
- Operating System: Debian
- Version: Debian GNU/Linux 11 (bullseye)
- Boot Disk Type: Balanced Persistent Disk OR SSD Persistent Disk
- Size: 80GB

### Identify and API Access
- Service Accounts: Compute Engine Default Service Account
- Access Scope: Allow Default Access

### Firewall
Verify "Allow HTTP Traffic" AND "Allow HTTS Traffic" are NOT CHECKED.

Leave all other selections default and choose Create.  The following screen will appear displaying the running instance of the new virtual macine.  Make note of the external IP address for later use.

## Setup VM Instance Firewall Rules
Select the Setup Firewall button on the VM Instance dashboard.  It will likely look like the below screenshot.

Select the two rules shown and choose Delete.

Select Add Firewall Rule to allow traffic form Metis HQ for checking the status of a verifier.

- Name: metis-hq-ingress
- Description: Allowing ingress from Metis HQ for verifier node activation.
- Logs: Optional
- Network: Default
- Priority: 1000
- Direction of Traffic: Ingress
- Action on Match: Allow
- Targets: All Instances in the Network
- Source Filter: IPV4 Ranges
- Source IPV4 Range: 3.13.115.31
- Second Source Filter: None
- Protocols and Ports: Specified Protocols & Ports - UDP + TCP - Port 8080

Select Create.

## Install Git on the VM Instance



Navigate back to the VM instance dashboard by clicking the 3 lines in the top left and selecting Cloud Computing, VM Instances.


SSH (secure shell) into the VM Instance by clicking SSH from the dashboard view.  If you are unfamiliar with SSH - it is simply a way to remotely login to the command line interface of a remote device.

When the remote terminal session opens, update & upgrade the Linux packages on the device by running the following two commands.  This is simply best practice when initializing a new linux VM.

`sudo apt-get update`

`sudo apt-get upgrade`

Install `git`
`sudo apt install git`

Install Docker
`sudo apt-get install docker`

Install Docker Compose
`sudo apt-get-install docker-compose`

Convenience - To be able to run docker and docker-compose commands without `sudo`, enter the following.
`sudo groupadd docker`

`sudo usermod -aG docker $USER`

`newgrp docker` 

Note: at the time of this document, there is a docker/docker-compose version bug that will cause the following error when trying to run docker containers.

To work around this error complete the following commands to remove docker-compose and download/install the most recent version (1.29.2 at the time of this tutorial).
`sudo apt-get remove docker-compose`

`sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

`sudo chmod +x /usr/local/bin/docker-compose`

Clone the Metis Verifier repo onto the remote instances
`git clone https://github.com/ericlee42/metis-verifier-node-setup.git`

Move into the repository directory 
`cd metis-verifier-node-setup`

Copy the Metis verifyer docker configuration file to a new file called `docker-compose.yml`
`cp docker-compose-mainnet.yml docker-compose.yml`.

Open the file to edit the necessary ETH RPC parameters 
`DATA_TRANSPORT_LAYER__L1_RPC_ENDPOINT = "<YOUR RPC ENDPOINT>"`
`ETH1_HTTP = "<YOUR RPC ENDPOINT>"`

RPC stands for "Remote Procedure Call".  An RPC endpoint represents a URL or IP address on a network where `clients` can ask for reqponses from `servers`.  In this case, we need a publically available RCP endpoint on the Ethereum network.  This will allow our Metis Verifier to make necessary interactions with the Ethereum network.

Although free Ethereum endpoints exist, they are often rate limited and do not offer sufficient uptime to run a professional Verifier.  Therefore, the best two options are running your own Ethereum node OR working through a service like [Infura.io](https://infura.io/).  If you choose to run your own Ethereum node, review the official [Ethereum Node Documentation](https://ethereum.org/en/run-a-node/) for instructions & options.

If you decide to go the Infura route, creating a project on infura provides an Ethereum RPC endpoint with up to 100,000 requests per day for free. For reference, at the time of this writing, a new Metis Verifier was making approximately 3000-4000 RPC requests per hour.  This number will increase and decrease with network usage.  Infura offers pay plans for > 100,000 requests per day.

From a decentralization persepctive - running a standalone node is always preferred.

Once you have your RPC endpoint, simply add it to the `docker-compose.yml` file in the previously mentioned locations.  The endpoint will be of the form `https://mainnet.infura.io/v3/<random characters>` if you used Infura.

# Starting the Node

Run the Data Transfer Layer Container
`docker-compose up -d dtl-mainnet`

Note: The first time this command is run, the respective container is pulled from the container repository.  After the container is pulled, it should automatically start running.

Verify the Data Transfer Layer Container has Started Successfully
`docker-compose logs dtl-mainnet | grep "L2" -A 5`

The above command should return the following logs:
"""
dtl-mainnet_1     | {"level":30,"time":1656290293461,"url":"https://andromeda.metis.io/?owner=1088","msg":"HTTP Server L2 RPC Provider initialized"}
dtl-mainnet_1     | {"level":30,"time":1656290293461,"msg":"Service L1_Transport_Server has initialized."}
dtl-mainnet_1     | {"level":30,"time":1656290293461,"msg":"Service L1_Ingestion_Service is initializing..."}
dtl-mainnet_1     | {"level":30,"time":1656290293463,"addressManager":"0x918778e825747a892b17C66fe7D24C618262867d","msg":"Using AddressManager"}
dtl-mainnet_1     | {"level":30,"time":1656290294404,"startingL1BlockNumber":13625200,"msg":"Starting sync"}
dtl-mainnet_1     | {"level":30,"time":1656290294505,"msg":"Service L1_Ingestion_Service has initialized."}
"""

Note: If the above command returns nothing, use `docker-compose logs -f dtl-mainnet` to follow the log output live and look for error messages.

Run the Data Transfer Layer Expose Container
`docker-compose up -d dtl-expose`

Run the L2GETH Container
`docker-compose up -d l2geth-mainnet`

Verify All Services are Running Successfully
`curl 'http://localhost:8080/verifier/get/true/1088'`

# What Just Happened?

Next, type `sudo apt install git` to install git.  Git is a universally accepted software version control package.  Git is required because we will be copying the git repository (read code) needed to run a Metis verifier onto our remote compute instance.

To clone the repository onto to the virtual instance, run the following command in the virtual console `git clone https://github.com/ericlee42/metis-verifier-node-setup.git`.  This will download a `metis-verifier-node-setup` folder and all of it's contents onto your remote device.

Within that folder is a Docker configuration file titled `docker-compose-mainnet.yml`.  Docker is a software application that allows developers to "container-ize" thier applications (eeping the environment of each application seperate).  In simple terms - docker containers are efficient ways to package and run application specific code on different devices.  In order to run containerized code - the docker software package is required.  This can be installed with `sudo apt install docker`.

Finally, the Metis verifier code is actually comprised of multiple docker containers.  The `docker-compose` software package is a docker library for managing multiple containers running on a single computer instance.  This can be installed with `sudo apt install docker-compose`.








