# Deploying a Simple Static Website on AWS with CDK and TypeScript

You may have come across several blogs on hosting static websites on S3. This guide demonstrates how to create and deploy a simple static website using AWS CDK with TypeScript. The project will cover:

- Setting up a basic AWS CDK project structure.
- Defining infrastructure as code (IaC) with TypeScript.
- Creating AWS resources necessary for hosting a static website.
- Deploying the website to AWS.

The website source code is hosted on GitHub: [kasukur/s3-website](https://github.com/kasukur/cdk-s3-website).

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1: Initialize the Project](#step-1-initialize-the-project)
3. [Step 2: Create a Simple Website](#step-2-create-a-simple-website)
4. [Step 3: Write CDK Code to Host the Website](#step-3-write-cdk-code-to-host-the-website)
5. [Step 4: Deploy the Website](#step-4-deploy-the-website)
6. [Full Command History](#full-command-history)
7. [Clean Up](#clean-up)
8. [Conclusion](#conclusion)
9. [Referrals](#referrals)

---

## Prerequisites

Before starting, ensure you have the following installed:

- **Node.js** (>= 14.x)
- **AWS CLI** (configured with credentials)
- **AWS CDK** (>= 2.x)

To install AWS CDK globally:

```bash
npm install -g aws-cdk
```

---

## Step 1: Initialize the Project

1. **Create a project directory** and navigate to it:

   ```bash
   mkdir cdk-s3-website && cd cdk-s3-website
   ```

2. **Initialize a new CDK app**:

   ```bash
   cdk init app --language typescript
   ```

3. Install dependencies for the CDK constructs we'll use:

   ```bash
   npm install @aws-cdk/aws-s3 @aws-cdk/aws-s3-deployment
   ```

4. Verify the setup by synthesizing the stack:

   ```bash
   cdk synth
   ```

   This command generates the CloudFormation template for your stack.

---

## Step 2: Create a Simple Website

1. **Download the website tempalte from [html5up.net](https://html5up.net/dimension/download)**:

2. unzip the file and navigate to the folder and copy all the files to the website folder.

```bash
cp -r ~/Downloads/html5up-dimension ~/Documents/cdk-s3-website/website/.
```

---

## Step 3: Write CDK Code to Host the Website

Edit the `lib/cdk-s3-website-stack.ts` file to define the resources:

```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as s3deploy from 'aws-cdk-lib/aws-s3-deployment';

export class CdkS3WebsiteStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create an S3 bucket for website hosting
    const websiteBucket = new s3.Bucket(this, 'WebsiteBucket', {
      websiteIndexDocument: 'index.html',
      publicReadAccess: true,
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Allow deletion of non-empty bucket
      autoDeleteObjects: true, // Automatically delete objects when bucket is removed
    });

    // Deploy the website content to the S3 bucket
    new s3deploy.BucketDeployment(this, 'DeployWebsite', {
      sources: [s3deploy.Source.asset('./website')],
      destinationBucket: websiteBucket,
    });

    // Output the website URL
    new cdk.CfnOutput(this, 'WebsiteURL', {
      value: websiteBucket.bucketWebsiteUrl,
    });
  }
}
```

---

## Step 4: Deploy the Website

Run the following commands to deploy your website:

1. **Bootstrap the environment**:

   ```bash
   cdk bootstrap
   ```

2. You might noticed the following error:

```bash
Error: Cannot use 'publicReadAccess' property on a bucket without allowing bucket-level public access through 'blockPublicAccess' property.
    at new Bucket (/Users/sridharkasukurthy/Documents/cdk-s3-website/node_modules/aws-cdk-lib/aws-s3/lib/bucket.js:1:24460)
    ...
```

Troubleshooting: The error occurs because the `publicReadAccess: true` property is used without explicitly allowing public access by disabling the `blockPublicAccess` feature. AWS CDK enforces stricter controls to prevent unintended public access.

Solution: To fix this, we need to configure the bucket to allow public access explicitly by setting the `blockPublicAccess` feature.

Updated Code:

```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as s3Deploy from 'aws-cdk-lib/aws-s3-deployment';

export class CdkS3WebsiteStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create an S3 bucket for the website with public access explicitly allowed
    const websiteBucket = new s3.Bucket(this, 'SriSriWebsiteBucket', {
      websiteIndexDocument: 'index.html',
      publicReadAccess: true,
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Optional: Remove the bucket on stack deletion
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ACLS, // Allows public access but blocks ACLs
    });

    // Deploy website contents to the bucket
    new s3Deploy.BucketDeployment(this, 'DeployWebsite', {
      sources: [s3Deploy.Source.asset('./website')],
      destinationBucket: websiteBucket,
    });

    // Output the website URL
    new cdk.CfnOutput(this, 'WebsiteURL', {
      value: websiteBucket.bucketWebsiteUrl,
    });
  }
}
```

3. **Bootstrap the environment again**:

   ```bash
   cdk bootstrap
   ```

4. **Deploy the stack**:

   ```bash
   cdk deploy
   ```

   > Do you wish to deploy these changes (y/n)? y

During deployment, the CDK will output the URL for the hosted website. You can access your site using this URL.

---

## Full Command History

Here’s a reference of all commands used:

```bash
1  mkdir cdk-s3-website && cd cdk-s3-website
2  cdk init app --language typescript
3  npm install @aws-cdk/aws-s3 @aws-cdk/aws-s3-deployment
4  git clone https://github.com/kasukur/s3-website.git website
5  cdk synth
6  cdk bootstrap
7  cdk deploy
```

---

## Clean Up

To avoid incurring unnecessary charges, delete the stack. If the `cdk destroy` command fails due to non-empty buckets, empty the bucket manually or use the `autoDeleteObjects` property in your stack as shown above:

1. **Manually empty the bucket**:

   ```bash
   aws s3 rm s3://<bucket-name> --recursive
   ```

2. **Destroy the stack**:

   ```bash
   cdk destroy
   ```

---

## Conclusion

This project demonstrated how to use AWS CDK with TypeScript to deploy a simple static website on AWS. The focus was on showcasing the CDK’s capabilities for infrastructure provisioning and deployment automation. You can now customize and expand upon this setup for more complex use cases.

---

<a name="referrals"></a>

### Referrals

- [AWS CDK Create Project](https://cdkworkshop.com/20-typescript/20-create-project/500-deploy.html)
- [How to Add S3 BucketPolicy with AWS CDK](https://stackoverflow.com/questions/60310575/how-to-add-s3-bucketpolicy-with-aws-cdk)
- [How to Force Delete a Non-Empty S3 Bucket with Versioning Enabled](https://stackoverflow.com/questions/62694166/how-to-force-delete-a-non-empty-s3-bucket-with-versioning-enabled)
- [Debugging AWS CDK Errors](https://debugthis.dev/cdk/2020-07-08-aws-cdk-errors/)
- Cover Image by [@glenncarstenspeters](https://unsplash.com/@glenncarstenspeters) from [Unsplash](https://unsplash.com/photos/person-using-macbook-pro-npxXWgQ33ZQ)

---
