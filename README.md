# AWS CDK + CodePipeline Assignment

A demonstration of using the AWS Cloud Development Kit (CDK) in TypeScript to provision AWS resources (all within the Free Tier) and wire up a GitHub-backed CodePipeline for continuous deployment.

---

## Overview

This project uses AWS CDK (TypeScript) to create three AWS resources:

- **S3 Bucket**: A versioned bucket for storing static assets.
- **Lambda Function**: A Node.js 18.x function that logs to CloudWatch and returns “Hello, World!”.
- **DynamoDB Table**: A simple table named `MyTable` with a string partition key `id`.

A **CodePipeline** watches the GitHub `main` branch. On each push it:

1. **Checks out** your CDK code.
2. **Runs** CodeBuild with `buildspec.yml`:
   - `npm install`
   - `npm run build` (TypeScript → JavaScript)
   - `cdk synth` (generates CloudFormation)
   - `cdk deploy --require-approval never --region us-east-1`

---

## Architecture

```text
 GitHub Repo
     ↓ (push)
 CodePipeline (us-east-1)
   ├── Source (GitHub)
   └── Build (CodeBuild)
         └── CDK Synth & Deploy
                ├── S3 Bucket (MyFirstBucket…)
                ├── Lambda Function (MyLambda…)
                └── DynamoDB Table (MyTable)
```

---

## Prerequisites

- **AWS Account** with permissions to create S3, Lambda, DynamoDB, IAM, SSM.
- **AWS CLI** installed & configured (`aws configure`).
- **Node.js & npm** (v16+).
- **AWS CDK Toolkit**:
  ```bash
  npm install -g aws-cdk
  ```
- **Git** and a GitHub account.

---

## Installation & Deployment

1. **Clone the repo**

   ```bash
   git clone https://github.com/nmrepos/PROG8860A2.git
   cd PROG8860A2
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **Bootstrap CDK** (once per account/region)

   ```bash
   cdk bootstrap aws://<ACCOUNT_ID>/us-east-1
   ```

4. **Deploy manually** (optional verification)

   ```bash
   npm run build
   cdk deploy --require-approval never --region us-east-1
   ```

5. **Push to GitHub** to trigger the pipeline:

   ```bash
   git add .
   git commit -m "Initial CDK deployment"
   git push origin main
   ```

---

## CI/CD Pipeline

- **Source**: GitHub `main` branch (via OAuth).
- **Build**: AWS CodeBuild uses the `buildspec.yml` at repo root.
- **Deploy**: CDK CLI in the build step runs `cdk deploy` automatically.

You can view the pipeline in the AWS Console under **CodePipeline** (region `us-east-1`).

---

## Verification

1. **S3**

   - Navigate to **S3** → confirm bucket `MyFirstBucket…` exists and versioning is enabled.
   - Upload & download a test file.

2. **Lambda**

   - Navigate to **Lambda** → `MyLambda…`.
   - Confirm Runtime: Node.js 18.x, Environment variable `BUCKET_NAME`.
   - Create a “Hello World” test event → invoke → see a 200 response and “Lambda invoked” log.

3. **DynamoDB**

   - Navigate to **DynamoDB** → Table `MyTable`.
   - Click **Explore table items** → add an item with `id="test123"` → verify it appears.

4. **Pipeline**

   - Push any small change to `main` → pipeline auto-triggers → both Source & Build stages turn green.

---

## Cleanup

To avoid ongoing charges:

1. **Destroy CDK stack**

   ```bash
   cdk destroy --force --region us-east-1
   ```

2. **Delete leftover S3 buckets** (if any) via the S3 console.

---

