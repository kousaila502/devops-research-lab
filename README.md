# DevOps Research Lab 🔬

> Real-world research documenting my master's thesis: **"Leveraging GitOps for Scalable and Maintainable CI/CD Pipelines"**

## 🎯 Research Overview

This repository documents my hands-on research comparing Traditional CI/CD vs GitOps methodologies, conducted as part of my master's thesis at École Supérieure d'Informatique (ESI-SBA).

**Key Finding:** GitOps demonstrated **5.1x faster deployment performance** with **80% cost optimization** achieved on Google Cloud Platform.

## 📊 Performance Results

| Methodology | Deployment Time | Automation Level | Human Intervention | Cost Efficiency |
|-------------|----------------|-----------------|-------------------|-----------------|
| Traditional CI/CD | 698 seconds (11m 38s) | 11% | 89% wait time | Baseline |
| GitOps (ArgoCD) | ~175 seconds | 100% | 0% | $78/month optimized |

## 🏗️ Research Infrastructure

**Test Application:** E-commerce microservices platform
- **Cart Service:** Java/Spring Boot + MongoDB/Redis
- **Order Service:** Python/FastAPI + PostgreSQL  
- **Product Service:** Node.js/Express + MongoDB
- **Search Service:** Node.js/Express + Elasticsearch
- **User Service:** Python/FastAPI + MongoDB/Redis
- **Frontend:** React/TypeScript + Nginx

**Cloud Infrastructure:**
- **Platform:** Google Kubernetes Engine (GKE)
- **Configuration:** e2-small instances with Spot VMs
- **Cost Optimization:** 80% reduction ($383 → $78/month)
- **Location:** northamerica-northeast1-a

## 🔬 What You'll Find Here

### [Research Methodology](research-methodology/)
- Testing environment setup (Minikube → GKE migration)
- 22 metrics collection approach
- Baseline establishment process

### [Monitoring Stack](monitoring-stack/)
- Prometheus configurations from actual research
- Grafana dashboard exports
- Pushgateway setup for CI/CD metrics

### [Testing Scenarios](testing-scenarios/)
- Traditional CI/CD with manual approval gates
- GitOps/ArgoCD automated deployment
- Performance measurement scripts

### [Results & Analysis](results-analysis/)
- Raw performance data and cost analysis
- Infrastructure optimization journey
- Error logs and solutions from real implementation

## 🎓 Academic Context

**Institution:** École Supérieure d'Informatique (ESI-SBA)  
**Program:** Master's in Information Systems Engineering  
**Timeline:** April 2025 - September 2025  
**Defense:** September 2025

---
*Contributing to academic understanding of modern DevOps practices and their quantifiable impact on software delivery performance.*