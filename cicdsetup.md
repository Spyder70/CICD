# CI/CD Setup for Deploying Static Site to AWS S3 with GitHub Actions

## Overview

This document explains the steps to automate deployment of a static website to an AWS S3 bucket using GitHub Actions.

---

## Prerequisites

- An AWS account with S3 service access.
- An S3 bucket created for hosting your static website.
- An IAM user with programmatic access and permissions to manage the S3 bucket.
- A GitHub repository containing your static site code.
- GitHub repository admin access to configure secrets and workflows.

---

## Step 1: Create IAM User with S3 Access

1. Go to AWS Console → IAM → Users → Create user.
2. Enter a user name (e.g., `github-deploy-user`).
3. Select **Programmatic access**.
4. Attach the policy `AmazonS3FullAccess` or create a custom policy with full S3 access.
5. Create the user and download the **Access Key ID** and **Secret Access Key**.

---

## Step 2: Configure S3 Bucket

1. Create your S3 bucket (e.g., `targaz`).
2. Enable **Static website hosting** in the bucket properties.
3. Disable ACLs by setting **Object Ownership** to **Bucket owner enforced**.
4. Configure **Bucket Policy** to allow public read access (optional for public websites).
5. Adjust **Block Public Access** settings as needed.

---

## Step 3: Add AWS Credentials to GitHub Secrets

1. Go to your GitHub repository → Settings → Secrets → Actions.
2. Add the following secrets:

   - `AWS_ACCESS_KEY_ID` : Your IAM user's Access Key ID.
   - `AWS_SECRET_ACCESS_KEY` : Your IAM user's Secret Access Key.
   - `AWS_REGION` : The AWS region where your bucket is located (e.g., `ap-southeast-2`).
   - `S3_BUCKET` : Your S3 bucket name (e.g., `targaz`).

---

## Step 4: Create GitHub Actions Workflow

1. In your repository, create a workflow file `.github/workflows/deploy.yml`.
2. Use the following example content:

```yaml
name: Deploy Static Site to S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Sync files to S3 bucket
      run: |
        aws s3 sync ./ s3://${{ secrets.S3_BUCKET }}/ --delete
