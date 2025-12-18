# Sports API Management System

## **Project Overview**
This project demonstrates building a containerized API management system for querying sports data. It leverages **Amazon ECS (Fargate)** for running containers, **Amazon API Gateway** for exposing REST endpoints, and an external **Sports API** for real-time sports data. The project showcases advanced cloud computing practices, including API management, container orchestration, and secure AWS integrations.

---

## **Features**
- Exposes a REST API for querying real-time sports data
- Runs a containerized backend using Amazon ECS with Fargate
- Scalable and serverless architecture
- API management and routing using Amazon API Gateway
 
---

## **Prerequisites**
- **Sports API Key**: Sign up for a free account and subscription & obtain your API Key at serpapi.com
- **AWS Account**: Create an AWS Account & have basic understanding of ECS, API Gateway, Docker & Python

### **AWS CLI Installation and Configuration**
Install AWS CLI to programmatically interact with AWS services:

**Installation:**
```bash
# Linux/macOS
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Windows (PowerShell)
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

# Verify installation
aws --version
```

**Configuration:**
```bash
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Default region: us-east-1
# Default output format: json
```

### **Python Dependencies Installation**
Set up Python environment and install required packages:

```bash
# Create virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install required dependencies
pip install Flask==2.2.5 requests==2.31.0

# Verify installation
python -c "import flask, requests; print('Dependencies installed successfully')"
```

### **Docker Installation**
Install Docker CLI and Desktop for containerization:

**Docker Desktop:**
- **Windows/macOS**: Download from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- **Linux**: 
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install docker.io docker-compose

# For WSL2 users:
sudo service docker start
# Add to ~/.bashrc for auto-start: echo 'sudo service docker start' >> ~/.bashrc

# For native Linux:
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER
```

**Verify Installation:**
```bash
docker --version
docker run hello-world
```

---

## **Technical Architecture**
![Brown Minimalist Lifestyle Daily Vlog YouTube Thumbnail (2)](https://github.com/user-attachments/assets/32e49fe6-df16-40cb-b262-af1478cf01d5)

---

## **Technologies**
- **Cloud Provider**: AWS
- **Core Services**: Amazon ECS (Fargate), API Gateway, CloudWatch
- **Programming Language**: Python 3.x
- **Containerization**: Docker
- **IAM Security**: Custom least privilege policies for ECS task execution and API Gateway

---

## **Project Structure**

```bash
containerized-sports-api/
├── app.py # Flask application for querying NFL schedule via SerpAPI
├── Dockerfile # Dockerfile to containerize the Flask app
├── requirements.txt # Python dependencies (Flask, requests)
├── .gitignore # Git ignore file
└── README.md # Project documentation
```

---

## **Setup Instructions**

### **Clone the Repository**
```bash
git clone https://github.com/gus-hub-tech/containerized-sports-api.git
cd containerized-sports-api
```
### **Get Your AWS Account ID**
```bash
# Get your AWS Account ID (save this for later steps)
aws sts get-caller-identity --query Account --output text
```

### **Create ECR Repository**
```bash
aws ecr create-repository --repository-name sports-api --region us-east-1
```

### **Build and Push Docker Image**
```bash
Get your AWS Account ID
aws sts get-caller-identity --query Account --output text

Create ECR Repository:
aws ecr create-repository --repository-name sports-api --region us-east-1


# Replace <AWS_ACCOUNT_ID> with your actual AWS Account ID from previous step
# Authenticate to ECR (expires after 12 hours)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

# Build Docker image
docker build --platform linux/amd64 -t sports-api .

# Tag image for ECR
docker tag sports-api:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest

# Push to ECR
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
```

### **Set Up ECS Cluster with Fargate**
1. **Create an ECS Cluster:**
- Navigate to [AWS ECS Console](https://console.aws.amazon.com/ecs/)
- Click **Clusters** in left sidebar → **Create Cluster**
- Cluster name: `sports-api-cluster`
- Infrastructure: Select **AWS Fargate (serverless)**
- Click **Create**

2. **Create a Task Definition:**
- Click **Task Definitions** → **Create new task definition**
- Task definition family: `sports-api-task`
- Launch type: **AWS Fargate**
- Operating system: **Linux/X86_64**
- CPU: **0.25 vCPU**, Memory: **0.5 GB**
- **Add Container:**
  - Container name: `sports-api-container`
  - Image URI: `<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest`
  - Port mappings: Container port `8080`, Protocol `TCP`
- **Environment Variables:**
  - Key: `SPORTS_API_KEY`
  - Value: `<YOUR_SERPAPI_API_KEY>` (get from [serpapi.com](https://serpapi.com))
- Click **Create**
3. Run the Service with an ALB
- Go to Clusters → Select Cluster → Service → Create.
- Capacity provider: Fargate
- Select Deployment configuration family (sports-api-task)
- Name your service (sports-api-service)
- Desired tasks: 2
- Networking: Create new security group
- Networking Configuration:
  - Type: All TCP
  - Source: Anywhere
- Load Balancing: Select Application Load Balancer (ALB).
- ALB Configuration:
 - Create a new ALB:
 - Name: sports-api-alb
 - Target Group health check path: "/sports"
 - Create service
4. Test the ALB:
- After deploying the ECS service, note the DNS name of the ALB (e.g., sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com)
- Confirm the API is accessible by visiting the ALB DNS name in your browser and adding /sports at end (e.g, http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports)

### **Configure API Gateway**
1. Create a New REST API:
- Go to API Gateway Console → Create API → REST API
- Name the API (e.g., Sports API Gateway)

2. Set Up Integration:
- Create a resource /sports
- Create a GET method
- Choose HTTP Proxy as the integration type
- Enter the DNS name of the ALB that includes "/sports" (e.g. http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports

3. Deploy the API:
- Deploy the API to a stage (e.g., prod)
- Note the endpoint URL

### **Test the System**
- Use curl or a browser to test:
```bash
curl https://<api-gateway-id>.execute-api.us-east-1.amazonaws.com/prod/sports
```

### **What We Learned**
Setting up a scalable, containerized application with ECS
Creating public APIs using API Gateway.

### **Future Enhancements**
Add caching for frequent API requests using Amazon ElastiCache
Add DynamoDB to store user-specific queries and preferences
Secure the API Gateway using an API key or IAM-based authentication
Implement CI/CD for automating container deployments

## License

This project is for educational and demonstration purposes.  
See individual files for additional licensing information if present.

---

*Feel free to expand, adapt, or integrate this template to fit your own applications!*


