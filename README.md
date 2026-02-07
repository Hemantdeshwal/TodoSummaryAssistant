Todo Summary Assistant
A full‑stack app to manage personal to‑do items, summarize pending tasks using Cohere LLM, and send the summary to a Slack channel.

Features
Create, edit, delete to‑do items (full CRUD).

View current to‑do list with status.

Summarize pending to‑dos via Cohere LLM.

Send the summary to a Slack channel using Incoming Webhooks.

Show notifications for success or failure of Slack operations.
​

Tech Stack
Frontend: React, HTML, CSS, JavaScript, Axios.

Backend: Spring Boot (Java 17+), Maven.

Database: MySQL (Spring Data JPA, Hibernate).

LLM: Cohere API.

Messaging: Slack Incoming Webhooks.

HTTP client: OkHttp.

Containerization: Docker.

CI: Jenkins.

Orchestration: Kubernetes.

GitOps: Argo CD.

1. Setup Instructions
1.1 Prerequisites
JDK 17

Maven 3.9+

Node.js 18+ and npm

MySQL 8.x

Docker (optional, for containers)

Kubernetes + kubectl (optional, for K8s)

Argo CD (optional, for GitOps)
​

1.2 Backend – Local Run
Clone the repo and go to the backend:


git clone https://github.com/<your-username>/TodoSummaryAssistant.git
cd TodoSummaryAssistant/Backend/todo-summary-assistant
Set env vars (adjust values):


export SPRING_DATASOURCE_URL="jdbc:mysql://localhost:3306/todo_db?createDatabaseIfNotExist=true"
export SPRING_DATASOURCE_USERNAME="root"
export SPRING_DATASOURCE_PASSWORD="your_mysql_password"
export COHERE_API_KEY="your_cohere_api_key"
export SLACK_WEBHOOK_URL="your_slack_webhook_url"
Build and run:

./mvnw clean package
java -jar target/todo-summary-assistant-*.jar
Backend URL: http://localhost:8080.
​

1.3 Frontend – Local Run
cd TodoSummaryAssistant/Frontend
npm install
npm start
Frontend URL: http://localhost:3000.
It calls the backend at http://localhost:8080 (change API base URL in frontend if needed).
​

1.4 Environment Variables
Backend expects:

SPRING_DATASOURCE_URL

SPRING_DATASOURCE_USERNAME

SPRING_DATASOURCE_PASSWORD

COHERE_API_KEY

SLACK_WEBHOOK_URL
​

In Kubernetes:

ConfigMap: DB URL, DB username.

Secret: DB password, Cohere key, Slack webhook.
​

1.5 Assumptions
MySQL is reachable and user can create/access todo_db.

Local MySQL runs on localhost:3306.

Cohere and Slack credentials are passed via env vars or K8s Secrets and never committed to Git.

Dev/prod use separate namespaces and ConfigMaps/Secrets in K8s.

2. Docker Usage
Backend Dockerfile: Backend/todo-summary-assistant/Dockerfile (multi‑stage, non‑root, externalized config).
​

Build:


cd Backend/todo-summary-assistant
docker build -t <hemantdeshwal19>/todo-backend:latest .
Run:


docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_URL="jdbc:mysql://host.docker.internal:3306/todo_db?createDatabaseIfNotExist=true" \
  -e SPRING_DATASOURCE_USERNAME="root" \
  -e SPRING_DATASOURCE_PASSWORD="your_mysql_password" \
  -e COHERE_API_KEY="your_cohere_api_key" \
  -e SLACK_WEBHOOK_URL="your_slack_webhook_url" \
  <hemantdeshwal19>/todo-backend:latest
Key choices: multi‑stage image, non‑root user, all config via env vars.
​

3. Kubernetes Deployment
K8s manifests live in k8s/:
​

k8s/dev: deployment.yaml, service.yaml, configmap.yaml, secret.yaml

k8s/prod: deployment.yaml, service.yaml, configmap.yaml, secret.yaml

Deployment
Image: hemantdeshwal19/todo-backend:latest (or a tag you set via GitOps).

CPU/memory requests & limits.

Liveness & readiness probes on /actuator/health, port 8080.

envFrom ConfigMap + Secret.
​

Service
Type: ClusterIP, port 80 → container port 8080.

Labels: app: todo-backend, env: dev|prod.
​

ConfigMap / Secret
ConfigMap: SPRING_DATASOURCE_URL, SPRING_DATASOURCE_USERNAME.

Secret: SPRING_DATASOURCE_PASSWORD, COHERE_API_KEY, SLACK_WEBHOOK_URL (base64).
​

4. GitOps with Argo CD
GitOps config: gitops/argocd-apps/.
​

dev-app.yaml
repoURL: https://github.com/<Hemantdeshwal>/TodoSummaryAssistant.git

path: k8s/dev

namespace: dev

prod-app.yaml
repoURL: https://github.com/<Hemantdeshwal>/TodoSummaryAssistant.git

path: k8s/prod

namespace: prod
​

Sync policy: automated, prune: true, selfHeal: true.
​

Flow:

Jenkins builds and pushes Docker image.

You update the image tag or manifest in k8s/dev or k8s/prod and push to Git.

Argo CD sees the Git change and syncs the cluster.

Rollback: revert the manifest change (e.g. image tag) in Git; Argo CD rolls the cluster back.

5. CI with Jenkins
A Jenkinsfile at the repo root will:
​

Checkout code.

Run ./mvnw clean verify in Backend/todo-summary-assistant.

Build Docker image tagged with commit SHA.

Push to Docker Hub using Jenkins credentials.

Not call kubectl or Argo CD; deployments are GitOps‑driven.

6. LLM (Cohere) Setup
Create account: https://cohere.ai

Get API key from Cohere dashboard.

Provide it as COHERE_API_KEY or via K8s Secret mapped to that env var.

Backend uses this key to call Cohere and summarize pending to‑dos.
​

7. Slack Integration Setup
Create Slack app at https://api.slack.com/apps.

Enable Incoming Webhooks.

Create a webhook for your target channel (e.g. #todos).

Set SLACK_WEBHOOK_URL or store it in a K8s Secret.

Backend posts summaries to that Slack channel.
​

8. Design / Architecture Decisions
Separate frontend and backend directories for independent deploys.

RESTful API between React and Spring Boot.

Spring Data JPA for MySQL access.

Service layer for business logic and external integrations.

OkHttp for Cohere and Slack HTTP calls.

CORS config in backend for local frontend dev.

Errors surfaced via frontend notifications and backend logs.

Config and secrets externalized (env vars, ConfigMaps, Secrets).

GitOps: cluster state = Git state; CI only builds & pushes images.

