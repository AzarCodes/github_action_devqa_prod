# 🚀 WEB CI/CD Pipeline with GitHub Actions

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
3. Click **"New repository secret"** and add each

-----
## 🛡️ Step 2: Add `.env` to `.gitignore`

To prevent accidental exposure of credentials, ensure `.env` is excluded from version control.

```bash
# .gitignore
.env
```
----------

```bash
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

      - name: Generate .env file from secrets
        run: |
          rm -f .env
          {
            echo "HOST=${{ secrets.HOST }}"
            echo "PROTOCOL=${{ secrets.PROTOCOL }}"
            echo "PORT=${{ secrets.PORT }}"
            echo "MONGO_PARENT_URI=${{ secrets.MONGO_PARENT_URI }}"
            echo "SECRET=${{ secrets.SECRET }}"
            echo "EMAIL=${{ secrets.EMAIL }}"
            echo "NAME=${{ secrets.NAME }}"
            echo "REMAIL=${{ secrets.REMAIL }}"
            echo "APPROVAL_EMAIL=${{ secrets.APPROVAL_EMAIL }}"
            echo "APPROVER_MAIL=${{ secrets.APPROVER_MAIL }}"
            echo "SENDGRID_KEY=${{ secrets.SENDGRID_KEY }}"
            echo "LOGIN_URL=${{ secrets.LOGIN_URL }}"
          } > .env

      - name: Secure .env file
        run: chmod 600 .env

      - name: Deploy with PM2
        run: |
          pm2 start server.js --name web || pm2 reload web
          pm2 save

      - name: Clean up on failure
        if: failure()
        run: pm2 delete web || true
```
----