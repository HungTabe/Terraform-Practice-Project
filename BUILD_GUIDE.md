# 📚 Hướng Dẫn Xây Dựng Project - Step by Step

> 🎓 **Mục đích**: File này hướng dẫn bạn tự xây dựng project từ đầu. Bạn sẽ code theo từng bước, và tôi sẽ kiểm tra code của bạn như một gia sư.

---

## 🎯 Cách Sử Dụng Guide Này

1. **Đọc kỹ từng bước** trước khi bắt đầu code
2. **Tự code** theo hướng dẫn (không copy-paste code có sẵn)
3. **Tự kiểm tra** bằng checklist ở cuối mỗi bước
4. **Commit code** sau mỗi bước hoàn thành
5. **Gửi cho tôi kiểm tra** nếu cần hỗ trợ

---

## 📋 Tổng Quan Các Bước

- [ ] **Bước 1**: Setup Project Structure
- [ ] **Bước 2**: Setup LocalStack với Docker Compose
- [ ] **Bước 3**: Tạo Terraform Provider Configuration
- [ ] **Bước 4**: Tạo S3 Module
- [ ] **Bước 5**: Tạo DynamoDB Module
- [ ] **Bước 6**: Tạo Lambda Function
- [ ] **Bước 7**: Tạo API Gateway Module
- [ ] **Bước 8**: Tạo Main Stack
- [ ] **Bước 9**: Setup CI/CD với GitHub Actions
- [ ] **Bước 10**: Testing và Validation

---

## 🚀 Bước 1: Setup Project Structure

### Mục tiêu
Tạo cấu trúc thư mục cơ bản cho project.

### Hướng dẫn

1. **Tạo các thư mục chính:**
   ```
   terraform-practice/
   ├── .github/
   │   └── workflows/
   ├── envs/
   │   └── dev/
   ├── modules/
   │   ├── api/
   │   ├── storage/
   │   ├── database/
   │   └── observability/
   ├── stacks/
   ├── localstack/
   └── lambda/
       └── hello-devq/
   ```

2. **Tạo file `.gitignore`:**
   - Ignore các file Terraform state
   - Ignore `.tfvars` files (chứa sensitive data)
   - Ignore các file tạm

3. **Tạo file `.editorconfig` (optional):**
   - Để đảm bảo code style nhất quán

### Checklist
- [ ] Đã tạo đầy đủ các thư mục
- [ ] Đã tạo `.gitignore` với các pattern phù hợp
- [ ] Cấu trúc thư mục đúng như mô tả

### Gợi ý `.gitignore`
```
# Terraform
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
*.tfvars
*.tfvars.json

# LocalStack
.localstack/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db
```

---

## 🐳 Bước 2: Setup LocalStack với Docker Compose

### Mục tiêu
Tạo Docker Compose file để chạy LocalStack locally.

### Hướng dẫn

1. **Tạo file `localstack/docker-compose.yml`:**
   - Sử dụng image `localstack/localstack:latest`
   - Expose port `4566` (LocalStack default port)
   - Set environment variables:
     - `SERVICES`: Danh sách các AWS services cần (apigateway, lambda, dynamodb, s3, cloudwatch, iam)
     - `DEBUG`: `1` để enable debug mode
     - `DATA_DIR`: `/tmp/localstack/data`
   - Mount volume để persist data

2. **Tạo file `localstack/README.md` (optional):**
   - Hướng dẫn cách start/stop LocalStack

### Checklist
- [ ] File `docker-compose.yml` đã được tạo
- [ ] Có thể chạy `docker-compose up -d` thành công
- [ ] LocalStack container đang chạy (kiểm tra bằng `docker ps`)
- [ ] Có thể truy cập LocalStack tại `http://localhost:4566`

### Gợi ý cấu trúc
```yaml
version: '3.8'

services:
  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"
    environment:
      - SERVICES=apigateway,lambda,dynamodb,s3,cloudwatch,iam
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
    volumes:
      - "./.localstack:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

---

## ⚙️ Bước 3: Tạo Terraform Provider Configuration

### Mục tiêu
Cấu hình Terraform để làm việc với LocalStack.

### Hướng dẫn

1. **Tạo file `envs/dev/provider.tf`:**
   - Configure AWS provider
   - Point đến LocalStack endpoints
   - Set region (ví dụ: `us-east-1`)
   - Disable SSL verification (vì LocalStack dùng HTTP)

2. **Tạo file `envs/dev/backend.tf`:**
   - Configure backend (có thể dùng local backend cho dev)
   - Hoặc comment out nếu chưa setup remote state

3. **Tạo file `envs/dev/terraform.tfvars.example`:**
   - Template file với các biến cần thiết
   - Không commit file `.tfvars` thực (đã có trong `.gitignore`)

4. **Tạo file `envs/dev/variables.tf`:**
   - Định nghĩa các biến sẽ dùng trong project

### Checklist
- [ ] File `provider.tf` đã cấu hình đúng LocalStack endpoints
- [ ] File `backend.tf` đã được tạo (có thể dùng local backend)
- [ ] File `terraform.tfvars.example` đã được tạo
- [ ] File `variables.tf` đã định nghĩa các biến cơ bản
- [ ] Chạy `terraform init` thành công

### Gợi ý cấu trúc `provider.tf`
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  access_key                  = "test"
  secret_key                  = "test"
  region                      = "us-east-1"
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true
  
  endpoints {
    apigateway     = "http://localhost:4566"
    lambda         = "http://localhost:4566"
    dynamodb       = "http://localhost:4566"
    s3             = "http://localhost:4566"
    cloudwatch     = "http://localhost:4566"
    iam            = "http://localhost:4566"
  }
}
```

---

## 📦 Bước 4: Tạo S3 Module

### Mục tiêu
Tạo Terraform module để quản lý S3 bucket cho static website.

### Hướng dẫn

1. **Tạo file `modules/storage/main.tf`:**
   - Tạo S3 bucket
   - Enable static website hosting
   - Set bucket policy để public read access

2. **Tạo file `modules/storage/variables.tf`:**
   - Định nghĩa các input variables:
     - `bucket_name`: Tên bucket
     - `enable_website`: Có enable website hosting không

3. **Tạo file `modules/storage/outputs.tf`:**
   - Output bucket name
   - Output website endpoint URL

4. **Tạo file `modules/storage/README.md`:**
   - Mô tả module và cách sử dụng

### Checklist
- [ ] Module có thể tạo S3 bucket thành công
- [ ] Static website hosting đã được enable
- [ ] Bucket policy cho phép public read
- [ ] Outputs đã được định nghĩa đúng

### Gợi ý
- S3 bucket name phải unique (có thể thêm random suffix)
- Nhớ set `force_destroy = true` cho dev environment
- Website endpoint format: `http://{bucket-name}.s3-website.localhost.localstack.cloud:4566`

---

## 🗄️ Bước 5: Tạo DynamoDB Module

### Mục tiêu
Tạo Terraform module để quản lý DynamoDB table.

### Hướng dẫn

1. **Tạo file `modules/database/main.tf`:**
   - Tạo DynamoDB table
   - Định nghĩa primary key (ví dụ: `id` là String)
   - Set billing mode là `PAY_PER_REQUEST` (cho dev)

2. **Tạo file `modules/database/variables.tf`:**
   - Định nghĩa:
     - `table_name`: Tên table
     - `hash_key`: Tên primary key

3. **Tạo file `modules/database/outputs.tf`:**
   - Output table name
   - Output table ARN

### Checklist
- [ ] DynamoDB table được tạo thành công
- [ ] Primary key đã được định nghĩa đúng
- [ ] Billing mode phù hợp cho dev environment

### Gợi ý
- Table name: ví dụ `reviews-table`
- Hash key: `id` (String)
- Có thể thêm tags để quản lý

---

## 🐍 Bước 6: Tạo Lambda Function

### Mục tiêu
Tạo Lambda function với Python để xử lý API requests.

### Hướng dẫn

1. **Tạo file `lambda/hello-devq/handler.py`:**
   - Tạo Lambda handler function
   - Xử lý các routes:
     - `GET /health`: Trả về health check
     - `GET /reviews/{id}`: Trả về review theo ID
   - Return JSON response với status code

2. **Tạo file `lambda/hello-devq/requirements.txt` (nếu cần):**
   - List các Python dependencies

3. **Tạo file `lambda/hello-devq/README.md`:**
   - Mô tả function và cách test locally

### Checklist
- [ ] Lambda handler function đã được tạo
- [ ] Có thể xử lý `/health` endpoint
- [ ] Có thể xử lý `/reviews/{id}` endpoint
- [ ] Response format đúng JSON

### Gợi ý cấu trúc handler
```python
import json

def handler(event, context):
    """
    Lambda handler function
    """
    path = event.get('path', '')
    http_method = event.get('httpMethod', '')
    
    # Health check endpoint
    if path == '/health' and http_method == 'GET':
        return {
            'statusCode': 200,
            'body': json.dumps({
                'status': 'healthy',
                'timestamp': '...'
            })
        }
    
    # Reviews endpoint
    if path.startswith('/reviews/') and http_method == 'GET':
        # Extract review ID from path
        # Return review data
        pass
    
    # Default 404
    return {
        'statusCode': 404,
        'body': json.dumps({'error': 'Not found'})
    }
```

---

## 🌐 Bước 7: Tạo API Gateway Module

### Mục tiêu
Tạo Terraform module để quản lý API Gateway và Lambda integration.

### Hướng dẫn

1. **Tạo file `modules/api/main.tf`:**
   - Tạo API Gateway REST API
   - Tạo Lambda function resource
   - Tạo Lambda permission để API Gateway invoke Lambda
   - Tạo API Gateway resources và methods:
     - `GET /health`
     - `GET /reviews/{id}`
   - Tạo API Gateway deployment
   - Tạo API Gateway stage

2. **Tạo file `modules/api/variables.tf`:**
   - Định nghĩa:
     - `lambda_function_name`: Tên Lambda function
     - `lambda_handler`: Handler path
     - `lambda_runtime`: Python runtime version
     - `lambda_source_path`: Path đến Lambda code

3. **Tạo file `modules/api/outputs.tf`:**
   - Output API Gateway URL
   - Output API Gateway ID

### Checklist
- [ ] API Gateway REST API đã được tạo
- [ ] Lambda function đã được tạo và package
- [ ] Lambda permission đã được set đúng
- [ ] API Gateway resources và methods đã được tạo
- [ ] API Gateway deployment và stage đã được tạo
- [ ] Có thể test API endpoints

### Gợi ý
- Lambda function cần được zip trước khi upload
- Sử dụng `archive_file` data source để zip Lambda code
- API Gateway stage name: `local` hoặc `dev`
- LocalStack API Gateway URL format: `http://localhost:4566/restapis/{api-id}/local/_user_request_`

---

## 🏗️ Bước 8: Tạo Main Stack

### Mục tiêu
Tạo main Terraform file để orchestrate tất cả các modules.

### Hướng dẫn

1. **Tạo file `stacks/main.tf`:**
   - Gọi các modules:
     - `modules/storage` cho S3
     - `modules/database` cho DynamoDB
     - `modules/api` cho API Gateway + Lambda
   - Pass các variables cần thiết
   - Tạo các resources dependencies nếu cần

2. **Tạo file `stacks/outputs.tf`:**
   - Output API Gateway URL
   - Output S3 website URL
   - Output DynamoDB table name

3. **Update file `envs/dev/main.tf` (hoặc tạo mới):**
   - Reference đến `stacks/main.tf`
   - Hoặc gọi trực tiếp các modules

### Checklist
- [ ] Tất cả modules đã được gọi đúng cách
- [ ] Variables đã được pass đầy đủ
- [ ] Dependencies giữa các resources đã được set đúng
- [ ] Outputs đã được định nghĩa
- [ ] Chạy `terraform plan` không có lỗi
- [ ] Chạy `terraform apply` thành công

### Gợi ý cấu trúc
```hcl
module "storage" {
  source = "../modules/storage"
  
  bucket_name   = var.bucket_name
  enable_website = true
}

module "database" {
  source = "../modules/database"
  
  table_name = var.dynamodb_table_name
  hash_key   = "id"
}

module "api" {
  source = "../modules/api"
  
  lambda_function_name = var.lambda_function_name
  lambda_handler       = "handler.handler"
  lambda_runtime       = "python3.9"
  lambda_source_path   = "../lambda/hello-devq"
}
```

---

## 🔄 Bước 9: Setup CI/CD với GitHub Actions

### Mục tiêu
Tạo GitHub Actions workflow để tự động validate và plan Terraform.

### Hướng dẫn

1. **Tạo file `.github/workflows/terraform-plan.yml`:**
   - Trigger trên push và pull request
   - Setup job với:
     - Checkout code
     - Setup Terraform
     - Run `terraform fmt -check`
     - Run `terraform init`
     - Run `terraform validate`
     - Run `terraform plan` (không apply)

2. **Tạo file `.github/workflows/terraform-apply.yml` (optional):**
   - Chỉ chạy khi merge vào main branch
   - Tự động apply (cẩn thận với production!)

### Checklist
- [ ] Workflow file đã được tạo
- [ ] Workflow trigger đúng events
- [ ] Các bước validation đã được thêm
- [ ] Workflow chạy thành công trên GitHub

### Gợi ý
```yaml
name: Terraform Plan

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
      
      - name: Terraform Format Check
        run: terraform fmt -check -recursive
      
      - name: Terraform Init
        run: terraform init
        working-directory: ./envs/dev
      
      - name: Terraform Validate
        run: terraform validate
        working-directory: ./envs/dev
      
      - name: Terraform Plan
        run: terraform plan
        working-directory: ./envs/dev
```

---

## ✅ Bước 10: Testing và Validation

### Mục tiêu
Test toàn bộ infrastructure và đảm bảo mọi thứ hoạt động đúng.

### Hướng dẫn

1. **Start LocalStack:**
   ```bash
   cd localstack
   docker-compose up -d
   ```

2. **Initialize và Apply Terraform:**
   ```bash
   cd envs/dev
   terraform init
   terraform plan
   terraform apply
   ```

3. **Test API Endpoints:**
   - Health check: `GET http://localhost:4566/restapis/{api-id}/local/_user_request_/health`
   - Get review: `GET http://localhost:4566/restapis/{api-id}/local/_user_request_/reviews/{id}`

4. **Test S3:**
   - Upload file lên S3 bucket
   - Truy cập static website URL

5. **Test DynamoDB:**
   - Put item vào table
   - Get item từ table

6. **Cleanup:**
   ```bash
   terraform destroy
   docker-compose down
   ```

### Checklist
- [ ] LocalStack đang chạy
- [ ] Terraform apply thành công
- [ ] API endpoints hoạt động đúng
- [ ] S3 bucket có thể truy cập
- [ ] DynamoDB table hoạt động
- [ ] Có thể cleanup thành công

---

## 📝 Lưu Ý Quan Trọng

### Best Practices

1. **Luôn chạy `terraform fmt`** trước khi commit
2. **Luôn chạy `terraform validate`** trước khi apply
3. **Review `terraform plan`** cẩn thận trước khi apply
4. **Commit thường xuyên** sau mỗi bước hoàn thành
5. **Viết commit message rõ ràng** (ví dụ: "Step 1: Setup project structure")

### Khi Gặp Lỗi

1. **Đọc error message cẩn thận**
2. **Check Terraform documentation**
3. **Check LocalStack logs**: `docker logs localstack`
4. **Hỏi tôi** nếu cần hỗ trợ!

### Resources Hữu Ích

- [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [LocalStack Documentation](https://docs.localstack.cloud/)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)

---

## 🎓 Cách Làm Việc Với Tôi (Gia Sư)

1. **Sau mỗi bước:**
   - Commit code của bạn
   - Gửi cho tôi xem code
   - Tôi sẽ review và đưa feedback

2. **Khi gặp lỗi:**
   - Mô tả lỗi bạn gặp phải
   - Gửi error message
   - Tôi sẽ hướng dẫn cách fix

3. **Khi cần giải thích:**
   - Hỏi bất kỳ câu hỏi nào
   - Tôi sẽ giải thích chi tiết

4. **Khi hoàn thành:**
   - Tôi sẽ review toàn bộ project
   - Đưa ra suggestions để improve

---

## 🎯 Checklist Tổng Thể

Sau khi hoàn thành tất cả các bước, bạn nên có:

- [ ] Project structure đầy đủ
- [ ] LocalStack chạy được
- [ ] S3 bucket được tạo và có thể truy cập
- [ ] DynamoDB table được tạo
- [ ] Lambda function hoạt động
- [ ] API Gateway có 2 endpoints hoạt động
- [ ] CI/CD pipeline chạy thành công
- [ ] Tất cả tests pass
- [ ] README.md đã được update
- [ ] Code đã được format và validate

---

**Chúc bạn học tốt! 🚀**

Nếu có bất kỳ câu hỏi nào, đừng ngại hỏi tôi nhé!

