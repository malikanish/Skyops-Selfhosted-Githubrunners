# 🚀 On-Demand GitHub Self-Hosted Runners with AWS

This repository contains the infrastructure and scripts to build a fully automated, on-demand GitHub self-hosted runner system using **AWS Lambda, API Gateway, Auto Scaling Group, Launch Templates, and EC2**.

Whenever a GitHub workflow job is triggered, it scales up an EC2 instance dynamically, registers it as a self-hosted runner, executes the job, and then terminates the instance to save cost.

👉 This approach helps achieve **up to 70–80% cost savings** compared to always-on runners.

---

## 📺 Video Walkthrough
🎥 Full tutorial is available on my channel: [SkyOpsTech](https://www.youtube.com/@SkyOpsTech)

---

## ⚙️ Architecture Flow
```
Developer pushes code → triggers GitHub Actions workflow  
Webhook → API Gateway → Lambda function  
Lambda updates Auto Scaling Group (ASG) → launches EC2 instance  
EC2 runs userdata.sh → installs & registers GitHub runner  
Runner executes job → shuts down after completion  
```

---

## 📂 Repository Structure
```
├── userdata.sh          # EC2 User Data script (GitHub runner setup)  
├── lambda_function.py   # Lambda scaling function (API Gateway handler)  
└── README.md            # Documentation  
```

---

## 🛠️ Setup Instructions

### 1️⃣ Launch Template
- **AMI**: Ubuntu (latest LTS)  
- **Instance type**: `t2.micro` (or higher for heavy workloads)  
- **Security group**: allow ports `22 (SSH)`, `80 (HTTP)`, `443 (HTTPS)`  
- **IAM role**: Attach `AmazonSSMManagedInstanceCore`  
- **User Data**: Add `userdata.sh` script  

---

### 2️⃣ IAM Roles
- **EC2 Role** → `AmazonSSMManagedInstanceCore`  
- **Lambda Role** → attach policies:  
  - `AmazonSSMFullAccess`  
  - `AutoScalingFullAccess`  
  - `AWSLambda_FullAccess`  

---

### 3️⃣ User Data Script
Located in `userdata.sh`.  

It will:  
- Install dependencies  
- Fetch GitHub PAT from **SSM Parameter Store**  
- Register EC2 instance as GitHub self-hosted runner  
- Start runner service  
- Shut down instance after **20 minutes**  

---

### ⚡ Before Using – Update Parameters
```bash
GH_OWNER="your-github-username"  
GH_REPO="your-repo-name"  
REGION="your-aws-region"  
SSM_PARAM_NAME="/github/selfhosted/pat"
```

---

### 4️⃣ Store GitHub Token in SSM
```bash
aws ssm put-parameter \
--name "/github/selfhosted/pat" \
--value "your-github-pat-token" \
--type SecureString \
--region us-east-1
```

---

### 5️⃣ Auto Scaling Group (ASG)
Create ASG using launch template. Example capacity:  

```
Min: 0  
Desired: 0  
Max: 3  
```

👉 Lambda will update ASG capacity based on webhook events.  

---

### 6️⃣ Lambda Function
Code: `lambda_function.py`  

**Environment Variables (set in Lambda Console):**  
```bash
ASG_NAME=github-runner-asg  
WEBHOOK_SECRET=your_webhook_secret  
GH_OWNER=your-github-username  
GH_REPO=your-repo-name  
SSM_PARAM_NAME=/github/selfhosted/pat  
REGION=us-east-1
```

---

### 7️⃣ API Gateway
- Create **HTTP API**  
- **Integration**: Lambda function  
- **Route**: `POST /webhook`  
- **Deploy stage**: `dev`  
- Copy **Invoke URL**  

---

### 8️⃣ GitHub Webhook
Go to Repo → **Settings → Webhooks → Add Webhook**  

- **Payload URL**: API Gateway Invoke URL  
- **Content type**: `application/json`  
- **Secret**: same as `WEBHOOK_SECRET`  
- **Events**: Push (or customize)  

---

### 9️⃣ Full Workflow
```
Push code → GitHub workflow triggers  
API Gateway → Lambda → ASG → EC2  
EC2 runner executes job  
Instance terminates after completion  
```

✅ Zero idle cost, fully automated 🚀  

---

## 📊 Benefits
- ⚡ On-demand scaling  
- 💰 70–80% CI/CD cost savings  
- 🔒 Secure GitHub token handling (via SSM)  
- 🔄 Automated cleanup of idle instances  

---

## 🙌 Outro
This setup ensures **scalable workflows, zero idle costs, and optimized CI/CD infrastructure**.  

⭐ Don’t forget to check out the full video tutorial and drop a ⭐ on this repo if it helped you!  
