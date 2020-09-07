# VPC Networking

## Task 1. Explore the default network

- View the subnets
```
gcloud compute networks list
```
- View the routes
```
gcloud compute routes list
```
- View the firewall rules
```
gcloud compute firewall-rules list
```
- Delete the Firewall rules
```
gcloud compute firewall-rules delete default-allow-icmp
gcloud compute firewall-rules delete default-allow-rdp
gcloud compute firewall-rules delete default-allow-ssh
gcloud compute firewall-rules delete default-allow-internal
```
Confirm the deletion with Y

- Delete the default network
```
gcloud compute networks delete default
```

## Task 2. Create an auto mode network
- Create an auto mode VPC network with firewall rules
```
gcloud compute networks create mynetwork --subnet-mode=auto --bgp-routing-mode=regional
gcloud compute firewall-rules create mynetwork-allow-icmp --network=mynetwork --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp
gcloud compute firewall-rules create mynetwork-allow-internal --network=mynetwork --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all
gcloud compute firewall-rules create mynetwork-allow-rdp --network=mynetwork --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389
gcloud compute firewall-rules create mynetwork-allow-ssh --network=mynetwork --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
```
- List all subnets and get IP address range
```
gcloud compute networks subnets list —mynetwork
```
Record the IP address range for the subnets in us-central1 and europe-west1. These will be referred to in the next steps.
us-central1   10.128.0.0/20
europe-west1  10.132.0.0/20
- Create a VM instance in us-central1
```
gcloud beta compute instances create mynet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=mynetwork --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mynet-us-vm
```
Verify that the Internal IP for the new instance was assigned from the IP address range for the subnet in us-central1 (10.128.0.0/20).
- Create a VM instance in europe-west1
```
gcloud beta compute instances create mynet-eu-vm --zone=europe-west1-c --machine-type=n1-standard-1 --subnet=mynetwork --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mynet-eu-vm
```
Verify that the Internal IP for the new instance was assigned from the IP address range for the subnet in europe-west1 (10.132.0.0/20).
- Verify connectivity for the VM instances
 1. Retrieve the internal and external ip address of mynet-eu-vm
 ```
 gcloud compute instances list mynet-eu-vm
 ```
  2. Connect to mynet-us-vm. Confirm Y to connect.
 ```
 gcloud compute ssh mynet-us-vm --zone=us-central1-c
 ```
 3. To test connectivity to mynet-eu-vm's internal IP, run the following command, replacing mynet-eu-vm's internal IP
 ```
 ping -c 3 <Enter mynet-eu-vm's internal IP here>
 ```
 4. Repeat the same test by running the following
 ```
 ping -c 3 mynet-eu-vm
 ```
 5. To test connectivity to mynet-eu-vm's external IP, run the following command, replacing mynet-eu-vm's external IP
 ```
 ping -c 3 <Enter mynet-eu-vm's external IP here>
 ```

Once done type `exit` to disconnect from SSH connection.

- Convert the network to a custom mode network
```
gcloud compute networks update mynetwork --switch-to-custom-subnet-mode
```
## Task 3. Create custom mode networks
- Create the managementnet network
 1. Create privatenet network:
```
gcloud compute networks create managementnet --subnet-mode=custom --bgp-routing-mode=regional
```
 2. Create the managementsubnet-us subnet:
```
gcloud compute networks subnets create managementsubnet-us --range=10.130.0.0/20 --network=managementnet --region=us-central1
```
- Create the privatenet network
 1. Create privatenet network:
```
gcloud compute networks create privatenet --subnet-mode=custom
```
 2. Create the privatesubnet-us subnet:
```
gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24
```
 3. Create the privatesubnet-eu subnet:
```
gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
```

List the available VPC networks:
```
gcloud compute networks list
```
List the available VPC subnets (sorted by VPC network):
```
gcloud compute networks subnets list --sort-by=NETWORK
```
- Create the firewall rules for managementnet
Create the managementnet-allow-icmp-ssh-rdp firewall rule:
```
gcloud compute firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
```
- Create the firewall rules for privatenet
Create the privatenet-allow-icmp-ssh-rdp firewall rule:
```
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```
List all the firewall rules (sorted by VPC network):
```
gcloud compute firewall-rules list --sort-by=NETWORK
```
- Create the managementnet-us-vm instance
Create the VM Instance:
```
gcloud beta compute instances create managementnet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=managementsubnet-us --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=managementnet-us-vm --reservation-affinity=any
```
- Create the privatenet-us-vm instance
Create the privatenet-us-vm instance:
```
gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=privatesubnet-us
```
List all the VM instances (sorted by zone):
```
gcloud compute instances list --sort-by=ZONE
```
## Task 4. Explore the connectivity across networks
- Ping the external IP addresses
 1. List the internal and external IP address of mynet-eu-vm, managementnet-us-vm, and privatenet-us-vm:
```
gcloud compute instances list mynet-eu-vm
gcloud compute instances list managementnet-us-vm
gcloud compute instances list privatenet-us-vm
```
Write down the IP addresses of the different VM's for use.
 2. Connect to mynet-us-vm. Confirm Y to connect.
```
gcloud compute ssh mynet-us-vm --zone=us-central1-c
```
 3. To test connectivity to mynet-eu-vm's external IP:
```
ping -c 3 <Enter mynet-eu-vm's external IP here>
```
 4. To test connectivity to managementnet-us-vm's external IP:
```
ping -c 3 <Enter managementnet-us-vm's external IP here>
```
 5. To test connectivity to privatenet-us-vm's external IP:
```
ping -c 3 <Enter privatenet-us-vm's external IP here>
```

Once done type `exit` to disconnect from SSH connection.

- Ping the internal IP addresses
 1. List the internal and external IP address of mynet-eu-vm, managementnet-us-vm, and privatenet-us-vm:
```
gcloud compute instances list mynet-eu-vm
gcloud compute instances list managementnet-us-vm
gcloud compute instances list privatenet-us-vm
```
Write down the IP addresses of the different VM's for use.
 2. Connect to mynet-us-vm. Confirm Y to connect.
```
gcloud compute ssh mynet-us-vm --zone=us-central1-c
```
 3. To test connectivity to mynet-eu-vm's internal IP
```
ping -c 3 <Enter mynet-eu-vm's internal IP here>
```
 4. To test connectivity to managementnet-us-vm's internal IP:
```
 ping -c 3 <Enter managementnet-us-vm's internal IP here>
```
 5. To test connectivity to privatenet-us-vm's internal IP:
```
ping -c 3 <Enter privatenet-us-vm's internal IP here>
```

Once done type `exit` to disconnect from SSH connection.








