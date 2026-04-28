# DevOps + AIOps Playbook

A production-style cloud-native system I designed and built end-to-end — microservices deployed on AWS EKS, automated with a full CI/CD pipeline, managed through GitOps, and extended with an AI-powered SRE assistant that investigates incidents autonomously.

---

## Why I Built This

I wanted to go beyond tutorials and build something that reflects how real DevOps systems actually work in production. That meant making actual engineering decisions: how to structure the infrastructure, how to wire CI/CD to GitOps so deployments are fully automated, and how to add AI-driven observability on top of a live Kubernetes cluster.

The result is a system where a code push triggers a build pipeline, updates the cluster through ArgoCD, and can be monitored and diagnosed by an AI agent — without manual intervention.

---

## Architecture

```
Developer Push
      │
      ▼
GitHub Actions CI
  ├── Build Docker images (7 services in parallel)
  ├── Push to Amazon ECR
  └── Update Kubernetes manifests in Git
            │
            ▼
        ArgoCD (GitOps)
          └── Detects Git change → syncs EKS cluster
                    │
                    ▼
            AWS EKS Cluster
              ├── Boutique Microservices (7 services)
              ├── Prometheus + Grafana (monitoring)
              └── Fluent Bit → CloudWatch (log forwarding)
                            │
                            ▼
                    Kira — AI SRE Assistant
                      ├── AWS Bedrock Agent
                      ├── Lambda: fetch CloudWatch logs
                      ├── Lambda: query Prometheus metrics
                      └── Lambda: check EKS cluster health
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Application | React, Node.js, PostgreSQL |
| Containers | Docker, Docker Compose |
| Orchestration | Kubernetes on AWS EKS |
| Infrastructure as Code | Terraform (VPC, EKS, ECR, ArgoCD) |
| CI/CD | GitHub Actions |
| GitOps | ArgoCD + Kustomize |
| Container Registry | Amazon ECR |
| Monitoring | Prometheus + Grafana |
| Log Forwarding | AWS Fluent Bit → CloudWatch |
| AIOps | AWS Bedrock Agent + AWS Lambda |

---

## What I Built

### Microservices Application — `projects/boutique-microservices/`

A multi-service e-commerce system with 7 independent services: auth, gateway, orders, order-service, product-service, user-service, and a React frontend. Each service runs in its own container and communicates over internal Kubernetes networking. I used Docker Compose for local development before moving to EKS.

### Cloud Infrastructure — `projects/Infrastructure/`

I wrote Terraform modules to provision the full AWS environment from scratch:

- **VPC** — custom subnets across availability zones with proper routing
- **EKS** — managed node group with autoscaling configuration
- **ECR** — one container registry per service
- **ArgoCD** — installed into the cluster via Helm at provision time, so GitOps is ready the moment infrastructure comes up

Running `terraform apply` stands up the entire platform. No manual console steps.

### CI/CD Pipeline — `.github/workflows/ci.yml`

The GitHub Actions pipeline uses a matrix strategy to build all 7 service images in parallel, tags each with the Git commit SHA for traceability, pushes to ECR, then automatically updates the Kubernetes manifests in the `gitops/` directory and commits them back to the repository. ArgoCD picks up the change and syncs the cluster. The result is a fully automated path from code to production.

### Kira — AI SRE Assistant — `projects/aiops-assistant/`

I built an AI-powered incident response agent on AWS Bedrock. Kira connects to three Lambda functions as action groups:

| Action | What it does |
|---|---|
| `fetch_logs` | Queries CloudWatch Logs for errors and stack traces |
| `fetch_metrics` | Queries Prometheus for CPU, memory, and latency metrics |
| `fetch_service_health` | Checks EKS cluster and node group status |

When something goes wrong, I can open the Streamlit UI and ask Kira in plain English. It queries the right data sources, reasons over the results, and gives back a root cause with evidence and a recommended fix — instead of me digging through dashboards manually.

**Example queries:**
- *Why are we seeing 503 errors in the last hour?*
- *Is CPU usage high across the boutique services?*
- *Are all pods healthy? Any recent restarts?*

---

## Repository Structure

```
devops-ai-playbook/
├── .github/
│   └── workflows/ci.yml          # CI pipeline — build, push, update manifests
├── docs/
│   ├── part1-system-design.md    # System design decisions and tradeoffs
│   ├── part2-workflow.md         # End-to-end flow from dev to production
│   └── claude-setup.md           # Claude Code + MCP server config
├── gitops/
│   ├── argo-cd.yml               # ArgoCD Application manifest
│   ├── kustomization.yml         # Kustomize entry point
│   └── k8s/                      # Kubernetes manifests for all services
├── projects/
│   ├── boutique-microservices/   # Application services + Docker Compose
│   ├── Infrastructure/           # Terraform modules
│   └── aiops-assistant/          # Bedrock Agent, Lambdas, Streamlit UI
└── CLAUDE.md                     # AI assistant configuration
```

---

## Running It

### Local development

```bash
cd projects/boutique-microservices
cp .env.example .env
docker-compose up -d
./health-check.sh
```

### Provision infrastructure

```bash
cd projects/Infrastructure
terraform init
terraform plan
terraform apply
```

### Deploy the AI assistant

```bash
cd projects/aiops-assistant

./setup-iam.sh        # Create IAM roles for Lambda and Bedrock
./deploy.sh           # Create Bedrock Agent + attach Lambda action groups

cp .env.example .env  # Add BEDROCK_AGENT_ID from deploy output

pip install -r requirements.txt
streamlit run app.py  # Open http://localhost:8501
```

---

## Author

**Anicet Hermann**
- GitHub: [@anicethermannzie](https://github.com/anicethermannzie)
- Email: hermann042009@gmail.com
