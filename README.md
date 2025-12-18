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
- **Sports API Key**: Sign up for a free account at [serpapi.com](https://serpapi.com) and obtain your API key
- **AWS Account**: Create an AWS account with basic understanding of ECS, API Gateway, Docker & Python
- **AWS CLI**: Configured with appropriate permissions for ECS, ECR, and API Gateway
- **Docker**: Installed and running on your local machine

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

# Add user to docker group
sudo usermod -aG docker $USER
```

**Start Docker Daemon:**
```bash
# For native Linux:
sudo systemctl start docker
sudo systemctl enable docker

# For WSL2:
sudo service docker start

# Alternative methods if above don't work:
sudo dockerd &  # Run in background
# Or install Docker Desktop and start it manually
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
### **Build and Push Docker Image**

**Step 1: Get AWS Account ID**
```bash
aws sts get-caller-identity --query Account --output text
```

**Step 2: Create ECR Repository**
```bash
aws ecr create-repository --repository-name sports-api --region us-east-1
```

**Step 3: Authenticate with ECR**
```bash
# Replace <AWS_ACCOUNT_ID> with your actual AWS Account ID from step 1
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

**Troubleshooting Authentication:**
- If you get "no basic auth credential" error, run the ECR login command above
- If you get "permission denied" error, ensure Docker daemon is running:
  ```bash
  sudo systemctl start docker  # or sudo service docker start
  ```
- ECR authentication expires after 12 hours, re-run the login command if needed

**Step 4: Build and Push**
```bash
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
3. **Create and Run ECS Service with ALB:**
- Go to **Clusters** → Select your cluster → **Services** → **Create**
- **Service Configuration:**
  - Capacity provider: **Fargate**
  - Task definition family: **sports-api-task**
  - Service name: **sports-api-service**
  - Desired tasks: **2**
- **Networking:**
  - Create new security group
  - Type: **All TCP**
  - Source: **Anywhere (0.0.0.0/0)**
- **Load Balancing:**
  - Select **Application Load Balancer (ALB)**
  - Create new ALB: **sports-api-alb**
  - Target group health check path: **"/"**
- Click **Create Service**

4. **Test the ALB:**
- Note the ALB DNS name (e.g., `sports-api-alb-<random>.us-east-1.elb.amazonaws.com`)
- Test endpoints:
  ```bash
  # Health check
  curl http://sports-api-alb-<random>.us-east-1.elb.amazonaws.com/
  
  # Sports API
  curl http://sports-api-alb-<random>.us-east-1.elb.amazonaws.com/sports
  ```

### **Configure API Gateway**

1. **Create REST API:**
- Go to [API Gateway Console](https://console.aws.amazon.com/apigateway/)
- Click **Create API** → **REST API** → **Build**
- API name: **Sports API Gateway**
- Click **Create API**

2. **Set Up Integration:**
- Create resource: **Actions** → **Create Resource**
  - Resource name: **sports**
  - Resource path: **/sports**
- Create method: Select **/sports** → **Actions** → **Create Method** → **GET**
- **Integration Setup:**
  - Integration type: **HTTP Proxy**
  - HTTP method: **GET**
  - Endpoint URL: `http://sports-api-alb-<random>.us-east-1.elb.amazonaws.com/sports`
- Click **Save**

3. **Deploy API:**
- **Actions** → **Deploy API**
- Deployment stage: **New Stage**
- Stage name: **prod**
- Click **Deploy**
- Note the **Invoke URL**

### **Test the Complete System**

**Test API Gateway Endpoint:**
```bash
# Replace with your actual API Gateway URL
curl https://<api-gateway-id>.execute-api.us-east-1.amazonaws.com/prod/sports
```

**Expected Response:**
```json
{
  "message": "NFL schedule fetched successfully.",
  "games": [
    {
      "away_team": "Team A",
      "home_team": "Team B",
      "venue": "Stadium Name",
      "date": "Date",
      "time": "Time ET"
    }
  ]
}
```

## **Key Learnings**
- Containerizing applications with Docker and deploying to AWS ECS Fargate
- Setting up Application Load Balancers for container orchestration
- Creating and managing REST APIs with Amazon API Gateway
- Integrating external APIs (SerpAPI) in cloud applications
- Implementing health checks and monitoring for containerized services

## **Architecture Benefits**
- **Scalability**: ECS Fargate automatically scales based on demand
- **Serverless**: No server management required
- **High Availability**: Multi-AZ deployment with ALB
- **Cost Effective**: Pay only for resources used
- **Security**: VPC isolation and IAM role-based access

## **Future Enhancements**
- **Caching**: Add Amazon ElastiCache for frequent API requests
- **Database**: Implement DynamoDB for user preferences and query history
- **Security**: Add API Gateway authentication (API keys, Cognito, or IAM)
- **Monitoring**: Integrate CloudWatch dashboards and alarms
- **CI/CD**: Implement automated deployments with CodePipeline
- **Multi-Region**: Deploy across multiple AWS regions for global availability

## **Troubleshooting**

**Common Issues:**
- **Docker daemon not running**: Run `sudo systemctl start docker`
- **ECR authentication failed**: Re-run the ECR login command
- **ECS task failing**: Check CloudWatch logs for container errors
- **API Gateway 502 errors**: Verify ALB health checks are passing
- **No sports data returned**: Verify SPORTS_API_KEY environment variable is set

**Useful Commands:**
```bash
# Check ECS service status
aws ecs describe-services --cluster sports-api-cluster --services sports-api-service

# View container logs
aws logs describe-log-groups --log-group-name-prefix "/ecs/sports-api"

# Test ALB directly
curl -v http://your-alb-dns-name.elb.amazonaws.com/sports
```

## **Cost Estimation**
- **ECS Fargate**: ~$0.04/hour per task (0.25 vCPU, 0.5 GB RAM)
- **Application Load Balancer**: ~$0.0225/hour + $0.008/LCU-hour
- **API Gateway**: $3.50 per million API calls
- **ECR**: $0.10/GB-month for storage

**Estimated monthly cost for low traffic**: $15-25/month

## **License**

This project is for educational and demonstration purposes.

---

**Ready to deploy your own containerized API? Fork this repository and customize it for your use case!**


