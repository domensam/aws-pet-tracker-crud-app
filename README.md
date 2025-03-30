# Hands-On Lab: Deploying a Web App on AWS (Beginner-Friendly)

## **1. Prerequisites**

Before starting, ensure you have:

- **AWS Account** with Admin access
- **AWS CLI** installed ([Download Here](https://aws.amazon.com/cli/))
- **Key Pair** for SSH access to EC2 instances

---

## **2. Step 1: Setup VPC and Networking**

We will create:
✅ A **VPC** with public and private subnets\
✅ **Internet Gateway (IGW)** for public subnets\
✅ **NAT Gateway** for private subnets\
✅ **Security Groups** for EC2, RDS, and ALB

### **Automate VPC Creation with CloudFormation**
Instead of manually creating a VPC, use this CloudFormation template to set up networking automatically:

1. Go to **AWS Console → CloudFormation → Create Stack**
2. Choose **Upload a template file**, then upload the following YAML file:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  LabVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref LabVPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref LabVPC
      CidrBlock: 10.0.2.0/24
      MapPublicIpOnLaunch: true

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref LabVPC
      CidrBlock: 10.0.3.0/24

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref LabVPC
      CidrBlock: 10.0.4.0/24

  InternetGateway:
    Type: AWS::EC2::InternetGateway

  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref LabVPC
      InternetGatewayId: !Ref InternetGateway
```

3. Click **Next**, give the stack a name (e.g., `LabVPC-Stack`), and deploy it.
4. Once complete, note down the **VPC ID** and **Subnet IDs**.

This will set up the VPC, public/private subnets, and internet gateway automatically.

---

## **3. Step 2: Deploy RDS Database (Each Student Creates Their Own RDS)**

Each student will create **their own RDS instance** to ensure full isolation.

1. Go to AWS Console → RDS → **Create Database**
2. Choose **PostgreSQL (or MySQL)**
3. Select **Free Tier** (or lowest-cost instance)
4. Set Database Name: `labdb_{your_name}`, Username: `admin`, Password: `LabPassword123`
5. Under Connectivity:
   - Choose **Lab-VPC**
   - Enable **Public Access** (for testing, otherwise use private access)
   - Select **RDS-SG**
6. Click **Create Database** and wait for the endpoint to be available
7. Copy the **RDS Endpoint** (this is unique for each student!)

---

## **4. Step 3: Launch Web App (EC2 Instances)**

We will launch **two EC2 instances** using a **pre-configured AMI** that already has the CRUD app installed.

### **Launch EC2 (via Console)**

1. Go to **EC2 → Launch Instance**
2. Use **AMI: Prebuilt Lab Image**
3. Choose instance type: `t2.micro`
4. Select **Lab-VPC**, place them in different public subnets
5. Choose **EC2-SG**
6. Click **Launch** and use an existing Key Pair
7. Repeat for the second EC2 instance

### **Configure the App to Connect to RDS**

1. SSH into one of the EC2 instances:
   ```bash
   ssh -i your-key.pem ec2-user@your-ec2-ip
   ```
2. Update the app’s environment variables with **your unique RDS Endpoint**:
   ```bash
   echo "DB_ENDPOINT='YOUR_UNIQUE_RDS_ENDPOINT'" >> /etc/environment
   source /etc/environment
   ```
3. Restart the app (if needed):
   ```bash
   sudo systemctl restart webapp
   ```

### **Verify EC2 Connectivity to Your RDS**

SSH into one EC2 instance and run:

```bash
psql -h YOUR_UNIQUE_RDS_ENDPOINT -U admin -d labdb_{your_name}
```

If connected successfully, the app is now communicating with **your own database**!

---

## **5. Step 4: Setup ALB (Application Load Balancer)**

1. Go to **EC2 → Load Balancers → Create Load Balancer**
2. Choose **Application Load Balancer**
3. Name: `Lab-ALB`
4. Select **Lab-VPC**, public subnets
5. Create a **Target Group** → Attach both EC2s
6. Set Listener on Port **80 → Forward to Target Group**
7. Click **Create Load Balancer**

### **Test ALB**

Find the **ALB DNS Name** from the AWS Console and open it in a browser. You should see the web app running!

---

## **6. Step 5: Cleanup (When Done)**

To avoid charges, **delete**:
✅ Your **RDS instance**
✅ EC2 instances
✅ ALB
✅ *VPC (if not reused)*

---

## **Final Notes**

- Each student **has their own RDS instance** with a **unique endpoint**.
- The **AMI includes the app**, so no manual deployment is needed.
- Keep security best practices in mind, such as **disabling public RDS access** after testing.

🎉 **Congratulations! You’ve deployed a web app on AWS!** 🎉
