# 🚀 Utkarsh India - Production Deployment Backend

This is the production backend setup guide for hosting the application on an **AWS EC2 instance**. The setup includes creating a VPC, subnet, internet gateway, and installing necessary tools like Nginx, Git, Certbot, and PM2 to run the application.

---

## 📁 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
  - [Create a VPC](#create-a-vpc)
  - [Create a Subnet](#create-a-subnet)
  - [Set Up an Internet Gateway](#set-up-an-internet-gateway)
  - [Create and Configure an EC2 Instance](#create-and-configure-an-ec2-instance)
  - [Install Dependencies](#install-dependencies)
  - [Deploy the Application](#deploy-the-application)
  - [Automate Deployment with GitHub Actions](#automate-deployment-with-github-actions)
- [Monitoring & Logs](#monitoring--logs)

---

## 🔍 Overview

The backend application is hosted on an **AWS EC2 instance** within a custom VPC. Key components include:

- **VPC and Networking**: Custom VPC with a subnet and internet gateway for secure and accessible deployment.
- **EC2 Instance**: Hosts the backend application.
- **Nginx**: Reverse proxy for routing requests.
- **Git**: For cloning the repository.
- **Certbot**: For SSL/TLS certificates to enable HTTPS.
- **PM2**: Process manager to run and manage the Node.js application.

The application is started using `pm2 start server.js --name app-name`.

---

## ✅ Prerequisites

- **AWS Account** with appropriate permissions (EC2, VPC, etc.)
- **AWS CLI** installed and configured (`aws configure`)
- **SSH Client** (e.g., OpenSSH, MobaXterm) to connect to the EC2 instance
- **Node.js Application Repository** (e.g., on GitHub, GitLab, or Bitbucket)
- **Domain Name** (optional, for SSL setup with Certbot)

---

## ⚙️ Setup Instructions

### Create a VPC

1. **Go to the AWS Management Console** > **VPC** > **Create VPC**.
2. Configure the VPC:
   - Name: `VPC-Name`
   - IPv4 CIDR block: `10.0.0.0/16`
   - Leave other settings as default.
3. Click **Create VPC**.

### Create a Subnet

1. In the VPC Dashboard, go to **Subnets** > **Create subnet**.
2. Configure the subnet:
   - VPC: Select `backend-vpc`.
   - Name: `backend-subnet`.
   - Availability Zone: Choose one (e.g., `us-east-1a`).
   - IPv4 CIDR block: `10.0.1.0/24`.
3. Click **Create subnet**.

### Set Up an Internet Gateway

1. In the VPC Dashboard, go to **Internet Gateways** > **Create internet gateway**.
2. Name it `backend-igw` and click **Create**.
3. Attach it to the VPC:
   - Select the internet gateway, click **Actions** > **Attach to VPC**.
   - Choose `backend-vpc`.
4. Update the route table:
   - Go to **Route Tables**, select the route table associated with `backend-vpc`.
   - Go to **Routes** > **Edit routes**.
   - Add a route: Destination `0.0.0.0/0`, Target `backend-igw`.
   - Save the changes.
5. Associate the subnet:
   - Go to **Subnet associations** > **Edit subnet associations**.
   - Select `backend-subnet` and save.

### Create and Configure an EC2 Instance

1. **Go to the EC2 Dashboard** > **Instances** > **Launch instances**.
2. Configure the instance:
   - Name: `Instance Name`.
   - AMI: Choose `Amazon Linux 2 AMI` (or your preferred OS).
   - Instance Type: `t2.micro` (or as needed).
   - Key Pair: Create or select an existing key pair for SSH access.
   - Network Settings:
     - VPC: `backend-vpc`.
     - Subnet: `backend-subnet`.
     - Auto-assign public IP: Enable.
   - Security Group:
     - Create a new security group or edit the default one.
     - Add rules:
       - SSH (port 22) from your IP (or `0.0.0.0/0` for testing).
       - HTTP (port 80) from `0.0.0.0/0`.
       - HTTPS (port 443) from `0.0.0.0/0`.
       - Custom TCP (port 3000, or your app’s port) from `0.0.0.0/0` (temporary, for testing).
3. Launch the instance.
4. Connect to the instance via SSH:
   ```bash
   ssh -i <your-key.pem> ec2-user@<instance-public-ip>
   ```

### Install Dependencies

1. **Update the system**:
   ```bash
   sudo yum update -y
   ```

2. **Install Nginx**:
   ```bash
   sudo amazon-linux-extras install nginx1 -y
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

3. **Install Git**:
   ```bash
   sudo yum install git -y
   ```

4. **Install Node.js and PM2**:
   - Install Node.js:
     ```bash
     curl -sL https://rpm.nodesource.com/setup_16.x | sudo bash -
     sudo yum install -y nodejs
     ```
   - Install PM2 globally:
     ```bash
     sudo npm install -g pm2
     ```

5. **Install Certbot for SSL**:
   ```bash
   sudo yum install -y certbot python3-certbot-nginx
   ```

6. **Clone the Repository**:
   - Clone your backend repository:
     ```bash
     git clone <repository-url>
     cd <repository-folder>
     ```
   - Install application dependencies:
     ```bash
     npm install
     ```

### Deploy the Application

1. **Start the Application with PM2**:
   - From the repository directory, start the app:
     ```bash
     pm2 start serverr.js --name app-name
     ```
   - Save the PM2 process list and enable startup:
     ```bash
     pm2 save
     pm2 startup
     ```
   - Follow the on-screen instructions to set up PM2 to run on system boot.

2. **Configure Nginx as a Reverse Proxy**:
   - Edit the Nginx configuration:
     ```bash
     sudo nano /etc/nginx/nginx.conf
     ```
   - Add a server block inside the `http` block (or modify the default server block):
     ```
     server {
         listen 80;
         server_name <your-domain>;

         location / {
             proxy_pass http://localhost:3000; # Adjust port if your app uses a different one
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }
     }
     ```
   - Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).
   - Test the configuration and restart Nginx:
     ```bash
     sudo nginx -t
     sudo systemctl restart nginx
     ```

3. **Set Up SSL with Certbot** (if you have a domain):
   - Run Certbot to obtain and install an SSL certificate:
     ```bash
     sudo certbot --nginx -d <your-domain>
     ```
   - Follow the prompts to configure HTTPS. Certbot will automatically update the Nginx configuration to use SSL.
   - Test automatic renewal:
     ```bash
     sudo certbot renew --dry-run
     ```

4. **Update Security Group**:
   - After setting up Nginx and SSL, update the EC2 security group to remove the rule for port 3000 (or your app’s port) if you allowed it earlier, as traffic should now go through Nginx (ports 80/443).

### Automate Deployment with GitHub Actions

To automate deployment when code is pushed to the repository, set up a GitHub Actions workflow. The workflow will SSH into the EC2 instance, navigate to the repository folder, and run a `.run.sh` script.

1. **Set Up SSH Access for GitHub Actions**:
   - Ensure your EC2 instance allows SSH access (port 22 is open in the security group).
   - Add your SSH private key to GitHub Secrets:
     - Go to your repository on GitHub > **Settings** > **Secrets and variables** > **Actions** > **New repository secret**.
     - Add a secret named `SSH_PRIVATE_KEY` with the contents of your private key (`<your-key.pem>`).
   - Add the EC2 instance’s public IP and username to GitHub Secrets as `EC2_HOST` (e.g., `<instance-public-ip>`) and `EC2_USER` (e.g., `ec2-user`).

2. **Create a GitHub Actions Workflow**:
   - In your repository, create a file at `.github/workflows/deploy.yml`:
     ```yaml
     name: CI/CD Pipeline

on:
  push:
    branches: [Production]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build the app (if build script exists)
        run: |
          if npm run | grep -q "build"; then
            echo "✅ Build script found. Running npm run build..."
            npm run build
          else
            echo "⚠️ No build script found in package.json. Skipping build step."
          fi

      - name: Generate clean .env from secrets
        run: |
          rm -f .env
          {
            echo "HOST=${{ secrets.HOST }}"
            echo "PROTOCOL=HTTPS"
            echo "PORT=4001"
            echo "DATABASE=${{ secrets.DATABASE }}"
            echo "MONGO_PARENT_URI=${{ secrets.MONGO_PARENT_URI }}"
            echo "MONGO_URI=${{ secrets.MONGO_URI }}"
            echo "SECRET=${{ secrets.SECRET }}"
            echo "EMAIL=${{ secrets.EMAIL }}"
            echo "NAME=${{ secrets.NAME }}"
            echo "REMAIL=${{ secrets.REMAIL }}"
            echo "APPROVAL_EMAIL=${{ secrets.APPROVAL_EMAIL }}"
            echo "OPENAI_API_KEY=${{ secrets.OPENAI_API_KEY }}"
            echo "APPROVER_MAIL=${{ secrets.APPROVER_MAIL }}"
            echo "RAZORPAY_KEY_ID=${{ secrets.RAZORPAY_KEY_ID }}"
            echo "RAZORPAY_KEY_SECRET=${{ secrets.RAZORPAY_KEY_SECRET }}"
            echo "PLAN_ID=${{ secrets.PLAN_ID }}"
            echo "SENDGRID_KEY=${{ secrets.SENDGRID_KEY }}"
            echo "SALESFORCE_AUTH_URL=${{ secrets.SALESFORCE_AUTH_URL }}"
            echo "SALESFORCE_TOKEN_URL=${{ secrets.SALESFORCE_TOKEN_URL }}"
            echo "SALESFORCE_REDIRECT_URI=${{ secrets.SALESFORCE_REDIRECT_URI }}"
            echo "ENCRYPTION_KEY=${{ secrets.ENCRYPTION_KEY }}"
            echo "CRYPTO_KEY=${{ secrets.CRYPTO_KEY }}"
            echo "AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}"
            echo "AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}"
            echo "AWS_REGION=${{ secrets.AWS_REGION }}"
            echo "AWS_S3_BUCKET_NAME=${{ secrets.AWS_S3_BUCKET_NAME }}"
          } > .env

      - name: Secure .env file
        run: chmod 600 .env

      - name: Deploy with PM2
        run: |
          pm2 start server.js --name app-name || pm2 reload app-name
          pm2 save

      - name: Clean up on failure
        if: failure()
        run: pm2 delete app-name || true
     ```

3. **Test the Workflow**:
   - Push a change to the `main` branch in your repository.
   - Go to the **Actions** tab in your GitHub repository to monitor the workflow.
   - The workflow will SSH into the EC2 instance, navigate to the repository folder, and execute `.run.sh`, which pulls the latest code, installs dependencies, and restarts the app with PM2.

---

## 📊 Monitoring & Logs

- **Application Logs**:
  - View PM2 logs:
    ```bash
    pm2 logs app-name
    ```
  - Check application logs in your repository folder (e.g., `logs/` if configured).
- **Nginx Logs**:
  - Access logs: `/var/log/nginx/access.log`
  - Error logs: `/var/log/nginx/error.log`
- **System Monitoring**:
  - Use AWS CloudWatch to monitor EC2 instance metrics (CPU, memory, etc.).
  - Set up CloudWatch alarms for critical events (configured separately).

---