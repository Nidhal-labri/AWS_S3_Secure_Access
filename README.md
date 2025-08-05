# 🔐 AWS Project: Creating Secure Access to S3 via VPC Endpoint and Bastion Host

This project demonstrates how I securely accessed Amazon S3 from an EC2 instance in a private subnet using a **VPC Gateway Endpoint** and a **Bastion Host (Jump Box)** for SSH access.

---
## 🌐 Why This Project?

This project was designed to explore how to securely access AWS services like S3 without exposing private instances to the internet. By configuring a bastion host in a public subnet and using a VPC endpoint for S3 access, I learned how to establish **secure, isolated environments** while ensuring essential access to cloud resources. This setup is commonly used in production environments to enhance security, reduce attack surfaces, and avoid unnecessary internet exposure.

---
## 🗺️ Architecture Diagram

<img width="3829" height="2100" alt="image" src="https://github.com/user-attachments/assets/32ad6a9d-09b3-4b43-a215-32ed08a71509" />

---

## 🧱 Key AWS Services Used

- **VPC** – Isolated networking environment for deploying resources.
- **Subnets** – Public and private subnets to segment resources.
- **Internet Gateway** – Enables internet access for public resources.
- **Route Tables** – Control routing paths inside the VPC.
- **Security Groups** – Act as stateful firewalls for EC2 instances.
- **EC2 Instances** – Bastion Host (public) and Endpoint Instance (private).
- **VPC Endpoint (Gateway)** – Private connection to S3 service.
- **AWS CLI** – Used to test and access S3 buckets from the private instance.

---
## 🛠️ Deployment Steps

### ✅ Step 1 – Creating a VPC, Subnets, Route Tables, Internet Gateway and VPC endpoint.

I created a VPC named `my_VPC` with the following settings:
- **IPv4 CIDR Block:** `192.168.0.0/26`
- **IPv6:** Not assigned

Then, I created an Internet Gateway:
- **Name:** `MyInternetGateway`
- **Attached to:** `my_VPC`

Next, I created two subnets:
- **Public Subnet**
  - **Name:** `Public Subnet`
  - **Availability Zone:** `us-east-1a`
  - **CIDR Block:** `192.168.0.1/27`
  - **Auto-assign Public IPv4:** Enabled
- **Private Subnet**
  - **Name:** `Private Subnet`
  - **Availability Zone:** `us-east-1b`
  - **CIDR Block:** `192.168.0.32/27`

I then created and associated route tables:
- **Public Route Table**
  - **Name:** `PublicRouteTable`
  - **Associated with:** `Public Subnet`
  - **Route:** `0.0.0.0/0 → MyInternetGateway`
- **Private Route Table**
  - **Name:** `PrivateRouteTable`
  - **Associated with:** `Private Subnet`

🗺️ **VPC Resource Map**  
<img width="1455" height="352" alt="16" src="https://github.com/user-attachments/assets/da58c593-ee6b-4b94-ae0e-c018d5420ae3" />

I also created a **Gateway VPC Endpoint** for the service `com.amazonaws.us-east-1.s3` within `my_VPC`, and attached it to the `PrivateRouteTable`.


### ✅ Step 2 – Creating Security Groups

I created two security groupes:

- **Bastion Security Group (Bastion-SG)**  
  - Inbound Rules:

| Type    | Protocol | Port Range | Source     |
|---------|----------|------------|------------|
| SSH     | TCP      | 22         | 0.0.0.0/0  |
| HTTP    | TCP      | 80         | 0.0.0.0/0  |
| HTTPS   | TCP      | 443        | 0.0.0.0/0  |

- **Endpoint Security Group (Endpoint-SG)**  
  - Inbound Rule:

| Type | Protocol | Port Range | Source       |
|------|----------|------------|--------------|
| SSH  | TCP      | 22         | `Bastion-SG` |
      

### ✅ Step 3 – Launching the Bastion Host and Endpoint Instance

Launched **two t2.micro instances** using **Amazon Linux 2 AMI** inside `my_VPC`:

- `Bastion-Host`:
  - Subnet: `public_subnet`
  - Auto-assign Public IP: Enabled
  - Security Group: Bastion-SG
  - Key pair: `myKey.pem`

- `Endpoint-Instance`:
  - Subnet: `private_subnet`
  - Auto-assign Public IP: Disabled
  - Security Group: `Endpoint-SG`
  - Key pair: `myKey.pem`

🔐 **Note:** I also created and associated a key pair named `myKey.pem` for SSH access.
<img width="1914" height="318" alt="17" src="https://github.com/user-attachments/assets/65d85cc4-41ab-4a08-8611-495d2cced6e2" />


---
### ✅ Step 4 – SSH Access via Bastion Host and Access S3 buckets 

First, I connected to the Bastion Host via SSH:
<img width="1919" height="828" alt="19" src="https://github.com/user-attachments/assets/d409b815-f4e8-4363-8bc6-d8c973cda56c" />

Then, I created and secured my key file using: 
```bash
vi myKey.pem        # Paste your private key content inside the file, then save and exit
chmod 400 myKey.pem
```

Next, I SSH'd into the Endpoint Instance from the Bastion Host
✅ Successfully accessing the private instance via the Bastion Host confirms that the Security Groups and routing are correctly configured.

<img width="1916" height="820" alt="image" src="https://github.com/user-attachments/assets/275d1986-e83f-4bc5-ab43-a960ec580900" />

I then ran aws configure to set up my AWS credentials:

<img width="721" height="107" alt="20" src="https://github.com/user-attachments/assets/1172a199-14b9-43a6-9969-328929d55550" />

✅ **After configuration, I was able to securely access, list, and interact with S3 buckets from the private instance**

<img width="662" height="52" alt="21" src="https://github.com/user-attachments/assets/e9b981e8-3299-49c6-9821-b7c4a16bd038" />

---

### ✍️ Author

Made with 💻 by **Nidhal Labri**  
🔗 [LinkedIn](https://www.linkedin.com/in/YOUR-USERNAME)
