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

Note: Metis verifiers must be approved by the official MetisDAO team before receiving verifier rewards.  For more information, send an email to verifier@metis.io.  Also note, this tutorial borrows heavily from the original (yet somewhat out of date) [Metis Verifier Tutorial](https://github.com/ericlee42/metis-verifier-node-setup).

It should also be noted that running a verifier node on "bare metal" (your own computer) is preferred when compared to centralized cloud providers.  Although centralized providers offer uptime guarantees and convenience - they also become single points of failure.  Chains must become less dependent on centralized providers for adequate decentralization to occur - encouraging Verifiers to explore their own hardware & decentralized cloud providers (such as [Askash](https://akash.network/)).

## Create a GCP Account

Navigate to the [Google Cloud Platform](https://cloud.google.com/) and login with your gmail account.  If you do not have a gmail account, you will need to create one.

Click the Console link to arrive at the cloud platform dashboard.

![console_button](/tutorials/images/console_button.jpg)

## Setup a GCP Billing Account

Navigate to the billing page within GCP and add a credit card.  At the time of this tutorial - a virtual machine with required Metis Verifier compute power will cost approximately $140 per month from GCP.

![billing](/tutorials/images/billing.jpg)


## Create a GCP Project

Google Cloud Platform projects are simply ways to organize any cloud computing tasks required to complete a certain goal. In this casem, running a Metis Verifier node.

Click the dropdown arrow at the top toolbar on the GCP dashboard homepage and select "New Project".

![create_project](/tutorials/images/create_project.jpg)

Choose a name for the project and select "Create".

![project_name](/tutorials/images/project_name.jpg)

## Setup a Compute Engine VM Instance

A Compute Engine VM Instance is a virtual machine hosted by Google.  These machines can be customized with hardware and software specifications as needed.  These machines are simply physical hardware that can be accessed remotely.

Inside the newly created project dashboard, select "Create a VM".  

![project_dashboard](/tutorials/images/project_dashboard.jpg)

You will be asked to enable the compute engine API.  Select "enable".

![compute_engine](/tutorials/images/compute_engine.jpg)

Choose the following hardware/software combination for the VM Instance.

### Basic Information 
- **Name:** Any name is acceptable
- **Labels:** Skip
- **Region & Zone:** Any region or zone is acceptable

### Machine Configuration
- **Machine Family:** General Purpose
- **Series: E2**
- **Machine Type:** Custom
- **Machine Cores:** 6 cores
- **Machine Memory:** 12 GB
- **Display Device:** Optional
- **Confidential VM Service:** Optional
- **Deploy Container:** Skip

### Boot Disk
- **Operating System:** Debian
- **Version:** Debian GNU/Linux 11 (bullseye)
- **Boot Disk Type:** Balanced Persistent Disk OR SSD Persistent Disk
- **Size:** 80GB

### Identify and API Access
- **Service Accounts:** Compute Engine Default Service Account
- **Access Scope:** Allow Default Access

### Firewall
Verify "Allow HTTP Traffic" AND "Allow HTTS Traffic" are **NOT CHECKED**.

Leave all other selections default and choose Create.  The following screen will appear displaying the running instance of the new virtual macine.  Make note of the external IP address for later use.

![vm_instance_dash](/tutorials/images/vm_instance_dash.jpg)

## Setup VM Instance Firewall Rules
Select the Setup Firewall button on the VM Instance dashboard.  

![firewall_button](/tutorials/images/firewall_button.jpg)

It will likely look like the below screenshot.

![firewall_rules](/tutorials/images/firewall_rules.jpg)

Select the two rules shown and choose Delete.

Select Add Firewall Rule to allow traffic form Metis HQ for checking the status of a verifier.

- **Name:** metis-hq-ingress
- **Description:** Allowing ingress from Metis HQ for verifier node activation.
- **Logs:** Optional
- **Network:** Default
- **Priority:** 1000
- **Direction of Traffic:** Ingress
- **Action on Match:** Allow
- **Targets:** All Instances in the Network
- **Source Filter:** IPV4 Ranges
- **Source IPV4 Range:** 3.13.115.31
- **Second Source Filter:** None
- **Protocols and Ports:** Specified Protocols & Ports - UDP + TCP - Port 8080

Select **Create**.

## Install Git on the VM Instance

Navigate back to the VM instance dashboard by clicking the 3 lines in the top left and selecting Compute Engine -> VM Instances.

![virtual_menu](/tutorials/images/virtual_menu.jpg)

SSH (secure shell) into the VM Instance by clicking SSH from the dashboard view.  If you are unfamiliar with SSH - it is simply a way to login to the command line interface of a remote device.

![vm_instance_dash](/tutorials/images/vm_instance_dash.jpg)

When the remote terminal session opens, update & upgrade the Linux packages on the device by running the following two commands.  This is simply best practice when initializing a new Linux VM.

`sudo apt-get update`

`sudo apt-get upgrade`

## Install Git

`sudo apt install git`

## Install Docker

`sudo apt-get install docker`

## Install Docker Compose

`sudo apt-get install docker-compose`

Convenience - To be able to run docker and docker-compose commands without `sudo`, enter the following.

`sudo groupadd docker`

`sudo usermod -aG docker $USER`

`newgrp docker` 

Note: at the time of this document, there is a docker/docker-compose version bug that will cause the following error when trying to run docker containers.

![docker_error](/tutorials/images/docker_error.jpg)

To work around this error complete the following commands to remove docker-compose and download/install the most recent version (1.29.2 at the time of this tutorial).

`sudo apt-get remove docker-compose`

`sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

`sudo chmod +x /usr/local/bin/docker-compose`

## Clone the Metis Verifier Repo

`git clone https://github.com/ericlee42/metis-verifier-node-setup.git`

## Move Into the Repository Directory 

`cd metis-verifier-node-setup`

## Copy the Metis Verifyer Docker Configuration File

`cp docker-compose-mainnet.yml docker-compose.yml`

## Open the File to Edit the Necessary ETH RPC Parameters 

`DATA_TRANSPORT_LAYER__L1_RPC_ENDPOINT = "<YOUR RPC ENDPOINT>"`

`ETH1_HTTP = "<YOUR RPC ENDPOINT>"`

RPC stands for "Remote Procedure Call".  An RPC endpoint represents a URL or IP address on a network where `clients` can ask for reqponses from `servers`.  In this case, we need a publically available RCP endpoint on the Ethereum network.  This will allow our Metis Verifier to make necessary interactions with the Ethereum network.

Although free Ethereum endpoints exist, they are often rate limited and do not offer sufficient uptime to run a professional Verifier.  Therefore, the best two options are running your own Ethereum node OR working through a service like [Infura.io](https://infura.io/).  If you choose to run your own Ethereum node, review the official [Ethereum Node Documentation](https://ethereum.org/en/run-a-node/) for instructions & options.

If you decide to go the Infura route, creating a project on infura provides an Ethereum RPC endpoint with up to 100,000 requests per day for free. For reference, at the time of this writing, a new Metis Verifier was making approximately 3000-4000 RPC requests per hour.  This number will increase and decrease with network usage.  Infura offers pay plans for > 100,000 requests per day.

From a decentralization persepctive - running a standalone node is always preferred.

Once you have your RPC endpoint, simply add it to the `docker-compose.yml` file in the previously mentioned locations.  

The endpoint will be of the form `https://mainnet.infura.io/v3/<random characters>` if you used Infura.

# Starting the Node

## Run the Data Transfer Layer Container.

`docker-compose up -d dtl-mainnet`

Note: The first time this command is run, the respective container is pulled from the container repository.  After the container is pulled, it should automatically start running.

## Verify the Data Transfer Layer Container has Started Successfully.

`docker-compose logs dtl-mainnet | grep "L2" -A 5`

The above command should return the following logs:

```
dtl-mainnet_1     | {"level":30,"time":1656290293461,"url":"https://andromeda.metis.io/?owner=1088","msg":"HTTP Server L2 RPC Provider initialized"}
dtl-mainnet_1     | {"level":30,"time":1656290293461,"msg":"Service L1_Transport_Server has initialized."}
dtl-mainnet_1     | {"level":30,"time":1656290293461,"msg":"Service L1_Ingestion_Service is initializing..."}
dtl-mainnet_1     | {"level":30,"time":1656290293463,"addressManager":"0x918778e825747a892b17C66fe7D24C618262867d","msg":"Using AddressManager"}
dtl-mainnet_1     | {"level":30,"time":1656290294404,"startingL1BlockNumber":13625200,"msg":"Starting sync"}
dtl-mainnet_1     | {"level":30,"time":1656290294505,"msg":"Service L1_Ingestion_Service has initialized."}
```

Note: If the above command returns something different, use `docker-compose logs -f dtl-mainnet` to follow the log output live and look for error messages.

## Run the Data Transfer Layer Expose Container.

`docker-compose up -d dtl-expose`

More info to come...

## Run the L2GETH Container.

`docker-compose up -d l2geth-mainnet`

More info to come...

## Verify All Services are Running Successfully

`curl 'http://localhost:8080/verifier/get/true/1088'`

More info to come...

## Wait for Node Sync

With all containers running - the new Metis Verifier Node is attempting to sync with the Ethereum network.  To complete this action, the node needs to download historical L1 transaction data (read - the actual Ethereum blockchain).  To check the status of this sync, complete the following.  Note: depending on node connection and hardware, this sync is expected to take multiple hours.

The following command will search the last 10 lines of the l2geth-mainnet container and return a line with the word `blocknumber`.

`docker-compose logs --tail 10 l2geth-mainnet | grep blocknumber`

Result:

```
l2geth-mainnet_1  | INFO [06-28|01:04:08.810] New block                                index=253130 l1-timestamp=1641658421 l1-blocknumber=13965770 tx-hash=0x42a072845627cef4ee7d298a8db97cbae87a315309244a294080e1b83cf55b99 queue-orign=sequencer gas=1196003    fees=0.020332051     elapsed=4.699ms
```

Compare the block number returned (`13965770` in the above example) to the most recent block number found on this [Etherscan Transaction Page](https://etherscan.io/address/0xf209815e595cdf3ed0aaf9665b1772e608ab9380).  When the block number of the Metis Verifier node reaches this block number - the node is sync'd.

Alternatively, the following command will return a hash associated with the most recently sync'd transaction.  When this hash is equivalent to the most recent has at the above link - the node is sync'd.

`curl 'http://localhost:8080/verifier/get/true/1088' | jq '.batch.l1TransactionHash'`

## Contact Metis Team
Once you have verified all the above steps are complete, contact the Metis team at verifier@metis.io and provide your ERC20 wallet address + virtual machine external IP address.

-------------
# What Just Happened?

Congratulations, with the exception of waiting on the Metis team to verify your verifier is accepted (pun intended) you're now running a Metis Verifier node!  

But what exactly did you just do?  Well, you created a Google Cloud Account. This allowed you to spin up a virual machine in the cloud.  You then logged into that virual machine over SSH (secure shell) and downloaded Git. Git is a universally accepted software version control package.  Git was required to copy the git repository (read code) from the Metis codebase to the virtual compute instance you created.

Next, becuase the Metis Verifier code is "containerized", you installed Docker.  Docker is an infrastructure software that allows developers to "containerize" thier applications (keeping the environment of each application seperate).  In simple terms, docker containers are efficient ways to package and run application specific code on different devices.

Additionally, the Metis Verifier code is actually comprised of multiple docker containers.  So you installed Docker Compose - an additional package specifically designed to simplify running multiple containers.

Finally, after cloning the codebase with git and installing docker + docker compose - you edited the main docker configuration file with an acceptable RPC endpoint and started the first container `dtl-mainnet`.  After you checked the logs to verify this container started correctly - you ran two additional containers `dtl-expose` and `l2geth-mainnet`.

# Usefual Extras

This section will include useful extra pieces of information as I learn more about Metis and Optimistic Rolloups in general.

## L2Geth

Geth is a command line interface package for running an Ethereum node (and interacting with the Ethereum network) implemented in the Go language.  L2Geth is a modified version of Geth specifically created for Optimistic L2s.

The offical Metis Verifier Documentation states that the L2Geth container does the following:

1. Gets states from DTL service
2. Reconstructs blocks locally
3. Provides a Web3 Interface for Your Applications

To be completed -> Provide more detail on the above.

## The Nuts and Bolts

If you have any interest in understanding more parts of the Optimistic Rollup puzzle - check out [How does Optimism's Rollup Really Work?](https://research.paradigm.xyz/optimism#optimistic-geth)






