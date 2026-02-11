# 🎬 Netflix DevSecOps Project

Projeto completo de CI/CD com deploy em Kubernetes (AWS EKS), incluindo
containerização, análise de qualidade de código, monitoramento e
integração contínua. 

Não coloquei os códigos dos arquivos pois tudo foi criado numa EC2, nada foi feito localmente.

------------------------------------------------------------------------

# 📌 Objetivo do Projeto

Construir um ambiente **DevSecOps real**, simulando um fluxo
profissional de:

-   Versionamento no GitHub
-   Integração Contínua com Jenkins
-   Análise de código com SonarQube
-   Build e Push de imagem Docker
-   Deploy automatizado em cluster Kubernetes (AWS EKS)
-   Monitoramento com Prometheus + Grafana

Este projeto foi executado manualmente, enfrentando e resolvendo
problemas reais de infraestrutura, autenticação, permissões e integração
entre ferramentas.

------------------------------------------------------------------------

# 🏗️ Arquitetura Geral

``` text
GitHub (Source Code)
        ↓
Jenkins (Pipeline CI/CD)
        ↓
SonarQube (Code Analysis)
        ↓
Docker Build
        ↓
DockerHub (Image Registry)
        ↓
AWS EKS (Kubernetes Cluster)
        ↓
Prometheus + Grafana (Monitoring)
```

------------------------------------------------------------------------

# ☁️ Infraestrutura Provisionada

## 🔹 AWS EC2 (Servidor CI)

Utilizada para:

-   Executar Jenkins em container
-   Executar SonarQube em container
-   Controlar deploy no EKS

Configurações importantes: - Ubuntu Server - Docker instalado - Docker
socket compartilhado com Jenkins

------------------------------------------------------------------------

# 🐳 Jenkins em Docker

Jenkins rodando como container Docker customizado contendo:

-   Docker CLI
-   AWS CLI
-   kubectl
-   Git
-   NodeJS

### Execução com montagem do Docker Socket

``` bash
-v /var/run/docker.sock:/var/run/docker.sock
```

Permite que o Jenkins execute `docker build` e `docker push`.

------------------------------------------------------------------------

# 🔐 Gerenciamento de Credenciais

Configurado no Jenkins:

-   dockerhub-creds (username + token)
-   SONAR_TOKEN (Secret Text)
-   aws-access-key (Secret Text)
-   aws-secret-key (Secret Text)
-   TMDB_V3_API_KEY (Secret Text)

Todas utilizadas via `withCredentials()`.

------------------------------------------------------------------------

# 🔍 SonarQube

## Instalação

Executado em container Docker separado.

``` bash
docker run -d -p 9000:9000 sonarqube
```

## Integração Jenkins

-   Token gerado no SonarQube
-   Configurado no Jenkins Global Tool Configuration
-   Scanner configurado como "sonar-scanner"

Pipeline executa:

``` bash
sonar-scanner \
  -Dsonar.projectKey=Netflix \
  -Dsonar.login=$SONAR_TOKEN
```

------------------------------------------------------------------------

# 📦 Containerização

## Dockerfile

Build multi-stage para gerar aplicação estática e servir via nginx.

``` dockerfile
FROM node:16-alpine as builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx:stable-alpine
WORKDIR /usr/share/nginx/html
COPY --from=builder /app/dist .
EXPOSE 80
```

------------------------------------------------------------------------

# 📤 DockerHub

-   Repositório criado manualmente
-   Autenticação feita via token
-   Push automático realizado pelo Jenkins

``` bash
docker push bmcruzz1/netflix-devsecops:latest
```

------------------------------------------------------------------------

# ☸️ Kubernetes (AWS EKS)

Cluster criado com:

``` bash
eksctl create cluster \
  --name netflix-devsecops-cluster \
  --region sa-east-1
```

Inclui:

-   Control Plane gerenciado
-   NodeGroup
-   VPC automática
-   IAM Roles

------------------------------------------------------------------------

# 📜 Manifests Kubernetes

## deployment.yaml

-   2 réplicas
-   ImagePullPolicy Always
-   Porta 80

## service.yaml

Tipo:

``` yaml
type: LoadBalancer
```

Resultado:

-   ELB público gerado automaticamente
-   Aplicação acessível via URL AWS

------------------------------------------------------------------------

# 🔄 Pipeline CI/CD

## Fluxo completo:

1.  Checkout do código
2.  Instalação dependências
3.  SonarQube Scan
4.  Docker Build
5.  Docker Push
6.  Deploy no EKS
7.  Rollout validation

------------------------------------------------------------------------

# 📊 Monitoramento

Stack instalada via Helm:

-   Prometheus
-   Grafana
-   kube-state-metrics

Dashboards utilizados:

-   Kubernetes Nodes
-   Pods
-   Deployments
-   CPU / Memória

------------------------------------------------------------------------

# 🐛 Problemas Reais Enfrentados

Durante o desenvolvimento, foram enfrentados e resolvidos problemas
como:

-   docker: permission denied
-   sonar-scanner not found
-   aws not found
-   kubectl not found
-   fatal: not in a git directory
-   autenticação EKS via IAM
-   AWS token via exec plugin
-   permissões aws-auth no cluster
-   falta de memória na EC2

Cada problema exigiu troubleshooting técnico, análise de logs e ajustes
estruturais na arquitetura.

------------------------------------------------------------------------

# 🧠 Conceitos Aplicados

-   CI/CD real
-   Docker-in-Docker via socket
-   IAM + RBAC Kubernetes
-   Credentials seguras no Jenkins
-   Multi-stage builds
-   Cluster externo com autenticação por token
-   Rollout controlado com kubectl


------------------------------------------------------------------------

# 📈 Possíveis Melhorias Futuras

-   Versionamento de imagem por build number
-   Deploy canário
-   ArgoCD
-   Terraform para infraestrutura
-   Jenkins dentro do EKS
-   GitHub Actions alternativa
-   Sonar Quality Gate bloqueando deploy

------------------------------------------------------------------------

# 👨‍💻 Autor

Bruno Martins \
Cloud & DevOps Engineer \

------------------------------------------------------------------------

# 🏁 Conclusão

Este projeto representa uma implementação prática de um pipeline
DevSecOps completo, abrangendo infraestrutura, automação, segurança,
monitoramento e deploy em cloud.

Foi desenvolvido com foco em aprendizado técnico profundo e resolução de
problemas reais de integração entre ferramentas.
