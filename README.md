# 🛒 CloudMart – Multicloud DevOps & AI E-commerce Platform

CloudMart is an AI-powered, production-ready e-commerce platform built over a 5-day hands-on bootcamp. It leverages modern DevOps practices, serverless computing, Kubernetes, and Generative AI (Claude 3 Sonnet, GPT-4o) across AWS, Google Cloud, and Azure.


### Key Components:
- **Frontend:** React + Tailwind UI, hosted on EKS
- **Backend:** Node.js + Express APIs running on EKS
- **Product Data:** Stored in Amazon DynamoDB
- **AI Agents:** Amazon Bedrock (Claude 3 Sonnet) + OpenAI GPT-4o
- **CI/CD:** GitHub + AWS CodePipeline + CodeBuild
- **Analytics:** Real-time order tracking in Google BigQuery
- **Sentiment Analysis:** Microsoft Azure AI Language Service

---

## 📆 5-Day Development Summary

### ✅ Day 1 – Infrastructure Provisioning (Terraform + AWS)
- Provisioned 3 DynamoDB tables: `cloudmart-products`, `cloudmart-orders`, `cloudmart-tickets`
- Created IAM roles and Lambda execution policies
- Defined all resources as reusable Terraform code

### ✅ Day 2 – Dockerization & Kubernetes (Amazon EKS)
- Created Dockerfiles for frontend and backend
- Pushed images to Amazon ECR
- Deployed apps to Amazon EKS via YAML manifests
- Exposed services with LoadBalancer + Ingress

### ✅ Day 3 – CI/CD Automation (GitHub + CodePipeline)
- Configured GitHub repo as source for CodePipeline
- Used CodeBuild to:
  - Install dependencies
  - Build Docker images
  - Push to ECR
  - Deploy using `kubectl`
- Validated end-to-end automation on every commit

### ✅ Day 4 – AI Agent (Amazon Bedrock + Lambda)
- Built a Claude 3 Sonnet agent for product recommendation
- Connected via OpenAPI schema and Lambda trigger
- Query logic handled in `list_products` Lambda → DynamoDB
- Enabled natural, personalized shopping experience via agent

### ✅ Day 5 – Real-Time Analytics + Sentiment Analysis
- Streamed orders from DynamoDB to Google BigQuery
- Built Lambda connector with `google_credentials.json`
- Ran Azure AI Language Sentiment API for customer feedback
- Final architecture enabled multicloud orchestration and analytics

---

## 🧠 Features

- 🔁 End-to-end CI/CD with GitHub → AWS CodePipeline
- 🛍️ Product recommendations via Claude 3 LLM Agent
- 💬 GPT-4o Assistant for 24/7 customer support
- 📈 Real-time order ingestion into Google BigQuery
- 💡 Azure AI-powered feedback sentiment analysis
- ☁️ Hybrid cloud compute: AWS, GCP, Azure integrated
- 📦 Fully containerized + Kubernetes-ready

---

## 🛠️ Technologies Used

| Stack        | Tools/Services                                                                 |
|--------------|----------------------------------------------------------------------------------|
| Infrastructure | Terraform, AWS IAM, EKS, Lambda, CodePipeline, CodeBuild                     |
| Backend      | Node.js, Express, Amazon DynamoDB, Lambda                                       |
| Frontend     | React, Tailwind CSS, Vite                                                       |
| CI/CD        | GitHub, CodeBuild, ECR, kubectl                                                 |
| AI Agents    | Amazon Bedrock (Claude 3 Sonnet), OpenAI GPT-4o Assistant, Azure AI Language    |
| Analytics    | Google BigQuery + Lambda Event Stream                                           |
| DevOps       | Docker, Kubernetes, AWS CLI, CloudWatch                                         |

---

