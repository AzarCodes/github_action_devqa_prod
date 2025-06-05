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
    branches: [UAT]
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

      - name: Generate .env file from UAT secrets
        run: |
          {
            echo "UAT_HOST=${{ secrets.UAT_HOST }}"
            echo "UAT_PROTOCOL=HTTPS"
            echo "UAT_PORT=4001"
            echo "UAT_DATABASE=${{ secrets.UAT_DATABASE }}"
            echo "UAT_MONGO_PARENT_URI=${{ secrets.UAT_MONGO_PARENT_URI }}"
            echo "UAT_MONGO_URI=${{ secrets.UAT_MONGO_URI }}"
            echo "UAT_SECRET=${{ secrets.UAT_SECRET }}"
            echo "UAT_EMAIL=${{ secrets.UAT_EMAIL }}"
            echo "UAT_NAME=${{ secrets.UAT_NAME }}"
            echo "UAT_REMAIL=${{ secrets.UAT_REMAIL }}"
            echo "UAT_APPROVAL_EMAIL=${{ secrets.UAT_APPROVAL_EMAIL }}"
            echo "UAT_OPENAI_API_KEY=${{ secrets.UAT_OPENAI_API_KEY }}"
            echo "UAT_APPROVER_MAIL=${{ secrets.UAT_APPROVER_MAIL }}"
            echo "UAT_RAZORPAY_KEY_ID=${{ secrets.UAT_RAZORPAY_KEY_ID }}"
            echo "UAT_RAZORPAY_KEY_SECRET=${{ secrets.UAT_RAZORPAY_KEY_SECRET }}"
            echo "UAT_PLAN_ID=${{ secrets.UAT_PLAN_ID }}"
            echo "UAT_SENDGRID_KEY=${{ secrets.UAT_SENDGRID_KEY }}"
            echo "UAT_SALESFORCE_AUTH_URL=${{ secrets.UAT_SALESFORCE_AUTH_URL }}"
            echo "UAT_SALESFORCE_TOKEN_URL=${{ secrets.UAT_SALESFORCE_TOKEN_URL }}"
            echo "UAT_SALESFORCE_REDIRECT_URI=${{ secrets.UAT_SALESFORCE_REDIRECT_URI }}"
            echo "UAT_ENCRYPTION_KEY=${{ secrets.UAT_ENCRYPTION_KEY }}"
            echo "UAT_CRYPTO_KEY=${{ secrets.UAT_CRYPTO_KEY }}"
            echo "UAT_AWS_ACCESS_KEY_ID=${{ secrets.UAT_AWS_ACCESS_KEY_ID }}"
            echo "UAT_AWS_SECRET_ACCESS_KEY=${{ secrets.UAT_AWS_SECRET_ACCESS_KEY }}"
            echo "UAT_AWS_REGION=${{ secrets.UAT_AWS_REGION }}"
            echo "UAT_AWS_S3_BUCKET_NAME=${{ secrets.UAT_AWS_S3_BUCKET_NAME }}"
            echo "UAT_LOGIN_URL=${{ secrets.UAT_LOGIN_URL }}"
          } > .env

      - name: Secure .env file
        run: chmod 600 .env

      - name: Deploy with PM2
        run: |
          pm2 start server.js --name dduat || pm2 reload dduat
          pm2 save

      - name: Clean up on failure
        if: failure()
        run: pm2 delete dduat || true
```
-----
## ✅ Final Checklist

-  Store all credentials as GitHub Secrets.
-  Delete original **.env** file containing secrets from the repo.
-   Add .env to *.gitignore*.
-  Make sure PM2 is installed and globally available on the server.
----