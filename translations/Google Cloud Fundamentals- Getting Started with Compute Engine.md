# Google Cloud Fundamentals: Getting Started with Compute Engine

> Course: Google Cloud Platform Fundamentals - Core Infrastructure
> Module: Virtual Machines in the Cloud
> 

## Task 1: Sign in to the Google Cloud Platform (GCP) Console
- Create my-vm-1 instance:
```
gcloud beta compute instances create my-vm-1 --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=my-vm-1 --reservation-affinity=any
```
- Create Allow HTTP traffic in firewall:
```
gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

## Task 2: Create a virtual machine using the gcloud command line
- List of all the zones in the region:
```
gcloud compute zones list | grep us-central1
```
- Choose a zone from that list other than the zone to which Qwiklabs assigned you
- Set your default zone to the one you just chose:
```
gcloud config set compute/zone us-central1-b
```

- Create a VM instance called my-vm-2 in that zone:

```
gcloud compute instances create "my-vm-2" \
--machine-type "n1-standard-1" \
--image-project "debian-cloud" \
--image "debian-9-stretch-v20190213" \
--subnet "default"
```
## Task 3: Connect between VM instances
- Connect to my-vm-2 VM using SSH:
```
gcloud compute ssh my-vm-2 --zone=us-central1-b
```
- Use the ping command to confirm that my-vm-2 can reach my-vm-1 over the network:
```
ping my-vm-1
```
Notice that the output of the ping command reveals that the complete hostname of my-vm-1 is my-vm-1.c.PROJECT_ID.internal, where PROJECT_ID is the name of your Google Cloud Platform project. GCP automatically supplies Domain Name Service (DNS) resolution for the internal IP addresses of VM instances.
- Press Ctrl+C to abort the ping command
- Use the ssh command to open a command prompt on my-vm-1
```
ssh my-vm-1
```
If you are prompted about whether you want to continue connecting to a host with unknown authenticity, enter yes to confirm that you do.
- Install the Nginx web server:
```
sudo apt-get install nginx-light -y
```
- Use the nano text editor to add a custom message to the home page of the web server:
```
sudo nano /var/www/html/index.nginx-debian.html
```
- Use the arrow keys to move the cursor to the line just below the h1 header. Add text like this, and replace YOUR_NAME with your name.
- Press Ctrl+O and then press Enter to save your edited file, and then press Ctrl+X to exit the nano text editor.
- Confirm that the web server is serving your new page. At the command prompt on my-vm-1:
```
curl http://localhost/
```
- To exit the command prompt on my-vm-1:
```
exit
```
You will return to the command prompt on my-vm-2
- To confirm that my-vm-2 can reach the web server on my-vm-1, at the command prompt on my-vm-2:
```
curl http://my-vm-1/
```
- Get external IP address of VM:
```
gcloud compute instances list my-vm-1
```
- Use external IP address and use curl commands to view website:
```
curl http://[EXTERNAL IP]/
```






