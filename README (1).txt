-----------------------------------------WEEK-1--------------------------------------------
🔵 PART 1: Create Linux (Ubuntu) EC2 Instance & Connect Using SSH
🧾 Prerequisites

Make sure you have:

AWS account

PuTTY installed (Windows)

PuTTYgen installed

Key pair (.pem file)

Internet connection

🔹 STEP 1: Create Ubuntu EC2 Instance

Login to AWS Management Console

Go to Services → EC2

Click Launch Instance

Configure instance

Name: Ubuntu-Instance

Select AMI → Choose Ubuntu Server

Instance type → Select t2.micro (Free tier)

Key pair → Click Create new key pair

Name: mykey

Type: RSA

Format: .pem

Click Download

Network settings:

Allow SSH (port 22)

Source: My IP

Click Launch Instance

Wait until instance status = Running

🔹 STEP 2: Check Security Group (Important)

Go to EC2 → Instances

Select instance

Scroll to Security

Click security group

Check inbound rules:

Type	Port	Source
SSH	22	My IP

If not present → Add rule → Save

🔹 STEP 3: Connect Using PuTTY (.pem → .ppk)

PuTTY cannot use .pem directly
So convert to .ppk first.

Convert .pem to .ppk

Open PuTTYgen

Click Load

Select .pem file

Click Open

Click Save private key

Save as .ppk

✔ Conversion complete

🔹 STEP 4: Connect Using PuTTY

Open PuTTY

In Host Name:

ubuntu@<Public-IP>

Example:

ubuntu@3.110.xxx.xxx

Left menu:

Connection → SSH → Auth → Credentials

Click Browse

Select .ppk file

Click Open

Click Yes (first time alert)

🎉 Connected to Ubuntu EC2

🔹 STEP 5: Verify Connection

Type command:

whoami

Output:

ubuntu

✔ SSH connection successful

🔵 PART 2: Connect EC2 Using AWS CloudShell
Steps

Open AWS Console

Click CloudShell icon (top right terminal)

Go to EC2 → Instances

Select instance → Click Connect

Choose SSH client tab

Copy SSH command:

ssh -i mykey.pem ubuntu@public-ip

In CloudShell → upload key:

Click Actions (+)

Upload .pem file

Give permission:

chmod 400 mykey.pem

Run SSH command again:

ssh -i mykey.pem ubuntu@public-ip

🎉 Connected successfully

🔵 PART 3: Create Windows EC2 & Connect Using RDP
🔹 STEP 1: Launch Windows Instance

Go to AWS Console → EC2

Click Launch Instance

Name: Windows-Server

AMI → Select Windows Server 2019/2022

Instance type → t2.micro

Create key pair → Download .pem

Security group:

Allow RDP (3389)

Source: My IP

Click Launch

Wait until running

🔹 STEP 2: Get Public IP

Go to EC2 → Instances

Select instance

Copy Public IP / DNS

🔹 STEP 3: Get Windows Password

Select instance

Click Connect

Choose RDP Client

Click Get Password

Upload .pem key

Click Decrypt Password

Copy password

🔹 STEP 4: Connect Using Remote Desktop

Press Windows + R

Type:

mstsc

Enter Public IP

Click Connect

Login:

Username: Administrator

Password: (decrypted password)

Click OK

🎉 Windows EC2 desktop opens
-------------------------------------------------------------------WEEK-2--------------------------------------------------
1. Create and Attach an EBS Volume to an EC2 Instance
Step 1: Create a New EBS Volume

Login to AWS Management Console.

Go to EC2 Dashboard.

In the left panel → click Volumes (under Elastic Block Store).

Click Create Volume.

Enter details:

Volume type: gp3 (recommended) or gp2

Size: (example 10 GB, 20 GB etc.)

Availability Zone: must be same as EC2 instance

Click Create Volume.

Your EBS volume is now created.

Step 2: Attach EBS Volume to EC2 Instance

In Volumes, select the newly created volume.

Click Actions → Attach Volume.

Choose the EC2 instance from dropdown.

Enter device name (example: /dev/xvdf).

Click Attach Volume.

Volume is now attached to EC2.

Step 3: Verify in EC2 Instance

Connect to EC2 using SSH:

ssh ec2-user@public-ip

Check disks:

lsblk

You will see new volume like:

xvdf

Format and mount (first time only):

sudo mkfs -t ext4 /dev/xvdf
sudo mkdir /data
sudo mount /dev/xvdf /data

Now EBS volume is ready to use.

2. Scale EC2 Instance (Increase CPU & RAM by Changing Instance Type)

To increase CPU and RAM → change instance type.

Step 1: Go to EC2 Dashboard

Open AWS Console

Go to EC2 → Instances

Select your EC2 instance

Step 2: Stop the Instance

You cannot change instance type while running.

Select instance

Click Instance State → Stop

Wait until instance stops

Step 3: Change Instance Type

Select the stopped instance

Click Actions → Instance Settings → Change Instance Type

Choose new instance type:

Example: t2.micro → t2.medium

More CPU + more RAM

Click Apply/Change

Step 4: Start the Instance

Select instance

Click Instance State → Start

Instance will start with:

Increased CPU

Increased RAM

Better performance

2.Attach and permanently mount an EBS volume to a Linux EC2 instance to ensure data persistence across reboots.
🔵 STEP 1: Create EBS Volume (20GB)

Go to AWS Console

EC2 → Volumes

Click Create Volume

Select:

Type: gp3

Size: 20 GB

Availability Zone: same as instance (important)

Click Create Volume

Done.

🔵 STEP 2: Attach volume to EC2

Select new volume

Click Actions → Attach volume

Choose your instance

Device name:

/dev/xvdf

Click Attach

Done.

🔵 STEP 3: Connect to EC2

Open PowerShell/terminal:

ssh -i volkey.pem ec2-user@YOUR-PUBLIC-IP

Example:

ssh -i volkey.pem ec2-user@54.234.134.132
🔵 STEP 4: Check new disk

Inside EC2 type:

lsblk

You will see:

nvme0n1  (main disk)
nvme1n1  (new EBS disk)

👉 nvme1n1 = new volume

🔵 STEP 5: Create partition

Run:

sudo fdisk /dev/nvme1n1

Press one by one:

n → Enter
p → Enter
Enter
Enter
w → Enter

Then run:

sudo partprobe
🔵 STEP 6: Format disk
sudo mkfs.xfs /dev/nvme1n1p1
🔵 STEP 7: Create folder
sudo mkdir /data
🔵 STEP 8: Mount volume
sudo mount /dev/nvme1n1p1 /data

Check:

df -h

You should see /data mounted.

🔵 STEP 9: Permanent mount (VERY IMPORTANT FOR EXAM)

Get UUID:

sudo blkid

Copy UUID of nvme1n1p1.

Open fstab:

sudo nano /etc/fstab

Add last line:

UUID=copy-paste-uuid   /data   xfs   defaults,nofail   0   0

Save:

Ctrl + X
Y
Enter

Apply:

sudo mount -a
🔵 STEP 10: Check persistence

Create file:

cd /data
touch test1
ls

Reboot:

sudo reboot

Login again:

df -h

If /data still mounted → SUCCESS 🎉

3. Create a snapshot of the attached EBS volume and use it to create and attach a new volume to an EC2 instance in another AWS region.
🔵 Step 1: Create Snapshot of EBS Volume

Login to AWS Management Console

Go to EC2 Dashboard

Click Volumes (left side)

Select the EBS volume (attached one)

Click Actions → Create Snapshot

Enter:

Name: mysnapshot

Description: EBS backup

Click Create Snapshot

Go to Snapshots section and wait until status =

completed

Snapshot created successfully.

🔵 Step 2: Copy Snapshot to Another Region

Go to Snapshots

Select created snapshot

Click Actions → Copy snapshot

Choose destination region
Example:

Source: N. Virginia (us-east-1)
Destination: Ohio (us-east-2)

Give name

Click Copy Snapshot

Wait until snapshot status becomes completed in destination region.

🔵 Step 3: Switch Region

Top right corner in AWS console → change region
to destination region (example Ohio).

Now you are in new region.

🔵 Step 4: Create Volume from Snapshot

Go to EC2 → Snapshots

Select copied snapshot

Click Actions → Create Volume

Choose:

Volume type: gp3

Size: same or bigger

Availability Zone: same as target EC2

Click Create Volume

New EBS volume created in new region.

🔵 Step 5: Attach Volume to EC2 (New Region)

Go to EC2 → Volumes

Select new volume

Click Actions → Attach Volume

Choose EC2 instance

Device name:

/dev/xvdf

Click Attach

🔵 Step 6: Connect to EC2 and Mount

Login:

ssh -i key.pem ec2-user@public-ip

Check disk:

lsblk

You will see:

nvme1n1

Mount:

sudo mkdir /data
sudo mount /dev/nvme1n1p1 /data

Check data:

cd /data
ls

Old files will be present.
---------------------------------------------WEEK-3----------------------------
🔹 STEP 1: Launch EC2 Instances (Amazon Linux)
Instance 1: EFS-1

Login to AWS Management Console

Open EC2 Dashboard

Click Launch Instance

Configure:

Name: EFS-1

AMI: Amazon Linux

Instance Type: t2.micro

Key Pair: EFS

VPC: Default

Subnet: Subnet-1a

Security Group (Important):

Create new SG

Allow NFS

Type: NFS

Port: 2049

Source: Anywhere (0.0.0.0/0)

Click Launch

Instance 2: EFS-2

Repeat the same steps

Change:

Name: EFS-2

Subnet: Subnet-1b

Use same security group

🔹 STEP 2: Create an EFS File System

Open EFS service

Click Create file system

Configure:

Name: Optional

VPC: Default

Enable Region

Click Next

Network Settings:

Remove default security groups

Select EC2 NFS security group

Click Next → Review

Click Create File System

🔹 STEP 3: Access EC2 Instances via PowerShell
Open Two PowerShell Windows
For EACH Instance (EFS-1 and EFS-2)

SSH into EC2

ssh -i path-to-keypair.pem ec2-user@<public-ip>

Switch to root user

sudo su

Create directory

mkdir efs

Install EFS utilities

yum install -y amazon-efs-utils

Verify installation

efs-utils --version

List files

ls
🔹 STEP 4: Attach (Mount) EFS to EC2

Go to EFS Dashboard

Click your EFS File System

Click Attach

Choose Mount via DNS

Copy the command (example):

mount -t efs fs-xxxx:/ efs

Run this command on:

EFS-1

EFS-2

🔹 STEP 5: Verify and Test EFS Sharing
On EFS-1
cd efs
touch testfile.txt
ls
On EFS-2
cd efs
ls

✅ You will see testfile.txt automatically
➡️ This confirms shared storage is working
-----------------------------------WEEK-4-------------------------------------------
////CHOOSE LABROLE IN IAM ROLES
1.Creating an S3 bucket, upload an object, and enable public access 
1️⃣ Create an S3 Bucket (with Public Access Enabled)
Step 1: Open S3 Service
•	Login to AWS Management Console
•	Go to Services → S3
•	Click Create bucket
________________________________________
Step 2: Bucket Configuration
•	Bucket name: my-public-bucket-123
•	Region: Choose nearest region
________________________________________
Step 3: Disable Block Public Access (IMPORTANT)
Under Block Public Access settings for this bucket:
❌ Uncheck Block all public access
You must uncheck ALL options:
✔ Check I acknowledge that this bucket will become public
👉 Click Create bucket
________________________________________
2️⃣ Enable Public Access at Bucket Level (ACL)
Step 4: Open the Bucket
•	Click on the bucket name
•	Go to Permissions tab
________________________________________
Step 5: Edit Bucket ACL
•	Scroll to Access Control List (ACL)
•	Click Edit
Under Public access:
•	Everyone (public access) → ✔ Read access
✔ Save changes
📌 This allows public access at bucket level
________________________________________
3️⃣ Upload Object to the Bucket
Step 6: Upload File
•	Go to Objects tab
•	Click Upload
•	Click Add files
•	Select file (e.g., image.jpg)
•	Click Upload
________________________________________
4️⃣ Enable Public Access for Object (ACL)
Step 7: Object-Level ACL Permission
•	Click on uploaded object
•	Go to Permissions tab
•	Scroll to Access Control List (ACL)
•	Click Edit
Under Public access:
•	Everyone (public access) → ✔ Read access
✔ Save changes
________________________________________
5️⃣ Verify Public Access
Step 8: Test Object URL
•	Copy Object URL
•	Open in browser (incognito)
✔ Object should be accessible without login
________________________________________
2. S3 Versioning & Cross-Region Replication 
________________________________________
1️⃣ Enabling Versioning in S3 (with Output Scenario)
🔸 Objective
To enable version control on an S3 bucket and observe object versions.
________________________________________
🔸 Steps to Enable Versioning
1.	Open AWS Management Console
2.	Navigate to S3
3.	Select the bucket
4.	Go to Properties tab
5.	Scroll to Bucket Versioning
6.	Click Edit
7.	Select Enable
8.	Click Save changes
✔ Versioning is now enabled
________________________________________
🔸 Output Scenario (What You Observe)
Step A: Upload Object
•	Upload file: report.pdf
📌 Output:
•	Version ID is assigned automatically
________________________________________
Step B: Modify and Re-upload Same Object
•	Upload report.pdf again (same name)
📌 Output:
•	Old version is preserved
•	New version becomes current
________________________________________
Step C: Delete Object
•	Click Delete on report.pdf
📌 Output:
•	Object appears deleted
•	A Delete Marker is created
•	Older versions still exist and can be restored
________________________________________

2️⃣ Cross-Region Replication (CRR) 
🔸 Objective
To replicate objects from one bucket to another in a different region using AWS-managed permissions.
________________________________________
🔸 Prerequisites (Very Important)
✔ Versioning must be enabled on both buckets
✔ Source and destination buckets must be in different regions
✔ AWS Academy automatically handles replication permissions
________________________________________
Step 1️⃣ Create Destination Bucket
1.	Go to S3 → Create bucket
2.	Bucket name: dest-crr-bucket
3.	Select different region
4.	Complete bucket creation
5.	Enable Versioning (same steps as above)
________________________________________
Step 2️⃣ Create Replication Rule (Without IAM Selection)
1.	Open Source bucket
2.	Go to Management tab
3.	Click Replication rules
4.	Click Create replication rule
________________________________________
Step 3️⃣ Configure Replication Rule
•	Rule name: academy-crr-rule
•	Status: Enabled
•	Scope: Apply to all objects
________________________________________
Step 4️⃣ Choose Destination Bucket
•	Select Another bucket
•	Choose Destination bucket
•	Confirm different region
📌 AWS Academy Note:
You will see an option like:
"AWS will create and manage the required permissions automatically"
✔ Select Use AWS-managed role / default role
________________________________________
Step 5️⃣ Replication Options
•	Replicate:
o	✔ New objects
o	❌ Existing objects (optional, depends on Academy permissions)
•	Encryption: Leave default
Click Save
✔ Replication rule is now active
________________________________________
3️⃣ CRR Output Scenario (What You Observe)
🔸 Step A: Upload New Object to Source Bucket
•	Upload file: image.png
📌 Output:
•	Object is uploaded normally
________________________________________
🔸 Step B: Verify Destination Bucket
•	Open destination bucket
•	Object image.png appears automatically
•	Version ID is present
✔ Replication successful
________________________________________

3.	Static Website Hosting in S3
(Using Bucket-Level & Object-Level Permissions – ACL Method)
________________________________________
🔹 Objective
To host a static website in Amazon S3 using index.html and error.html, and allow public access using ACL-based bucket and object permissions.
________________________________________
🔹 Prerequisites
•	AWS / AWS Academy account
•	Two files:
o	index.html
o	error.html
________________________________________
1️⃣ Create an S3 Bucket
1.	Login to AWS Management Console
2.	Go to Services → S3
3.	Click Create bucket
4.	Enter:
o	Bucket name: my-static-site-bucket-123
o	Region: Nearest region
5.	Under Block Public Access:
o	❌ Uncheck Block all public access
o	✔ Acknowledge the warning
6.	Click Create bucket
✔ Bucket created successfully
________________________________________
2️⃣ Upload Website Files
1.	Open the bucket
2.	Go to Objects tab
3.	Click Upload
4.	Click Add files
5.	Select:
o	index.html
o	error.html
6.	Click Upload
✔ Files uploaded
________________________________________
3️⃣ Enable Static Website Hosting
1.	Go to Properties tab
2.	Scroll to Static website hosting
3.	Click Edit
4.	Select Enable
5.	Choose Host a static website
6.	Enter:
o	Index document: index.html
o	Error document: error.html
7.	Click Save changes
✔ Static website hosting enabled
________________________________________
4️⃣ Enable Bucket-Level Public Access (ACL)
🔹 Purpose
Allows public users to access objects inside the bucket.
________________________________________
1.	Go to Permissions tab
2.	Scroll to Access Control List (ACL)
3.	Click Edit
4.	Under Public access:
o	Everyone (public access) → ✔ Read
5.	Click Save changes
✔ Bucket-level public read access enabled
________________________________________
5️⃣ Enable Object-Level Public Access (ACL)
Steps (For Each File)
1.	Go to Objects tab
2.	Click on index.html
3.	Go to Permissions
4.	Scroll to Access Control List (ACL)
5.	Click Edit
6.	Under Public access:
o	Everyone (public access) → ✔ Read
7.	Save changes
🔁 Repeat the same steps for error.html
✔ Object-level public access enabled
________________________________________
6️⃣ Access the Static Website
1.	Go to Properties
2.	Scroll to Static website hosting
3.	Copy the Bucket website endpoint
4.	http://my-static-site-bucket-123.s3-website-region.amazonaws.com
5.	Paste into browser
✔ index.html page loads successfully
________________________________________
7️⃣ Error Page Output Scenario
🔹 Test Error Page
•	Open an invalid URL:
•	http://bucket-name.s3-website-region.amazonaws.com/invalid.html
📌 Output:
•	error.html page is displayed
✔ Error page working correctly
----------------------------------------------WEEK-5------------------------------------
🔹 STEP 1: Create a Custom VPC

Login to AWS Management Console

Search VPC → Open VPC Dashboard

Click Create VPC

Select VPC only

Enter:

Name: Custom-VPC

IPv4 CIDR block: 10.0.0.0/16

Click Create VPC

✔ Now your VPC is created.

🔹 STEP 2: Create Public & Private Subnets

We will create 2 subnets in same AZ.

✅ Create Public Subnet

Go to Subnets

Click Create subnet

Select VPC → Custom-VPC

Availability Zone → e.g., ap-south-1a

Subnet name → Public-Subnet

IPv4 CIDR → 10.0.1.0/24

Click Create subnet

✅ Create Private Subnet

Click Create subnet

Select same VPC

Same Availability Zone

Subnet name → Private-Subnet

IPv4 CIDR → 10.0.2.0/24

Click Create subnet

✔ Now you have 2 subnets.

🔹 STEP 3: Create and Attach Internet Gateway (IGW)

Go to Internet Gateways

Click Create Internet Gateway

Name → Custom-IGW

Click Create

Click Attach to VPC

Select Custom-VPC

Click Attach

✔ Internet Gateway attached.

🔹 STEP 4: Create Route Tables
✅ Public Route Table

Go to Route Tables

Click Create route table

Name → Public-RT

Select VPC → Custom-VPC

Click Create

➤ Add Internet Route

Select Public-RT

Go to Routes tab

Click Edit routes

Add route:

Destination → 0.0.0.0/0

Target → Select Custom-IGW

Save

➤ Associate with Public Subnet

Go to Subnet Associations

Click Edit associations

Select Public-Subnet

Save

✔ Public subnet now has internet access.

✅ Private Route Table

Click Create route table

Name → Private-RT

Select VPC

Click Create

⚠ Do NOT add IGW route.

➤ Associate Private Subnet

Go to Subnet Associations

Click Edit

Select Private-Subnet

Save

✔ Private subnet isolated.

🔹 STEP 5: Enable Auto-Assign Public IP (Public Subnet Only)

Go to Subnets

Select Public-Subnet

Click Actions → Edit subnet settings

Enable:
✅ Auto-assign public IPv4 address

Save

🔹 STEP 6: Create Security Groups

Go to Security Groups → Create security group

Example Security Group (Public EC2)

Inbound Rules:

SSH (22) → Your IP

HTTP (80) → 0.0.0.0/0

HTTPS (443) → 0.0.0.0/0

Click Create

🔹 STEP 7: Launch EC2 Instances
✅ Launch Public EC2

Go to EC2 → Launch instance

Name → Public-EC2

Choose Amazon Linux

Choose Instance type (t2.micro)

Select:

VPC → Custom-VPC

Subnet → Public-Subnet

Auto assign Public IP → Enabled

Attach Public Security Group

Launch with Key Pair

✔ This instance gets Public IP.

✅ Launch Private EC2

Launch new instance

Name → Private-EC2

VPC → Custom-VPC

Subnet → Private-Subnet

Auto assign Public IP → Disabled

Attach Security Group

Launch

✔ This instance will NOT get Public IP.

🔹 STEP 8: Setup NAT Gateway (For Private Internet Access)

Without NAT, Private EC2 cannot access internet.

✅ Create NAT Gateway

Go to NAT Gateways

Click Create NAT Gateway

Subnet → Public-Subnet

Allocate Elastic IP

Click Create

✅ Add NAT Route to Private Route Table

Go to Route Tables

Select Private-RT

Edit routes

Add:

Destination → 0.0.0.0/0

Target → NAT Gateway

Save

✔ Now private EC2 can access internet.

🔹 STEP 9: Testing
✅ Test Public EC2

SSH into Public EC2:

ssh -i key.pem ec2-user@Public-IP

Test internet:

ping google.com

✔ Should work.

✅ Test Private EC2

You cannot SSH directly.

Instead:

SSH into Public EC2

From Public EC2:

ssh ec2-user@Private-IP

Test internet from Private EC2:

ping google.com

✔ Should work via NAT

