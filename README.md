# 🌐 AWS Static Website Infrastructure

> Serverless static website infrastructure provisioned as code using AWS CloudFormation.  
> Demonstrates S3, Route 53, CloudWatch and IAM integration following least privilege principles.

---

## 📐 Architecture

```
                        ┌─────────────────────────────────────────┐
                        │              AWS Cloud                   │
                        │                                          │
  User Browser          │   Route 53          S3 Bucket           │
  ─────────────  ──────►│   (DNS)      ──────► (Static Hosting)   │
                        │                          │               │
                        │                     CloudWatch           │
                        │                     (Monitoring)         │
                        │                          │               │
                        │                      SNS Topic           │
                        │                   (Alerts → Email)       │
                        └─────────────────────────────────────────┘
```

**Services used:**

| Service | Purpose |
|---|---|
| **S3** | Stores and serves static website files |
| **Route 53** | DNS management and domain routing |
| **CloudWatch** | Monitors 4xx errors and bucket metrics |
| **SNS** | Email notifications triggered by alarms |
| **IAM** | Deployment role with least privilege permissions |
| **CloudFormation** | Provisions all infrastructure as code |

---

## 🗂️ Repository Structure

```
aws-static-site/
├── template.yaml          # CloudFormation template (full infrastructure)
├── website/
│   ├── index.html         # Main page
│   └── error.html         # 404 error page
├── scripts/
│   └── deploy.sh          # Automated deploy script
└── README.md
```

---

## ⚙️ Prerequisites

- AWS account with CloudFormation, S3, Route 53 and IAM permissions
- AWS CLI v2 configured (`aws configure`)
- Domain registered in Route 53 or externally
- MFA enabled on the AWS account (security best practice)

---

## 🚀 Deployment — Step by Step

### 1. Clone the repository

```bash
git clone https://github.com/rugerious/aws-static-site.git
cd aws-static-site
```

### 2. Deploy the CloudFormation stack

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name static-website \
  --parameter-overrides \
      DomainName=tharoger.com \
      AlarmEmail=your@email.com \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### 3. Upload website files to S3

```bash
aws s3 sync ./website/ s3://$(aws cloudformation describe-stacks \
  --stack-name static-website \
  --query "Stacks[0].Outputs[?OutputKey=='BucketName'].OutputValue" \
  --output text)
```

### 4. Confirm SNS email subscription

After deployment, check your inbox and confirm the SNS subscription to start receiving CloudWatch alerts.

### 5. Verify the deployment

```bash
# View stack outputs (site URL, bucket name, etc.)
aws cloudformation describe-stacks \
  --stack-name static-website \
  --query "Stacks[0].Outputs"
```

---

## 🔒 Security Decisions

**Least Privilege IAM**  
The IAM Role is scoped to only the actions strictly required on the bucket (`s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`, `s3:ListBucket`). No access to other services or buckets.

**MFA on root account**  
The AWS account uses MFA on both the root user and the IAM Admin user, following AWS security best practices.

**Explicit Bucket Policy**  
Public access to the bucket is granted only through an explicit Bucket Policy with `s3:GetObject`, scoped to objects within this project's bucket only.

---

## 📊 Monitoring

The stack automatically creates a CloudWatch alarm that:

- Monitors **4xx errors** on the S3 bucket
- Evaluates every **5 minutes**
- Sends an **email alert via SNS** if more than **10 errors** occur in a period

To view metrics in the console:
```
CloudWatch → Metrics → S3 → BucketName, FilterId
```

---

## 🧹 Cleanup (Avoid Ongoing Costs)

To delete all resources created by this stack:

```bash
# 1. Empty the bucket first (required before stack deletion)
aws s3 rm s3://[bucket-name] --recursive

# 2. Delete the full stack
aws cloudformation delete-stack --stack-name static-website

# 3. Confirm deletion
aws cloudformation wait stack-delete-complete --stack-name static-website
```

---

## 💰 Cost Estimate

| Service | Estimated Cost |
|---|---|
| S3 (< 1 GB, < 10k requests/month) | ~$0.00 (Free Tier) |
| Route 53 Hosted Zone | ~$0.50/month |
| CloudWatch (basic metrics) | $0.00 (Free Tier) |
| SNS (< 1000 emails/month) | $0.00 (Free Tier) |
| **Total estimate** | **~$0.50/month** |

---

## 📚 Concepts Demonstrated

- **Infrastructure as Code (IaC)** — full infrastructure defined in CloudFormation YAML, version-controlled in Git
- **Least Privilege** — IAM Role scoped to minimum required permissions
- **Observability** — CloudWatch Alarm with automated notification pipeline
- **DNS Management** — Route 53 Hosted Zone and A Record with S3 Alias target
- **Idempotency** — template can be re-applied without side effects

---

## 🔗 References

- [AWS S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [CloudFormation User Guide](https://docs.aws.amazon.com/cloudformation/)
- [Route 53 Developer Guide](https://docs.aws.amazon.com/route53/)
- [CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

---

*Project developed as part of AWS Solutions Architect Associate (SAA-C03) exam preparation*
