# 🚀 Production CI/CD Pipeline with GitHub Actions

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
    branches: Production
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

      - name: Generate .env from secrets
        run: |
          echo "HOST=${{ secrets.HOST }}" > .env
          echo "PROTOCOL=HTTPS" >> .env
          echo "PORT=4001" >> .env
          echo "DATABASE=${{ secrets.DATABASE }}" >> .env
          echo "MONGO_PARENT_URI=${{ secrets.MONGO_PARENT_URI }}" >> .env
          echo "MONGO_URI=${{ secrets.MONGO_URI }}" >> .env
          echo "SECRET=${{ secrets.SECRET }}" >> .env
          echo "EMAIL=${{ secrets.EMAIL }}" >> .env
          echo "NAME=${{ secrets.NAME }}" >> .env
          echo "REMAIL=${{ secrets.REMAIL }}" >> .env
          echo "APPROVAL_EMAIL=${{ secrets.APPROVAL_EMAIL }}" >> .env
          echo "OPENAI_API_KEY=${{ secrets.OPENAI_API_KEY }}" >> .env
          echo "APPROVER_MAIL=${{ secrets.APPROVER_MAIL }}" >> .env
          echo "RAZORPAY_KEY_ID=${{ secrets.RAZORPAY_KEY_ID }}" >> .env
          echo "RAZORPAY_KEY_SECRET=${{ secrets.RAZORPAY_KEY_SECRET }}" >> .env
          echo "PLAN_ID=${{ secrets.PLAN_ID }}" >> .env
          echo "SENDGRID_KEY=${{ secrets.SENDGRID_KEY }}" >> .env
          echo "SALESFORCE_AUTH_URL=${{ secrets.SALESFORCE_AUTH_URL }}" >> .env
          echo "SALESFORCE_TOKEN_URL=${{ secrets.SALESFORCE_TOKEN_URL }}" >> .env
          echo "SALESFORCE_REDIRECT_URI=${{ secrets.SALESFORCE_REDIRECT_URI }}" >> .env
          echo "ENCRYPTION_KEY=${{ secrets.ENCRYPTION_KEY }}" >> .env
          echo "CRYPTO_KEY=${{ secrets.CRYPTO_KEY }}" >> .env
          echo "AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}" >> .env
          echo "AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}" >> .env
          echo "AWS_REGION=${{ secrets.AWS_REGION }}" >> .env
          echo "AWS_S3_BUCKET_NAME=${{ secrets.AWS_S3_BUCKET_NAME }}" >> .env

      - name: Deploy with PM2
        run: |
          pm2 start server.js --name ddutkarshprod || pm2 reload ddutkarshprod

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