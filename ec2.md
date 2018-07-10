# EC2 notes

resizable computing in the cloud

"Elastic Compute Cloud"

get new servers in minutes

capacity can scale up and down, and also scale out w/ a load balancer

pay for the capacity that you actually use

## pricing options

On demand - fixed rate by the hour (or second) with no commitment

Good for no commitment, unpredicatble workload, and for dev/testing. We'll use this for learning

Reserved - you get a capacity reservation w/ amazion. get a discount on the hourly charge but you're locked in for your term

Good for predictable, steady usage. Can pay up front for discounts. up to 75% cheaper than on-demand. Can also have instances that are convertable between different types, or scheduled reservations.

Spot - lets you bid a price for your instance capacity. Price moves around all the time like a stock market, but it can save you a lot if your timing is flexible

Good for if your app is only profitable at low compute prices. i.e. genome computing at 4 AM on Sundays

Dedicated hosts - physical servers dedicated for your own use. Lets you use server-bound licenses.

Good for if you aren't allowed to use multi-tenant virtualization because of laws.

## Instance types

There's a lot of instance types (around 11) don't need to memorize them for this exam but some of the others might need them

T2 is for low cost, general purpose computing. That's what we're going to use for this class.

Letter is for type, number is for "versions"

pneumonic is FIGHT DR MCPX. Don't need this yet but it is interesting.

## EBS

Virtual disk in the cloud.

Block storage that you can attach to ec2 instances.

they are placed in specific AZs and it automatically replicated.

You get a root device volume attached to your EC2 instance, and then you can mount more volumes.

Types:

* GP2 - general purpose SSD. goes up to 10k IOPS plus burst if necessaey
* IO1 - Provisions SSD. Used for if you have more than 10k IOPS. Used for databases and stuff
* ST1 - throughput optimized HDD. Used for big data. Can't be used as a boot volume.
* SC1 - Cold HDD. Low cost. Can't be used as boot
* Magnetic - low cost boot drive. Deprecated but still available.

## Exam tips

Know 4 pricing models: On demand, reserved, spot, and dedicated hosts

Spot will not charge you for the fuill hour if AWS terminates you, but it will charge you for the full hour if YOU terminate it

FIGHT DR MCPX

EBS types: General SSD, provisions IOPS SSD, Magnetic throughput optimized, magnetic cold hdd, and previous generation magnetic.

------

## First lab: launch an ec2 instance

start by clicking launch instance. You get a list of default AMIs

picked the Amazon Linux AMI, and a t2.micro

you can select a VPC in here, this will be important later. Using the default VPC for now.

subnets map to availability zones 1:1

shutdown behavior can set whether the VM gets destroyed or just stopped at an OS shutdown.

Can specifiy tenancy and monitoring.

Can add bootstrap scripts (will get into this later)

next section adds EBS volumes

can also add tags. Will get into this later (helps control costs). Ideally you want to tag everything

next is security groups _security groups are just virtual firewalls_

Can make a new security group here. Added ssh, http, and https. Restricted ssh to my IP address (actually didn't do this because work network). Left other sources open from everywhere.

Need to create a key pair (or select an existing one) as the final step in creating. Created one and downloaded it.

Launched the instance, it starts off in the pending state. We can check out what we set in the console.

sshed into the ec2 instance (had to chmod 400 the key for that)

`ssh ec2-user@<public-ip> -i pair`

had to use mobile to shell into the instance... so that's not ideal

`sudo su` to get root

`yum update` to apply updates

`yum install http` to get apache

`service httpd start`

put a file in `/var/www/html` and you can see it online!

Part 2 of the lecture starts here

looking in console at the various fields in description tab on tha console

because termination protection was turned on when we set this up, it won't let us terminate. You have to go in and manually turn off the termination protection

Looking at the status checks tab you can see the system status check and the instance status check

System status check checks if you can get network packets to the instance. Just checking the underlying hypervisor/networking.

Instance status check sees if you can actually get traffic to the OS. Instance status check failures can *probably* be fixed with a reboot. This is more of a sysops administrator question.

Monitoring tab has default, every 5 minutes checking. Lots of metrics from cloudwatch. Can turn on detail monitoring for an additional charge. Later on in the cloudwatch section we'll go into what all of these are.

Tags tab shows us what we set up earlier.

Terminating the instance in order to delete it

Looking at the reserved instances section just to see the menu. It'll price you right there and let you buy it.

looking at volume encryption, you can encrypt additional volumes, but not the root volume at launch. To encrypt the root volume you need to create a backup copy of the root volume and encrypt it while making the copy. This will be covered in a later lab

### Lab summary

Can create an ec2 instance

Can create ssh keys

Can ssh into the new instance

Learned about security groups

applied updates and installed apache on ec2 instance

Looks at ec2 details in console

### Know for exam

Termination protection is turned off by default

Default action when an instance is terminated is to delete the attached ebs instance. Can change this setting.

EBS root volumes cannot be encrypted, but you can use 3rd party tools, or you can create a copy of the AMI and encrypt it there.

Additional volumes *can* be encrypted.

## How to use putty for windows lecture

SKIPPED!

## Security groups lab

A security group is a virtual firewall

Every EC2 instance has at least one security group.

created a new ec2 instance with the security group from last time, installed apache and added a basic html file

checked security group rules in the console.

Delete http rule, and *immediately* we are not able to access the web page.

Know for exam: security rule changes apply immediately.

Added rule back and we were able to get back in.

Deleted outbound rule, and we were still abel to load the web page.

Security groups are stateful. All inbound rules will automatically be allowed out.

Later, with network access control lists, those are stateless so you need inbound and outbound.

Security group rules are stateful (not sure of the connection between state and inbound-outbound mapping but we'll probably come back to it later)

In security groups, you can't *deny* specific traffic, you can only allow it. It's a whitelist.

There's a default security group in each AZ, and it's set up by default to allow all traffic from instances in the same security group.

added RDP and MYSQL/Aurora to default group

in ec2 console, Actions -> networking -> change security groups to add another group

we added that default security group to our ec2. You can add multiple security groups to an instance.

VPC: stateless, Security group: stateful. It's been repeated a lot.

## Upgrading EBS volumes

created an ec2 instance with 3 different types of EBS volumes attached to it

it wasn't free so I cancelled creating it

EBS volumes have to be in the same AZ as the EC2 instance. Makes sense because latency.

You can tell which volumes are root devices because they have a "snapshot"

You can change the volume type and increase the size through the volumes console. (except on standard magnetic, can't change size on that)

to move a volume to another AZ (common exam question) you can create a snapshot in the volumes console (takes like 5 minutes)

Then from the snapshot you can create an image, volume, etc.

Create a new volume and you can change the type, size, and AZ. This is how you could move a volume to a new zone and thus a new instance.

Can also copy the snapshot to a new region. You'd have to do this to move an ec2 instance from one region to another.

Creating an image from a snapshot lets you boot an EC2 image from that instance. Snapshots are from backups, images are used for creating new ec2s

Image == AMI

Can create a snapshot then an image from a snapshot, or you can create an image directly from an ec2 instance if you'd prefer

By default, the root volume gets terminated when your EC2 is terminated, BUT the other attached volumes won't be terminated by default.

### Exam tips

Volume = Virtual hard disk

Root device = boot disk

Snapshots are saved in S3. They are a point in time copy of a Volume.

Snapshots are incremental, only the blocks that have changed since your last snapshot are moved to S3.

You shoudl stop an instance before taking a snapshot *probably*

you can create an AMI from Instances and Snapshots

You can change EBS volume sizes on the fly (this is relatively new)

Volumes will ALWAYS be in the same availability zone as the EC2 instance.

To move an EC2 to another AZ/Region, take a snapshot or an image, and then copy that to another region.

Security wise, snapshots of encrypted volumes are encrypted automatically. Same goes when restoring a volume from an ecrypted snapshot

You can share snapshots, but you can only do it if they are unencrypted

copying images/snapshots is SUPER important for the exam so know how to do it.