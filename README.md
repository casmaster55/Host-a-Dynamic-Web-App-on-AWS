# **Hosting a Dynamic Web App on AWS – DevOps Project**  

## **Overview**  
This project demonstrates how to **host a dynamic web application on AWS** using a **highly available and fault-tolerant** architecture. The application is deployed on **Amazon EC2 instances**, utilizing **Auto Scaling, Load Balancer, RDS, and S3** to ensure scalability, security, and efficiency.  

## **Project Architecture**  
This setup follows **best DevOps practices** to ensure **high availability, security, and automation**.  

### **1. Networking & Security**  
- **VPC (Virtual Private Cloud)**: Configured with **public and private subnets** across **two availability zones** for redundancy.  
- **Internet Gateway**: Enables public internet access for instances in the **public subnets**.  
- **NAT Gateway**: Allows instances in **private subnets** to access the internet securely.  
- **Security Groups**: Acts as a firewall to control inbound and outbound traffic.  

### **2. Compute & Scalability**  
- **EC2 Instances**: Hosts the web application.  
- **Auto Scaling Group (ASG)**: Dynamically scales EC2 instances based on traffic load.  
- **Application Load Balancer (ALB)**: Distributes traffic across multiple EC2 instances for high availability.  

### **3. Domain & DNS Management**  
- **Route 53**: Used to register the domain name and create **DNS records** to route traffic.  

### **4. Database & Storage**  
- **Private Subnets for Web & Database Servers**:  
  - Web servers and database servers are placed in **private subnets** for **security**.  
- **Amazon RDS (Relational Database Service)**:  
  - A **MySQL database** is used for the application.  
  - **Flyway** is used for database schema migrations.  
- **S3 Storage**: Stores website files and application backups.  

### **5. Deployment Process**  
- The web application is **hosted on EC2 instances**.  
- The application code is stored in an **S3 bucket** and synced to EC2.  
- Once configured, an **Amazon Machine Image (AMI)** is created.  
- **GitHub** is used for version control.  

---

## **Deployment Steps**  

### **1. Set Up AWS Environment**  
1. **Create a VPC** with **public and private subnets** in two **availability zones**.  
2. Configure an **Internet Gateway** and attach it to the VPC.  
3. Create **Security Groups** for EC2, Load Balancer, and RDS.  
4. Deploy a **NAT Gateway** in the **public subnet** to allow private instances to access the internet.  

### **2. Deploy the Web Application on EC2**  
The following **Bash script** installs all necessary dependencies, syncs the web files from S3, and configures the Apache web server on an **Amazon Linux EC2 instance**:  

```bash
#!/bin/bash

# Update all packages
sudo yum update -y

# Install Apache Web Server and enable it
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP and required extensions
sudo dnf install -y \
php \
php-pdo \
php-openssl \
php-mbstring \
php-exif \
php-fileinfo \
php-xml \
php-ctype \
php-json \
php-tokenizer \
php-curl \
php-cli \
php-fpm \
php-mysqlnd \
php-bcmath \
php-gd \
php-cgi \
php-gettext \
php-intl \
php-zip

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Enable 'mod_rewrite' in Apache for .htaccess support
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Define the S3 bucket containing web files
S3_BUCKET_NAME=aosnote-shopwise-web-files

# Sync web files from S3 to EC2
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# Change directory to web root
cd /var/www/html

# Extract application code
sudo unzip shopwise.zip

# Move files to the web root
sudo cp -R shopwise/. /var/www/html/

# Clean up extracted files
sudo rm -rf shopwise shopwise.zip

# Set proper permissions
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# Open .env file to configure database credentials
sudo vi .env

# Restart Apache server
sudo service httpd restart
```

---

### **3. Configure Auto Scaling & Load Balancing**  
1. Set up an **Application Load Balancer (ALB)** to distribute traffic.  
2. Configure an **Auto Scaling Group (ASG)** to dynamically create and terminate EC2 instances.  

### **4. Configure Domain & DNS**  
1. Register a domain using **Amazon Route 53**.  
2. Create a **DNS record set** pointing to the Load Balancer.  

### **5. Database Migration with Flyway**  
The following **Bash script** is used to download and apply database migrations using **Flyway**:  

```bash
#!/bin/bash

S3_URI=s3://aosnote-sql-files/V1__shopwise.sql
RDS_ENDPOINT=dev-rds-db.cu2idoemakwo.us-east-1.rds.amazonaws.com
RDS_DB_NAME=applicationdb
RDS_DB_USERNAME=azeezs
RDS_DB_PASSWORD=azeezs123

# Update all packages
sudo yum update -y

# Download and extract Flyway
sudo wget -qO- https://download.red-gate.com/maven/release/com/redgate/flyway/flyway-commandline/10.9.1/flyway-commandline-10.9.1-linux-x64.tar.gz | tar -xvz 

# Create a symbolic link for Flyway
sudo ln -s $(pwd)/flyway-10.9.1/flyway /usr/local/bin

# Create SQL directory
sudo mkdir sql

# Download migration SQL script from S3
sudo aws s3 cp "$S3_URI" sql/

# Run Flyway migration
flyway -url=jdbc:mysql://"$RDS_ENDPOINT":3306/"$RDS_DB_NAME" \
  -user="$RDS_DB_USERNAME" \
  -password="$RDS_DB_PASSWORD" \
  -locations=filesystem:sql \
  migrate
```

---

## **Key Benefits of This Architecture**  
✅ **High Availability** – Multi-AZ deployment with **Auto Scaling**.  
✅ **Fault Tolerance** – Automatic instance replacement if failures occur.  
✅ **Security Best Practices** – Private subnets, **security groups**, and **NAT Gateway** protect internal resources.  
✅ **Scalability & Elasticity** – Load Balancer and Auto Scaling dynamically handle **traffic spikes**.  

---

## **Project Repository**  
All scripts, configurations, and reference architecture diagrams are available in the **GitHub repository**.  

🔗 [GitHub Repository] https://github.com/casmaster55/Host-a-Dynamic-Web-App-on-AWS/edit/main/README.md


## **Future Improvements**  
- Implement **CI/CD pipeline** for automated deployments.  
- Add **monitoring & logging** with AWS **CloudWatch & AWS Config**.  
- Use **Terraform or CloudFormation** for infrastructure as code (IaC).  

---

## **Conclusion**  
This project successfully deploys a **scalable, highly available, and fault-tolerant** dynamic web application on **AWS** using **EC2, Auto Scaling, Load Balancer, RDS, and S3**. It follows best DevOps practices for **security, automation, and performance**.  


This **README** is now **complete, detailed, and easy to follow**! Let me know if you need further refinements! 😊🚀
