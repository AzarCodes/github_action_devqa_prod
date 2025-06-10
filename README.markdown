# 🚀 Utkarsh India - Production Deployment FrontEnd

This is the production frontend codebase hosted via **AWS Amplify**. Amplify automatically builds and deploys the app whenever new changes are pushed to the repository.

> ✅ No additional configuration required — Amplify uses its default settings for building and deploying the app.

---

## 📁 Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Deployment Flow](#deployment-flow)
- [Default Build Settings](#default-build-settings)
- [Branching Strategy](#branching-strategy)
- [Monitoring & Logs](#monitoring--logs)
- [Custom Domain & SSL Setup](#custom-domain--ssl-setup)
- [Support](#support)

---

## 🔍 Overview

The frontend application is deployed using **AWS Amplify Hosting**, which provides:

- **Automatic builds** when changes are pushed to the repo
- **Zero-config deployment**
- **Global CDN distribution**
- **Custom domain and HTTPS support**

All deployments are triggered from the `main` branch.

---

## ⚙️ How It Works

AWS Amplify monitors this Git repository (GitHub, GitLab, Bitbucket, or AWS CodeCommit) for changes. When a commit is pushed:

1. Amplify detects the change
2. Automatically triggers a new build
3. Deploys the updated app if the build succeeds

No manual steps or configuration files (`amplify.yml`) are needed unless customization is required later.

---

## 🔄 Deployment Flow

To deploy changes:

1. **Make updates in your local development environment.**
   - Clone the repository if you haven't already:
     ```bash
     git clone <repository-url>
     cd <repository-folder>
     ```
   - Install dependencies:
     ```bash
     npm install
     ```
     or
     ```bash
     yarn install
     ```

2. **Commit and push your changes to the `main` branch:**
   ```bash
   git add .
   git commit -m "Update feature"
   git push origin main
   ```

3. **Amplify will:**
   - Pull the latest code
   - Run the default build command
   - Deploy the new version

4. **Configure AWS Amplify (Initial Setup):**
   - Follow the prompts to:
     - Select your AWS profile
     - Choose the environment (e.g., `prod` for production)
     - Select the frontend framework (e.g., React, Vue, Angular)
     - Specify the source directory (e.g., `src`)
     - Set the build output directory (e.g., `build` or `dist`)

5. **Add Hosting with Amplify (Initial Setup):**
   - Choose **Amplify Hosting** (Continuous deployment). AWS Amplify will prompt you to select the respective repository (e.g., GitHub, GitLab, Bitbucket, or AWS CodeCommit). After selecting the repository, Amplify automatically generates a default `amplify.yml` build template tailored to your frontend framework.

---

## 📋 Default Build Settings

AWS Amplify automatically generates a default `amplify.yml` file when you set up hosting. This file defines the build settings for your project. Below is an example of the default configuration Amplify might generate for a typical frontend project:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: build
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

You can modify this file in the AWS Amplify Console under **Build settings** if your project requires custom build commands or a different output directory.

---

## 🌿 Branching Strategy

- **main**: Production-ready code. All deployments are triggered from this branch.
- **dev**: Development branch for integrating features and testing.
- **feature/***: Feature branches for specific tasks (e.g., `feature/login-page`).

To work on a new feature:
1. Create a feature branch:
   ```bash
   git checkout -b feature/<feature-name>
   ```
2. Make changes, commit, and push:
   ```bash
   git add .
   git commit -m "Add <feature-name>"
   git push origin feature/<feature-name>
   ```
3. Create a pull request to merge into `dev` for testing.
4. Once approved, merge `dev` into `main` to trigger a production deployment.

---

## 📊 Monitoring & Logs

- **Build Logs**: View build and deployment logs in the AWS Amplify Console under the **App** section.
- **App Monitoring**: Use the Amplify Console to monitor deployment status, rollbacks, and domain management.
- **Custom Metrics**: If needed, integrate AWS CloudWatch for additional monitoring (configured separately).

---

## 🌐 Custom Domain & SSL Setup

AWS Amplify supports custom domains and automatically provisions SSL certificates for secure HTTPS access. Follow these steps to set up a custom domain:

1. **Go to the AWS Amplify Console**:
   - Navigate to your app in the Amplify Console.
   - Select the **Domain management** section.

2. **Add a Custom Domain**:
   - Click **Add domain**.
   - Enter your domain (e.g., `example.com` or `www.example.com`).
   - If your domain is not managed by AWS Route 53, you’ll need to update your DNS provider with the CNAME records provided by Amplify.

3. **Configure Subdomains** (optional):
   - Amplify allows you to set up subdomains (e.g., `www.example.com` or `app.example.com`).
   - Map subdomains to specific branches if needed (e.g., `main` for production).

4. **SSL Certificate**:
   - Amplify automatically provisions and renews an SSL certificate for your domain via AWS Certificate Manager (ACM).
   - It may take a few minutes for the DNS verification to complete.

5. **Verify and Test**:
   - Once the domain status is **Available** in the Amplify Console, test your domain (e.g., `https://example.com`).
   - Ensure the site loads securely with HTTPS.

For more details, refer to the [AWS Amplify Custom Domains Documentation](https://docs.aws.amazon.com/amplify/latest/userguide/custom-domains.html).

---
