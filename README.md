# 🚀 Terraform Practice Project - AWS Infrastructure with LocalStack

<div align="center">

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)

**DevQ-Inspired AWS Infrastructure Practice with Terraform & LocalStack**

[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![LocalStack](https://img.shields.io/badge/LocalStack-FF6B6B?style=flat&logo=docker&logoColor=white)](https://localstack.cloud/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)](https://github.com/features/actions)

</div>

---

## 📋 Project Overview

**Terraform Practice Project** là một dự án học tập và thực hành Terraform để triển khai infrastructure trên AWS. Dự án sử dụng **LocalStack** để mô phỏng các dịch vụ AWS locally, giúp học và test miễn phí mà không cần tài khoản AWS thực.

### 🎯 Main Objectives

- **Học Terraform** thông qua thực hành triển khai AWS infrastructure
- **LocalStack Integration** để test miễn phí các dịch vụ AWS locally
- **Public Access** cho collaboration và review
- **CI/CD Pipeline** với GitHub Actions cho automation
- **Remote State Management** cho team collaboration

---

## 🏗️ Architecture Overview

Minimal AWS stack inspired by DevQ architecture:

- **API Gateway + Lambda**: Public HTTP endpoint cho health checks và mock data retrieval
- **DynamoDB**: Lưu trữ simple review items
- **S3 Static Website**: Host documentation và artifacts
- **CloudWatch**: Logs cho Lambda và API Gateway
- **IAM Roles**: Least privilege cho Lambda và API Gateway
- **Remote State**: S3 + DynamoDB hoặc Terraform Cloud cho collaboration

---

## 🚀 Key Features

### 🌐 Public Access
- [x] API endpoint: `GET /health` và `GET /reviews/{id}`
- [x] Static site: Hosted trên S3 với public read access
- [x] Public GitHub repository với documentation

### 💻 Local Development
- [x] LocalStack để simulate AWS services
- [x] Docker Compose cho easy setup
- [x] Cost-free testing environment

### 🤝 Collaboration
- [x] Public repository với README, architecture diagram
- [x] Issues và Pull Requests enabled
- [x] Contribution guidelines

### 🔄 CI/CD
- [x] GitHub Actions cho `terraform fmt`, `validate`, và `plan`
- [x] OIDC-based AWS authentication cho secure automation
- [x] Automated checks trên PRs

### 🔒 Security
- [x] No secrets in repo
- [x] GitHub Secrets và AWS Secrets Manager
- [x] IAM roles với least privilege

---

## 📁 Repository Structure

```
terraform-practice/
├── README.md
├── .github/
│   └── workflows/
│       └── terraform-plan.yml
├── envs/
│   └── dev/
│       ├── backend.tf
│       ├── provider.tf
│       └── terraform.tfvars
├── modules/
│   ├── api/              # API Gateway + Lambda
│   ├── storage/          # S3 buckets
│   ├── database/         # DynamoDB tables
│   └── observability/   # CloudWatch logs
├── stacks/
│   └── main.tf
├── localstack/
│   └── docker-compose.yml
└── lambda/
    └── hello-devq/
        └── handler.py
```

---

## 🛠️ Tech Stack

### Infrastructure as Code
- **Terraform**: Infrastructure provisioning và management
- **Terraform Cloud/S3**: Remote state management
- **DynamoDB**: State locking

### AWS Services
- **API Gateway**: REST API endpoints
- **Lambda**: Serverless functions
- **DynamoDB**: NoSQL database
- **S3**: Static website hosting và storage
- **CloudWatch**: Logging và monitoring
- **IAM**: Access control

### Development Tools
- **LocalStack**: Local AWS service emulation
- **Docker**: Containerization
- **GitHub Actions**: CI/CD automation
- **Python**: Lambda functions

---

## 📦 Installation and Setup

> 📚 **Bắt đầu từ đây?** Xem [BUILD_GUIDE.md](./BUILD_GUIDE.md) để có hướng dẫn chi tiết từng bước xây dựng project!

### Prerequisites

**Required (cho Local Development):**
- **Docker** & **Docker Compose** - Để chạy LocalStack
- **Terraform** >= 1.0 - Infrastructure as Code tool
- **Git** - Version control

**Optional (cho AWS Production Deployment):**
- **AWS CLI** - Nếu muốn deploy lên AWS thực
- **Python 3.x** - Nếu muốn develop/test Lambda functions locally

> 💡 **Lưu ý**: LocalStack chạy qua Docker, **KHÔNG cần cài LocalStack riêng**. Chỉ cần Docker là đủ!

### Cài đặt Prerequisites

#### 1. Docker & Docker Compose

**Windows:**
- Download [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Install và khởi động Docker Desktop

**Kiểm tra cài đặt:**
```bash
docker --version
docker-compose --version
```

#### 2. Terraform

**Windows:**
- Download từ [terraform.io/downloads](https://www.terraform.io/downloads)
- Hoặc dùng Chocolatey: `choco install terraform`

**Kiểm tra cài đặt:**
```bash
terraform --version
```

#### 3. Git (nếu chưa có)

**Windows:**
- Download từ [git-scm.com](https://git-scm.com/download/win)
- Hoặc dùng Chocolatey: `choco install git`

**Kiểm tra cài đặt:**
```bash
git --version
```

### 1. Clone Repository

```bash
git clone https://github.com/your-username/terraform-practice-project.git
cd terraform-practice-project
```

### 2. Setup LocalStack (chạy qua Docker)

LocalStack được chạy qua Docker, không cần cài đặt riêng:

```bash
cd localstack
docker-compose up -d
```

**Kiểm tra LocalStack đã chạy:**
```bash
docker ps
# Bạn sẽ thấy container localstack đang chạy
```

**LocalStack sẽ expose các services tại:**
- API Gateway: `http://localhost:4566`
- Lambda: `http://localhost:4566`
- DynamoDB: `http://localhost:4566`
- S3: `http://localhost:4566`
- CloudWatch: `http://localhost:4566`

### 3. Configure Environment

```bash
cd envs/dev
cp terraform.tfvars.example terraform.tfvars
```

Edit `terraform.tfvars`:

```hcl
# LocalStack Configuration
endpoints = {
  apigateway     = "http://localhost:4566"
  lambda         = "http://localhost:4566"
  dynamodb       = "http://localhost:4566"
  s3             = "http://localhost:4566"
  cloudwatch     = "http://localhost:4566"
  iam            = "http://localhost:4566"
}

# AWS Configuration (for real deployment)
# aws_region = "us-east-1"
# aws_profile = "default"
```

### 4. Initialize Terraform

```bash
terraform init
```

### 5. Plan & Apply

```bash
# Review changes
terraform plan

# Apply infrastructure
terraform apply
```

### 6. Test Endpoints

- **Health Check**: `http://localhost:4566/restapis/{api-id}/local/_user_request_/health`
- **Get Review**: `http://localhost:4566/restapis/{api-id}/local/_user_request_/reviews/{id}`

---

## 🚀 Deployment Options

### Local Development (LocalStack)

```bash
# Start LocalStack
docker-compose up -d

# Deploy to LocalStack
cd envs/dev
terraform apply
```

### AWS Production

```bash
# Configure AWS credentials
aws configure

# Update backend.tf for S3 remote state
# Update terraform.tfvars with real AWS values

# Deploy to AWS
terraform apply
```

### Remote State (Collaboration)

- **Option 1**: S3 + DynamoDB backend
- **Option 2**: Terraform Cloud

---

## 🧪 Testing

### Terraform Validation

```bash
# Format code
terraform fmt -recursive

# Validate configuration
terraform validate

# Check plan
terraform plan
```

### CI/CD Checks

GitHub Actions automatically runs:
- `terraform fmt -check`
- `terraform validate`
- `terraform plan`

---

## 📚 API Endpoints

### Health Check

```http
GET /health
```

Response:
```json
{
  "status": "healthy",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### Get Review

```http
GET /reviews/{id}
```

Response:
```json
{
  "id": "review-123",
  "content": "Review content here"
}
```

---

## 🤝 Contributing

### Development Workflow

1. Fork repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Create Pull Request

### Code Standards

- **Terraform**: Follow HashiCorp best practices
- **Formatting**: Use `terraform fmt`
- **Validation**: All changes must pass `terraform validate`
- **Documentation**: Update README for significant changes

---

## 🔒 Security Best Practices

- ✅ No secrets committed to repository
- ✅ Use GitHub Secrets for sensitive data
- ✅ IAM roles với least privilege principle
- ✅ Remote state encryption
- ✅ Regular security audits

---

## 📄 License

Dự án này được phát hành dưới [MIT License](LICENSE).

---

## 👥 Team

- **Infrastructure Engineer**: [Your Name]
- **DevOps**: Terraform + GitHub Actions
- **Architecture**: AWS Services + LocalStack

---

## 📞 Support & Contact

- **Issues**: [GitHub Issues](https://github.com/your-username/terraform-practice-project/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-username/terraform-practice-project/discussions)
- **Documentation**: See `docs/` folder

---

## 🎯 Roadmap

### Phase 1 (Current)
- [x] Project structure setup
- [x] LocalStack integration
- [x] Basic AWS resources (Lambda, API Gateway, DynamoDB, S3)
- [x] CI/CD pipeline

### Phase 2
- [ ] Advanced Lambda functions
- [ ] CloudWatch dashboards
- [ ] Multi-environment support (dev, staging, prod)
- [ ] Documentation improvements

### Phase 3
- [ ] Advanced monitoring và alerting
- [ ] Cost optimization
- [ ] Security hardening
- [ ] Performance testing

---

<div align="center">

**🚀 Terraform Practice Project**

*Learn Infrastructure as Code with AWS and LocalStack*

[![Made with ❤️](https://img.shields.io/badge/Made%20with-❤️-red?style=flat)](https://github.com/your-username/terraform-practice-project)
[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)

</div>
