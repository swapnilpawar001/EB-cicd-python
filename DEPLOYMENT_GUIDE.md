# AWS Deployment Guide for Flask Application

This guide explains how to deploy your Flask application to AWS Elastic Beanstalk using CloudFormation **via AWS Console** (no CLI commands required).

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Preparation Steps](#preparation-steps)
3. [Deploy CloudFormation Stack](#deploy-cloudformation-stack)
4. [Post-Deployment](#post-deployment)
5. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### What You Need:
- [ ] Active AWS account with Administrator access (or permissions to create IAM roles, S3 buckets, and Elastic Beanstalk resources)
- [ ] Your AWS Account ID (you can find this in AWS Console top-right corner)
- [ ] GitHub Personal Access Token (Optional - only if you want CI/CD automation)

### GitHub Token (Optional - Only for CI/CD)
If you want automatic deployment from GitHub:
1. Go to GitHub: Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a name like "AWS Pipeline"
4. Select scopes: `repo` and `admin:repo_hook`
5. Click "Generate token"
6. **Copy and save the token** (you won't see it again!)

---

## Preparation Steps

### Step 1: Create Application ZIP File

1. Download all your project files to your local computer
2. Select these files:
   - `application.py`
   - `requirements.txt`
   - `Procfile`
   - `buildspec.yml`
3. Right-click and compress to ZIP (or use your preferred compression tool)
4. Name it: `flask-app.zip`

**Important**: Do NOT include folders like `.git`, `venv`, `__pycache__`, or the CloudFormation files.

### Step 2: Create S3 Bucket and Upload Your Application

1. **Open AWS Console** and go to **S3** service
2. Click **"Create bucket"**
   - **Bucket name**: `flask-python-app-artifacts-YOUR_ACCOUNT_ID`
     - Replace `YOUR_ACCOUNT_ID` with your actual AWS Account ID
     - Example: `flask-python-app-artifacts-123456789012`
   - **Region**: Choose your preferred region (e.g., `us-east-1`)
   - **Block Public Access**: Keep all checkboxes CHECKED (default)
   - Leave other settings as default
   - Click **"Create bucket"**

3. **Enable Versioning** on your bucket:
   - Click on your newly created bucket
   - Go to **"Properties"** tab
   - Find **"Bucket Versioning"** and click **"Edit"**
   - Select **"Enable"**
   - Click **"Save changes"**

4. **Upload your application**:
   - Click on your bucket name
   - Click **"Upload"**
   - Click **"Add files"** and select `flask-app.zip`
   - Click **"Upload"**
   - Wait for upload to complete
   - You should see `flask-app.zip` in your bucket

✅ **Required Prerequisite Complete!** You now have:
- S3 bucket: `flask-python-app-artifacts-YOUR_ACCOUNT_ID`
- Application file: `flask-app.zip` uploaded to that bucket

---

## Deploy CloudFormation Stack

Now you're ready to deploy! This is the main step.

### Step 1: Open CloudFormation Console

1. Go to AWS Console and search for **"CloudFormation"**
2. Make sure you're in the **same region** where you created your S3 bucket
3. Click **"Create stack"** → **"With new resources (standard)"**

### Step 2: Upload Template

1. On the "Create stack" page:
   - Select **"Upload a template file"**
   - Click **"Choose file"**
   - Select your `cloudformation.yml` file
   - Click **"Next"**

### Step 3: Configure Stack Parameters

1. **Stack name**: `flask-python-app-stack` (or your preferred name)

2. **Parameters** - Fill in these values:

   | Parameter | Value | Notes |
   |-----------|-------|-------|
   | **ApplicationName** | `flask-python-app` | Keep default or customize |
   | **EnvironmentName** | `flask-python-app-env` | Keep default or customize |
   | **SolutionStackName** | `64bit Amazon Linux 2023 v4.3.3 running Python 3.11` | Keep default (or check latest available) |
   | **InstanceType** | `t3.micro` | Choose from dropdown (t3.micro is free tier eligible) |
   | **GitHubRepo** | `swapnilpawar001/EB-cicd-python` | Keep default or change to your repo |
   | **GitHubBranch** | `main` | Keep default or specify your branch |
   | **GitHubToken** | Leave **EMPTY** or paste your token | Empty = No CI/CD, Token = Auto CI/CD |

3. Click **"Next"**

### Step 4: Configure Stack Options

1. **Tags** (Optional): Add any tags you want for organization
2. **Permissions**: Leave as default (CloudFormation will create needed roles)
3. **Stack failure options**: Keep default (Rollback on failure)
4. Click **"Next"**

### Step 5: Review and Create

1. Review all your settings
2. Scroll to bottom and check the box: ☑️ **"I acknowledge that AWS CloudFormation might create IAM resources with custom names"**
3. Click **"Submit"**

### Step 6: Wait for Stack Creation

1. You'll see your stack status as **"CREATE_IN_PROGRESS"**
2. Click the refresh button periodically
3. **Wait 10-15 minutes** for completion
4. Status will change to **"CREATE_COMPLETE"** when done

**What's happening during this time:**
- Creating IAM roles and policies
- Setting up Elastic Beanstalk application
- Launching EC2 instances
- Configuring Load Balancer
- If CI/CD enabled: Setting up CodePipeline and CodeBuild

---

## Monitoring Deployment Progress

While waiting, you can watch the progress:

1. **CloudFormation Console**:
   - Click on your stack name
   - Go to **"Events"** tab
   - Watch resources being created in real-time

2. **Check Elastic Beanstalk**:
   - Open **Elastic Beanstalk** service in AWS Console
   - Click on your application name
   - Click on environment name
   - You'll see environment health and status

---

## Post-Deployment

### Step 1: Get Your Application URL

Once the stack status shows **"CREATE_COMPLETE"**:

1. In **CloudFormation Console**:
   - Click on your stack name (`flask-python-app-stack`)
   - Click on the **"Outputs"** tab
   - Find the row with **Key**: `ApplicationURL`
   - Copy the **Value** (this is your application URL)
   - Example: `flask-python-app-env.us-east-1.elasticbeanstalk.com`

2. Open your browser and go to:
   ```
   http://YOUR_APPLICATION_URL
   ```
   
3. You should see your Flask application with the message!

### Step 2: Verify Everything is Working

1. **Check Elastic Beanstalk Health**:
   - Go to **Elastic Beanstalk** service
   - Click on your application
   - Click on your environment name
   - Health should show **"Ok"** (green)

2. **Check CI/CD Pipeline** (if you provided GitHub token):
   - Go to **CodePipeline** service
   - You should see `flask-python-app-pipeline`
   - Click on it to see the pipeline stages
   - All stages (Source → Build → Deploy) should show **"Succeeded"** (green)

### Step 3: View Resources Created

Your CloudFormation stack created:
- ✅ S3 bucket for artifacts (you already had this)
- ✅ IAM roles (3 roles: Service, EC2, CodeBuild, CodePipeline)
- ✅ Elastic Beanstalk Application
- ✅ Elastic Beanstalk Environment (with EC2 instances and Load Balancer)
- ✅ CodeBuild Project (if CI/CD enabled)
- ✅ CodePipeline (if CI/CD enabled)

---

## Updating the Application

### Option A: Manual Update (No CI/CD)

If you left GitHub token empty:

1. Make changes to your application code
2. Create new ZIP file with updated code
3. Go to **S3** → Your bucket → Upload the new ZIP
4. Go to **Elastic Beanstalk** → Your application → Your environment
5. Click **"Upload and deploy"**
6. Choose your new ZIP file
7. Enter a version label (e.g., "v2")
8. Click **"Deploy"**

### Option B: Automatic Update (With CI/CD)

If you provided GitHub token:

1. Make changes to your code
2. Commit and push to GitHub:
   ```bash
   git add .
   git commit -m "Update application"
   git push origin main
   ```
3. Go to **CodePipeline** in AWS Console
4. Watch your pipeline automatically run and deploy!

---

## Cleanup (Delete All Resources)

To delete everything and stop incurring charges:

1. Go to **CloudFormation** service
2. Select your stack (`flask-python-app-stack`)
3. Click **"Delete"**
4. Click **"Delete stack"** to confirm
5. Wait 5-10 minutes for deletion to complete

**Then manually delete the S3 bucket:**
1. Go to **S3** service
2. Select your bucket (`flask-python-app-artifacts-YOUR_ACCOUNT_ID`)
3. Click **"Empty"** 
4. Confirm by typing "permanently delete"
5. After it's empty, click **"Delete"**
6. Confirm deletion

✅ All resources cleaned up!

---

## Troubleshooting

### Stack Creation Failed

**How to check:**
1. Go to **CloudFormation** → Your stack
2. Look at the **Status** - if it shows **"CREATE_FAILED"** or **"ROLLBACK_COMPLETE"**
3. Click the **"Events"** tab
4. Look for entries with **Status** = "CREATE_FAILED" (red)
5. Read the **Status reason** column for the error message

**Common Issues:**

| Issue | Solution |
|-------|----------|
| **"flask-app.zip does not exist"** | Make sure you uploaded `flask-app.zip` to your S3 bucket |
| **"Access Denied" or permission errors** | Your AWS user needs Administrator access or specific IAM permissions |
| **"Bucket already exists"** | Someone else has that bucket name. Use a unique name with your account ID |
| **"Solution stack not available"** | The Python version may not be available in your region. Check Elastic Beanstalk console for available platforms |

### Environment Not Healthy

**How to check:**
1. Go to **Elastic Beanstalk** service
2. Click on your environment
3. Look at the health status

**If health is not "Ok" (green):**
1. Click on **"Causes"** to see what's wrong
2. Click on **"Logs"** → **"Request Logs"** → **"Last 100 Lines"**
3. Click **"Download"** and review the logs

**Common fixes:**
- Wait 5-10 more minutes (deployment takes time)
- Check if your Python code has errors
- Verify `requirements.txt` has correct package versions

### Application Not Loading in Browser

**Checklist:**
- [ ] Stack status is **"CREATE_COMPLETE"** (green)
- [ ] Environment health is **"Ok"** (green)
- [ ] You're using `http://` (not `https://`)
- [ ] URL is correct (check Outputs tab)
- [ ] Wait a few more minutes for load balancer to be ready

### CI/CD Pipeline Issues

**Check pipeline status:**
1. Go to **CodePipeline** service
2. Click on `flask-python-app-pipeline`
3. Look at each stage (Source, Build, Deploy)

**If Build stage fails:**
1. Click **"Details"** on the failed Build stage
2. It will open CodeBuild
3. Click on **"Build logs"** to see what failed
4. Usually it's a Python dependency issue - check `requirements.txt`

**If Deploy stage fails:**
1. Check Elastic Beanstalk environment health
2. Review Elastic Beanstalk logs

### Need More Help?

1. **Check CloudFormation Events**: Shows exactly what's being created/deleted
2. **Check Elastic Beanstalk Logs**: Shows application errors
3. **Check CodeBuild Logs**: Shows build errors (if using CI/CD)
4. **AWS Service Health**: Check if AWS services are experiencing issues in your region

---

## Important Notes

### 💰 Costs
This deployment uses AWS resources that incur costs:
- **EC2 instances** (~$0.01/hour for t3.micro - Free Tier eligible)
- **Application Load Balancer** (~$0.025/hour)
- **S3 storage** (~$0.023/GB/month)
- **Data transfer** (varies)

**Estimated cost**: $20-30/month (if not on free tier)

**Free Tier**: 750 hours/month of t3.micro for 12 months

### 🔒 Security Notes
- Application uses HTTP (port 80) - not encrypted
- For production, configure HTTPS with SSL certificate
- All S3 buckets are private (good!)
- IAM roles follow least privilege principle

### 📝 Two Deployment Modes

**Without GitHub Token (Manual):**
- ✅ Simple setup
- ✅ Full control over deployments
- ❌ Manual updates required

**With GitHub Token (CI/CD):**
- ✅ Automatic deployments on every push
- ✅ Full pipeline visibility
- ❌ More complex (but automated!)

---

## Summary - Quick Reference

### Prerequisites Checklist
- [ ] AWS account with admin access
- [ ] AWS Account ID noted down
- [ ] ZIP file of application created (`flask-app.zip`)
- [ ] S3 bucket created: `flask-python-app-artifacts-YOUR_ACCOUNT_ID`
- [ ] ZIP file uploaded to S3 bucket
- [ ] GitHub token (optional, for CI/CD)

### Deployment Steps
1. Open CloudFormation Console
2. Create Stack → Upload `cloudformation.yml`
3. Fill in parameters (leave GitHub token empty if no CI/CD)
4. Acknowledge IAM resource creation
5. Submit and wait 10-15 minutes
6. Get URL from Outputs tab
7. Open URL in browser

### After Deployment
- URL from CloudFormation → Outputs tab
- Monitor health in Elastic Beanstalk console
- View pipeline in CodePipeline (if CI/CD enabled)

---

## Additional Resources

- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [AWS Free Tier Details](https://aws.amazon.com/free/)
- [Flask Documentation](https://flask.palletsprojects.com/)

---

**🎉 You're all set! Enjoy your deployed Flask application on AWS!**
