# 🚀 DevQA CI/CD Pipeline with GitHub Actions

This repository contains a secure and production-ready CI/CD pipeline configuration for deploying a Node.js application using PM2 and GitHub Actions. Sensitive environment variables are managed through GitHub Secrets and automatically injected into a `.env` file at runtime.

---

## 📁 Project Structure

- `.github/workflows/deploy.yml`: GitHub Actions workflow file.
- `.env`: Environment variable file (auto-generated during deployment).
- `server.js`: Main application entry point.
- `package.json`: Node.js dependency and script definitions.

---

## 🔐 Step 1: Add Secrets to GitHub

All sensitive credentials (e.g., database URIs, API keys, SMTP settings) must be stored securely.

1. Go to your GitHub repository.
2. Navigate to **Settings → Secrets and variables → Actions**.
3. Click **"New repository secret"** and add each of the following keys:

| GitHub Secret Name               | Description                      |
|----------------------------------|----------------------------------|
| `MONGO_PARENT_URI`              | MongoDB connection string (root) |
| `MONGO_URI`                     | MongoDB database URI             |
| `SECRET`                        | JWT secret or app secret         |
| `EMAIL`                         | Default sender email             |
| `NAME`                          | App or brand name                |
| `REMAIL`                        | Reply-to email address           |
| `APPROVAL_EMAIL`                | Workflow approval email          |
| `OPENAI_API_KEY`                | OpenAI API Key                   |
| `APPROVER_MAIL`                 | Internal approver email          |
| `RAZORPAY_KEY_ID`               | Razorpay public key              |
| `RAZORPAY_KEY_SECRET`           | Razorpay secret key              |
| `PLAN_ID`                       | Razorpay subscription plan ID    |
| `SENDGRID_KEY`                  | SendGrid API Key                 |
| `SALESFORCE_AUTH_URL`           | Salesforce OAuth2 URL            |
| `SALESFORCE_TOKEN_URL`          | Salesforce token URL             |
| `SALESFORCE_REDIRECT_URI`       | Salesforce redirect URI          |
| `ENCRYPTION_KEY`                | AES encryption key               |
| `CRYPTO_KEY`                    | Additional crypto key            |
| `AWS_ACCESS_KEY_ID`             | AWS IAM access key               |
| `AWS_SECRET_ACCESS_KEY`         | AWS IAM secret key               |
| `AWS_REGION`                    | AWS region                       |
| `AWS_S3_BUCKET_NAME`            | S3 bucket name                   |
| `HOST`                          | Deployment host (e.g., domain)   |
| `DATABASE`                      | MongoDB database name            |

---

## 🛡️ Step 2: Add `.env` to `.gitignore`

To prevent accidental exposure of credentials, ensure `.env` is excluded from version control.

```bash
# .gitignore
.env
```
---

## Step 3: GitHub Actions Workflow Configuration

Here’s the CI/CD pipeline defined in .github/workflows/deploy.yml

```bash

name: CI/CD Pipeline

on:
  push:
    branches: ["UAT", "DEV-NEW"]
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

      - name: Generate and validate .env file
        run: |
          # Determine environment based on branch
          if [[ "${{ github.ref_name }}" == "DEV-NEW" ]]; then
            HOST=${{ secrets.DEV_HOST }}
            DATABASE=${{ secrets.DEV_DATABASE }}
            MONGO_PARENT_URI=${{ secrets.DEV_MONGO_PARENT_URI }}
            MONGO_URI=${{ secrets.DEV_MONGO_URI }}
            SECRET=${{ secrets.DEV_SECRET }}
            EMAIL=${{ secrets.DEV_EMAIL }}
            NAME=${{ secrets.DEV_NAME }}
            REMAIL=${{ secrets.DEV_REMAIL }}
            APPROVAL_EMAIL=${{ secrets.DEV_APPROVAL_EMAIL }}
            OPENAI_API_KEY=${{ secrets.DEV_OPENAI_API_KEY }}
            APPROVER_MAIL=${{ secrets.DEV_APPROVER_MAIL }}
            RAZORPAY_KEY_ID=${{ secrets.DEV_RAZORPAY_KEY_ID }}
            RAZORPAY_KEY_SECRET=${{ secrets.DEV_RAZORPAY_KEY_SECRET }}
            PLAN_ID=${{ secrets.DEV_PLAN_ID }}
            SENDGRID_KEY=${{ secrets.DEV_SENDGRID_KEY }}
            SALESFORCE_AUTH_URL=${{ secrets.DEV_SALESFORCE_AUTH_URL }}
            SALESFORCE_TOKEN_URL=${{ secrets.DEV_SALESFORCE_TOKEN_URL }}
            SALESFORCE_REDIRECT_URI=${{ secrets.DEV_SALESFORCE_REDIRECT_URI }}
            ENCRYPTION_KEY=${{ secrets.DEV_ENCRYPTION_KEY }}
            CRYPTO_KEY=${{ secrets.DEV_CRYPTO_KEY }}
            AWS_ACCESS_KEY_ID=${{ secrets.DEV_AWS_ACCESS_KEY_ID }}
            AWS_SECRET_ACCESS_KEY=${{ secrets.DEV_AWS_SECRET_ACCESS_KEY }}
            AWS_REGION=${{ secrets.DEV_AWS_REGION }}
            AWS_S3_BUCKET_NAME=${{ secrets.DEV_AWS_S3_BUCKET_NAME }}
            LOGIN_URL=${{ secrets.DEV_LOGIN_URL }}
          elif [[ "${{ github.ref_name }}" == "UAT" ]]; then
            HOST=${{ secrets.UAT_HOST }}
            DATABASE=${{ secrets.UAT_DATABASE }}
            MONGO_PARENT_URI=${{ secrets.UAT_MONGO_PARENT_URI }}
            MONGO_URI=${{ secrets.UAT_MONGO_URI }}
            SECRET=${{ secrets.UAT_SECRET }}
            EMAIL=${{ secrets.UAT_EMAIL }}
            NAME=${{ secrets.UAT_NAME }}
            REMAIL=${{ secrets.UAT_REMAIL }}
            APPROVAL_EMAIL=${{ secrets.UAT_APPROVAL_EMAIL }}
            OPENAI_API_KEY=${{ secrets.UAT_OPENAI_API_KEY }}
            APPROVER_MAIL=${{ secrets.UAT_APPROVER_MAIL }}
            RAZORPAY_KEY_ID=${{ secrets.UAT_RAZORPAY_KEY_ID }}
            RAZORPAY_KEY_SECRET=${{ secrets.UAT_RAZORPAY_KEY_SECRET }}
            PLAN_ID=${{ secrets.UAT_PLAN_ID }}
            SENDGRID_KEY=${{ secrets.UAT_SENDGRID_KEY }}
            SALESFORCE_AUTH_URL=${{ secrets.UAT_SALESFORCE_AUTH_URL }}
            SALESFORCE_TOKEN_URL=${{ secrets.UAT_SALESFORCE_TOKEN_URL }}
            SALESFORCE_REDIRECT_URI=${{ secrets.UAT_SALESFORCE_REDIRECT_URI }}
            ENCRYPTION_KEY=${{ secrets.UAT_ENCRYPTION_KEY }}
            CRYPTO_KEY=${{ secrets.UAT_CRYPTO_KEY }}
            AWS_ACCESS_KEY_ID=${{ secrets.UAT_AWS_ACCESS_KEY_ID }}
            AWS_SECRET_ACCESS_KEY=${{ secrets.UAT_AWS_SECRET_ACCESS_KEY }}
            AWS_REGION=${{ secrets.UAT_AWS_REGION }}
            AWS_S3_BUCKET_NAME=${{ secrets.UAT_AWS_S3_BUCKET_NAME }}
            LOGIN_URL=${{ secrets.UAT_LOGIN_URL }}
          else
            echo "Unsupported branch: ${{ github.ref_name }}"
            exit 1
          fi

          cat <<EOF > .env
          HOST=$HOST
          PROTOCOL=HTTPS
          PORT=4001
          DATABASE=$DATABASE
          MONGO_PARENT_URI=$MONGO_PARENT_URI
          MONGO_URI=$MONGO_URI
          SECRET=$SECRET
          EMAIL=$EMAIL
          NAME=$NAME
          REMAIL=$REMAIL
          APPROVAL_EMAIL=$APPROVAL_EMAIL
          OPENAI_API_KEY=$OPENAI_API_KEY
          APPROVER_MAIL=$APPROVER_MAIL
          RAZORPAY_KEY_ID=$RAZORPAY_KEY_ID
          RAZORPAY_KEY_SECRET=$RAZORPAY_KEY_SECRET
          PLAN_ID=$PLAN_ID
          SENDGRID_KEY=$SENDGRID_KEY
          SALESFORCE_AUTH_URL=$SALESFORCE_AUTH_URL
          SALESFORCE_TOKEN_URL=$SALESFORCE_TOKEN_URL
          SALESFORCE_REDIRECT_URI=$SALESFORCE_REDIRECT_URI
          ENCRYPTION_KEY=$ENCRYPTION_KEY
          CRYPTO_KEY=$CRYPTO_KEY
          AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
          AWS_REGION=$AWS_REGION
          AWS_S3_BUCKET_NAME=$AWS_S3_BUCKET_NAME
          LOGIN_URL=$LOGIN_URL
          EOF

          # Validate all required variables
          required_vars=(HOST DATABASE MONGO_PARENT_URI MONGO_URI SECRET EMAIL NAME REMAIL APPROVAL_EMAIL OPENAI_API_KEY APPROVER_MAIL RAZORPAY_KEY_ID RAZORPAY_KEY_SECRET PLAN_ID SENDGRID_KEY SALESFORCE_AUTH_URL SALESFORCE_TOKEN_URL SALESFORCE_REDIRECT_URI ENCRYPTION_KEY CRYPTO_KEY AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION AWS_S3_BUCKET_NAME LOGIN_URL)
          for var in "${required_vars[@]}"; do
            if ! grep -q "^$var=" .env || grep -q "^$var=$" .env; then
              echo "Error: Missing or empty value for $var"
              exit 1
            fi
          done

      - name: Secure .env file
        run: chmod 600 .env

      - name: Deploy with PM2
        run: |
          echo "Deploying branch: ${{ github.ref_name }}"
          if [[ "${{ github.ref_name }}" == "DEV-NEW" ]]; then
            pm2_app_name="dddevnew"
          elif [[ "${{ github.ref_name }}" == "UAT" ]]; then
            pm2_app_name="dduat"
          else
            echo "Unknown branch for PM2 deployment"
            exit 1
          fi

          pm2 start server.js --name "$pm2_app_name" || pm2 reload "$pm2_app_name"

      - name: Clean up on failure
        if: failure()
        run: pm2 delete all || true
```
---
## ✅ Final Checklist

-  Store all credentials as GitHub Secrets.
-  Delete original **.env** file containing secrets from the repo.
-   Add .env to *.gitignore*.
-  Make sure PM2 is installed and globally available on the server.