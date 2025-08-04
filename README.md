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

### ✅ Step 1 – Creating a VPC, subnets, route tables and an Internet Gateway

Created a VPC named my_VPC with:
- IPv4 CIDR Block: `192.168.0.0/26`  
- No IPv6 CIDR Block
- 
I created an internet geteway with the Name: `MyInternetGateway` and attach it to `myVPC`

I created two subnets:
- **Public Subnet**  
  - Name: `Public Subnet`  
  - AZ: `us-east-1a`  
  - CIDR Block: `192.168.0.1/27`
  - Auto-assignn public IPv4 adresses is enabled
- **Private Subnet**  
  - Name: `Private Subnet`  
  - AZ: `us-east-1b`  
  - CIDR Block: `192.168.0.32/27`

I created Route Tables and Associate them with the subnets 
- **Public Route Table**  
  - Name: `PublicRouteTable`  
  - Associated with: `Public Subnet`  
  - Route: `0.0.0.0/0 → MyInternetGateway`

- **Private Route Table**  
  - Name: `PrivateRouteTable`  
  - Associated with: `Private Subnet`

🗺️ **VPC Resource Map** 
<img width="1455" height="352" alt="16" src="https://github.com/user-attachments/assets/da58c593-ee6b-4b94-ae0e-c018d5420ae3" />


### ✅ Step 2 – Creating Security Groups

I created two security groupes:

- **Bastion Security Group (Bastion-SG)**  
  - Inbound Rules:
    - SSH: `0.0.0.0/0`
    - HTTP: `0.0.0.0/0`
    - HTTPS: `0.0.0.0/0`

- **Endpoint Security Group (Endpoint-SG)**  
  - Inbound Rule:
    - SSH: `Source = Bastion-SG`
      

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
### ✅ Step 4 – SSH Access via Bastion Host

```bash
# 1. SSH into Bastion Host
ssh -i myKey.pem ec2-user@<Bastion_Public_IP>

# 2. On the Bastion Host, create and secure your key file
vi myKey.pem  # Paste private key content
chmod 400 myKey.pem

# 3. SSH into the Endpoint Instance from the Bastion Host
ssh -i myKey.pem ec2-user@<Endpoint_Private_IP>
```

✅ Successfully accessing the private instance via the bastion confirms that the security groups and routing are correctly configured.

---

### 🧪 Step 10 – Testing Connectivity

Connected to `public_instance` using EC2 Instance Connect.

From the public instance, tested:

- ✅ Connectivity to the private instance  
  ```bash
  ping 10.0.2.128 -c 5
  ```

- ✅ Connectivity to the internet  
  ```bash
  ping google.com -c 5
  ```

📸 _See test images below._

✅ These results confirm that the VPC configuration, security group rules, and routing tables are working correctly.

---

### ✍️ Author

Made with 💻 by **Nidhal Labri**  
🔗 [LinkedIn](https://www.linkedin.com/in/YOUR-USERNAME)
