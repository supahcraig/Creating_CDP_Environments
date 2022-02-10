THIS HAS BEEN RELOCATED TO CLDR/Building Environments/CDP 60 Day Trial/

# How to install CDP PVC base 60 day trial on AWS

## Build out AWS Environment

### EC2/VPC
stand up 3 Centos8 EC2 instances on m5.2xlarge
You'll need a VPC with subnets that provide public IP's

This is the image I used.  Other images may work, each with their own unique challenges.
Community AMI:  CentOS 8.2.2004 x86_64 - ami-000e7ce4dd68e7a11

I turned this into a launch template that also performs a few configuration steps. 
cnelson2-PVCBase-LaunchTemplate which can be found in the SE sandbox on AWS (not the public cloud sandbox)

### Security Group
Make sure you can SSH to all 3 nodes from your home IP
Add a rule to your security group to allow all/any trafic from that security group.  (Basically allow access from itself)


### Centos additional setup
#### Yum Repo Update
(this is now baked into my Launch Template)

Followed information found at this link:  https://forketyfork.medium.com/centos-8-no-urls-in-mirrorlist-error-3f87c3466faa

Run this on each node to allow yum installs to work:

```
sudo sed -i -e "s|mirrorlist=|#mirrorlist=|g" /etc/yum.repos.d/CentOS-*
sudo sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*
```

#### Swapiness & Defrag
(this is now baked into my Launch Template)
Run this on each node to allow the Cloudera install to work:

```
sudo sysctl vm.swappiness=10
echo "vm.swappiness=10" >> /etc/sysctl.conf

sudo echo never > /sys/kernel/mm/transparent_hugepage/defrag
sudo echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

## Install Cloudera Manager

https://www.cloudera.com/downloads/cdp-private-cloud-trial.html
>> CDP private cloud base, try now
>> try now
>> accept/submit

This will get you to the URL for the CDP bin to download
https://www.cloudera.com/downloads/cdp-private-cloud-trial/cdp-private-cloud-base-trial.html  (unsure if you can go directly here)

The URL found from the above steps is what is used in the wget command below

** On node 1
```
yum install -y wget`

wget https://archive.cloudera.com/cm7/7.4.4/cloudera-manager-installer.bin
chmod u+x cloudera-manager-installer.bin
sudo ./cloudera-manager-installer.bin
```

accept license...

## Create a Cluster
Then go to node 1 public IP in browser, port 7180 (cloudera manager)
admin/admin

* Cluster setup
https://docs.cloudera.com/cdp-private-cloud-base/7.1.7/installation/topics/cdp-quick-start-deployment-streams-install-runtime.html

60 day trial

put in all 3 hosts comma separated.  Public IP, private hostname, whatever, it all will work..
Hit search, it should find all your hosts.




Cloudera Rpository, use parcels, most recent CDH runtime, no additional parcels

Cloudera provided JDK

Another user:  centos
all hosts accept same private key, choose your pem file, no passphrase.

Install will go mostly unattended.

### Add Services




## Clear health issues

### Time stuff
Run all this on each server:

```
yum install -y chronyd
echo "server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4" >> /etc/chrony.conf
systemctl restart chronyd
```


