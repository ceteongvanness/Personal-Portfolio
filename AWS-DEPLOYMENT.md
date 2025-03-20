# AWS Deployment Guide for Personal Portfolio

This guide provides step-by-step instructions for deploying your personal portfolio website on AWS services. These instructions are beginner-friendly and don't require prior AWS experience.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Step 1: Create an AWS Account](#step-1-create-an-aws-account)
- [Step 2: Set Up Amazon S3 Bucket](#step-2-set-up-amazon-s3-bucket)
- [Step 3: Upload Your Website Files](#step-3-upload-your-website-files)
- [Step 4: Test Your Website](#step-4-test-your-website)
- [Step 5: Set Up CloudFront (Optional)](#step-5-set-up-cloudfront-optional)
- [Step 6: Add a Custom Domain (Optional)](#step-6-add-a-custom-domain-optional)
- [Step 7: Set Up Continuous Deployment (Optional)](#step-7-set-up-continuous-deployment-optional)
- [Troubleshooting](#troubleshooting)
- [Cost Estimation](#cost-estimation)

## Prerequisites
- Your portfolio website files ready for deployment
- A credit/debit card for AWS account verification (minimal charges expected)
- (Optional) A custom domain name if you want a personalized URL

## Step 1: Create an AWS Account
1. Go to [aws.amazon.com](https://aws.amazon.com/) and click "Create an AWS Account"
2. Follow the registration process:
   - Provide email address and AWS account name
   - Enter your personal information
   - Add payment information (required even for free tier)
   - Verify your identity via phone
   - Select the free basic support plan

## Step 2: Set Up Amazon S3 Bucket
S3 (Simple Storage Service) is an object storage service that we'll use to host your static website.

1. **Log into the AWS Management Console** at [console.aws.amazon.com](https://console.aws.amazon.com/)
2. **Create a new S3 bucket**:
   - Search for "S3" in the services search bar
   - Click "Create bucket"
   - Enter a globally unique bucket name (e.g., "your-name-portfolio")
   - Uncheck "Block all public access" (necessary for website hosting)
   - Acknowledge the warning that appears
   - Keep other settings as default
   - Click "Create bucket"

3. **Enable Static Website Hosting**:
   - Select your newly created bucket
   - Go to the "Properties" tab
   - Scroll down to "Static website hosting"
   - Click "Edit"
   - Select "Enable"
   - Enter "index.html" as the Index document
   - (Optional) Enter "error.html" as the Error document if you have one
   - Click "Save changes"

4. **Set Bucket Policy for Public Access**:
   - Go to the "Permissions" tab
   - Under "Bucket policy", click "Edit"
   - Copy and paste this policy (replace "your-bucket-name" with your actual bucket name):
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::your-bucket-name/*"
       }
     ]
   }
   ```
   - Click "Save changes"

## Step 3: Upload Your Website Files
1. Go to the "Objects" tab of your bucket
2. Click "Upload"
3. Drag and drop all your website files or click "Add files" to select them
   - Make sure to maintain your folder structure
   - Ensure that all files are included (HTML, CSS, JavaScript, images, etc.)
4. Keep default settings and click "Upload"
5. Verify all files appear correctly in your bucket

## Step 4: Test Your Website
1. Go back to the "Properties" tab
2. Scroll down to "Static website hosting" section
3. You'll see a "Bucket website endpoint" URL (looks like `http://your-bucket-name.s3-website-region.amazonaws.com`)
4. Click the link or copy it to your browser to see your live website
5. Test all pages and functionalities to ensure everything works correctly

## Step 5: Set Up CloudFront (Optional)
CloudFront is AWS's content delivery network (CDN) that speeds up distribution of your website and provides HTTPS.

1. Search for "CloudFront" in the AWS services search bar
2. Click "Create Distribution"
3. Under "Origin domain", select your S3 bucket website endpoint
4. For "Origin path", leave it blank
5. For "Default root object", enter "index.html"
6. Under "Custom SSL certificate" (optional):
   - You can request a free SSL certificate via AWS Certificate Manager
   - Follow the prompts to validate ownership of your domain
7. Leave other settings as default for now
8. Click "Create distribution"
9. Wait for the distribution to deploy (status will change from "In Progress" to "Deployed")
10. Once deployed, use the CloudFront domain name (e.g., `d1234abcdef.cloudfront.net`) to access your website

## Step 6: Add a Custom Domain (Optional)
If you want to use your own domain name instead of the AWS URLs:

1. **Register a domain** (if you don't already have one):
   - You can use Route 53 (AWS's domain registrar) or any other registrar
   - Follow the registration process and complete the purchase

2. **Create a hosted zone in Route 53**:
   - Search for "Route 53" in the AWS services
   - Go to "Hosted zones" and click "Create hosted zone"
   - Enter your domain name and click "Create"

3. **Set up DNS records**:
   - If using CloudFront:
     - Create an "A" record
     - Set "Alias" to "Yes"
     - Select your CloudFront distribution as the alias target
   - If using S3 directly:
     - Create a CNAME record pointing to your S3 website endpoint

4. **Update nameservers at your domain registrar** (if not using Route 53):
   - Copy the nameservers from your Route 53 hosted zone
   - Update the nameservers at your domain registrar

5. **Update CloudFront distribution** (if using CloudFront):
   - Edit your distribution
   - Add your custom domain in the "Alternate Domain Names (CNAMEs)" field
   - Update and deploy the changes

## Step 7: Set Up Continuous Deployment (Optional)
For automated deployments whenever you update your website:

1. **Create an IAM User for deployments**:
   - Search for "IAM" in AWS services
   - Create a new user with programmatic access
   - Attach a policy granting S3 and CloudFront permissions
   - Save the access key ID and secret access key

2. **Use GitHub Actions** (if using GitHub):
   - Create a `.github/workflows/deploy.yml` file in your repository:
   ```yaml
   name: Deploy to AWS
   
   on:
     push:
       branches: [ main ]
   
   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
       - uses: actions/checkout@v3
       
       - name: Configure AWS credentials
         uses: aws-actions/configure-aws-credentials@v1
         with:
           aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
           aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
           aws-region: us-east-1
           
       - name: Deploy to S3
         run: aws s3 sync . s3://your-bucket-name --delete
         
       - name: Invalidate CloudFront cache
         run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
   ```
   - Add your AWS credentials as GitHub secrets

## Troubleshooting

### Common Issues:
1. **Website not accessible:**
   - Verify bucket policy allows public read access
   - Check that static website hosting is enabled
   - Ensure index.html is at the root of your bucket

2. **CSS/JS not loading:**
   - Check file paths in your HTML
   - Verify all files were uploaded correctly
   - Check for CORS issues (may need to configure CORS on your bucket)

3. **CloudFront showing old content:**
   - Create an invalidation to clear the cache
   - In CloudFront console, select your distribution → Invalidations → Create Invalidation
   - Enter "/*" to invalidate all paths

4. **Custom domain not working:**
   - DNS changes can take up to 48 hours to propagate
   - Verify DNS records are correctly configured
   - Check that your CloudFront distribution includes the custom domain

## Cost Estimation
AWS has a pay-as-you-go pricing model. For a personal portfolio website with light traffic:

- **S3**: ~$0.023 per GB/month for storage + minimal request charges
  - Example: A 50MB website might cost $0.01/month for storage
  
- **CloudFront**: ~$0.085 per GB of data transfer + minimal request charges
  - Example: 1GB of monthly traffic might cost $0.09/month
  
- **Route 53** (if using): $0.50/month for hosted zone + domain registration fee

**Total estimated monthly cost**: $1-3 for most personal portfolio websites

---

If you encounter any issues not covered in this guide, visit the [AWS Documentation](https://docs.aws.amazon.com/) or contact AWS Support.
