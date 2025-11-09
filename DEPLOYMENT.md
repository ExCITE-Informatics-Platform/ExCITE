# ExCITE Platform Deployment Guide

Comprehensive deployment strategies for the ExCITE integrated healthcare informatics platform.

---

## Table of Contents

- [Deployment Overview](#deployment-overview)
- [Pre-Deployment Checklist](#pre-deployment-checklist)
- [Strategy 1: Single-Server Development](#strategy-1-single-server-development)
- [Strategy 2: Distributed Multi-Server](#strategy-2-distributed-multi-server)
- [Strategy 3: Azure Cloud Production](#strategy-3-azure-cloud-production)
- [Module-Specific Deployment](#module-specific-deployment)
- [Network Configuration](#network-configuration)
- [Security Hardening](#security-hardening)
- [Monitoring and Observability](#monitoring-and-observability)
- [Backup and Disaster Recovery](#backup-and-disaster-recovery)
- [Performance Tuning](#performance-tuning)

---

## Deployment Overview

### Architecture Decision Matrix

| Criteria | Single-Server | Distributed | Azure Cloud |
|----------|--------------|-------------|-------------|
| **Users** | 5-8 | 20-30 | 30-50+ |
| **Cost** | Low ($0 dev) | Medium (hardware) | High ($535-910/mo) |
| **Complexity** | Low | Medium | High |
| **Scalability** | Limited | Moderate | High |
| **Availability** | Basic | Improved | Enterprise |
| **Best For** | Development, Testing | Small Production | Enterprise, Azure AD Integration |

### Resource Requirements Summary

**Single-Server (All Modules)**:
- CPU: 12 cores minimum, 24 recommended
- RAM: 32GB minimum, 56GB recommended
- Disk: 140GB minimum, 500GB recommended
- Network: 1Gbps internal, 100Mbps external

**Distributed (3 Servers)**:
- Total CPU: 16 cores, 40GB RAM, 400GB disk
- Network: 10Gbps internal between servers recommended

**Azure Cloud**:
- 14 vCPUs, 40GB RAM, 400GB disk (managed)
- See [Azure Cloud Production](#strategy-3-azure-cloud-production) for detailed VM sizing

---

## Pre-Deployment Checklist

### System Requirements

- [ ] **Operating System**: Ubuntu 22.04 LTS / Debian 11+ / RHEL 8+ (or macOS/Windows with Docker Desktop)
- [ ] **Docker**: >= 28.3.2
- [ ] **Docker Compose**: V2 (included with Docker Desktop)
- [ ] **Git**: Latest version
- [ ] **Python**: 3.9+ (3.11 recommended) for InformaticsClassroom
- [ ] **Node.js**: 18+ (20 recommended) for InformaticsClassroom frontend
- [ ] **Disk Space**: Verify available space per deployment strategy
- [ ] **Ports Available**: 80, 443, 3000, 5001, 5173, 8000, 8888

### External Services

- [ ] **Azure AD Tenant** (if using InformaticsClassroom with Azure AD)
- [ ] **OMOP Vocabularies** downloaded from https://athena.ohdsi.org/ (5-10GB)
- [ ] **SSL Certificates** (Let's Encrypt or custom) for production
- [ ] **DNS Records** configured for public deployments

### API Keys and Secrets

- [ ] **JUPYTERHUB_CRYPT_KEY**: `openssl rand -hex 32`
- [ ] **Database Passwords**: Strong passwords for PostgreSQL instances
- [ ] **Anthropic Claude API** (optional - WintEHR AI features)
- [ ] **Tavily API** (optional - WintEHR web research)
- [ ] **Azure AD Client ID/Secret** (InformaticsClassroom production)

### Security Preparation

- [ ] **Firewall Rules**: Configure host firewall (ufw, firewalld, Azure NSG)
- [ ] **SSH Keys**: Disable password auth, use key-based authentication
- [ ] **Secrets Management**: Prepare .env files with strong passwords
- [ ] **SSL/TLS**: Obtain certificates for HTTPS (production)

---

## Strategy 1: Single-Server Development

**Use Case**: Local development, testing, small workshops, proof-of-concept

**Resource Requirements**:
- **Development**: 12 CPU cores, 32GB RAM, 140GB disk
- **Recommended**: 24 CPU cores, 56GB RAM, 500GB disk
- **Max Users**: 5-8 concurrent

### Step-by-Step Deployment

#### 1. Prepare Host System

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker (if not already installed)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify Docker installation
docker --version
docker compose version

# Install Python and Node.js (for InformaticsClassroom)
sudo apt install python3.11 python3.11-venv python3-pip nodejs npm -y

# Verify versions
python3.11 --version  # Should be 3.11+
node --version         # Should be v18+
npm --version          # Should be 9+
```

#### 2. Clone ExCITE Repository

```bash
cd ~
git clone <excite-repo-url> ExCITE
cd ExCITE
```

#### 3. Deploy WintEHR (FHIR EHR System)

```bash
cd WintEHR

# Copy and configure
cp config.example.yaml config.yaml

# Optional: Edit config.yaml for custom settings
# - Anthropic Claude API key for AI features
# - Tavily API key for web research
# - Patient cohort size (default: 50)

# Deploy (generates synthetic patients, starts all services)
./deploy.sh

# Wait for deployment to complete (5-10 minutes)
# Verify: http://localhost:3000 (UI), http://localhost:8888/fhir (FHIR API)
```

**Expected Services**:
- Frontend: http://localhost:3000
- FHIR Server: http://localhost:8888/fhir
- Backend API: http://localhost:8000
- PostgreSQL: localhost:5432
- Redis: localhost:6379

#### 4. Deploy Broadsea (OHDSI Platform)

```bash
cd ../Broadsea

# Copy environment file
cp .env.example .env

# Generate secure passwords
echo "WEBAPI_DATASOURCE_PASSWORD=$(openssl rand -base64 32)" >> .env
echo "POSTGRES_PASSWORD=$(openssl rand -base64 32)" >> .env

# Download OMOP Vocabularies (REQUIRED for research)
# 1. Go to https://athena.ohdsi.org/
# 2. Create account (free)
# 3. Select vocabularies: SNOMED CT, LOINC, RxNorm, ICD-10, CPT-4
# 4. Download bundle (5-10GB)
# 5. Extract to ./omop_vocab directory

mkdir -p omop_vocab
# Extract downloaded vocabularies here

# Deploy with profiles (CRITICAL - services won't start without profiles)
docker compose --profile atlas-from-image \
               --profile webapi-from-image \
               --profile cdm-postprocessing \
               --profile solr-vocab-with-import up -d

# Load OMOP vocabularies (10-30 minutes depending on size)
docker compose --profile omop-vocab-pg-load up

# Verify: http://127.0.0.1/atlas (Broadsea Atlas UI)
```

**Expected Services**:
- Atlas (Cohort Definition): http://127.0.0.1/atlas
- WebAPI: http://127.0.0.1/WebAPI
- Traefik (Reverse Proxy): http://127.0.0.1
- PostgreSQL: localhost:5432
- SOLR (Vocabulary Search): localhost:8983

**Important Notes**:
- Broadsea uses **Docker Compose profiles** - services won't start without them
- OMOP vocabularies are REQUIRED for Atlas to function
- Vocabulary loading is a one-time operation but takes 10-30 minutes

#### 5. Deploy JupyterHubClassroom

```bash
cd ../JupyterHubClassroom

# Copy environment file
cp .env.example .env

# Generate required secrets
echo "JUPYTERHUB_CRYPT_KEY=$(openssl rand -hex 32)" >> .env
echo "POSTGRES_PASSWORD=$(openssl rand -base64 32)" >> .env

# Optional: Configure resource limits (edit .env)
# DOCKER_NOTEBOOK_CPU_LIMIT=2.0
# DOCKER_NOTEBOOK_MEM_LIMIT=2G

# Run setup script (builds images, starts services)
./scripts/setup.sh

# Verify: http://localhost:8000
```

**Expected Services**:
- JupyterHub: http://localhost:8000
- PostgreSQL: localhost:5433 (different port to avoid conflicts)

**First-Time Setup**:
1. Navigate to http://localhost:8000
2. Click "Sign Up"
3. Create admin account (first user is auto-admin)
4. Approve additional users via Admin panel

**InformaticsLessons Integration**:
- Course materials are pre-configured via nbgitpuller
- Users access lessons via: http://localhost:8000/hub/user-redirect/git-pull?repo=https://github.com/UltraUB/InformaticsLessons&branch=main&subPath=lessons

#### 6. Deploy InformaticsClassroom (Optional)

```bash
cd ../InformaticsClassroom

# Backend setup
python3.11 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

# Copy and configure environment
cp .env.example .env

# For development (SQLite + local auth)
echo "DATABASE_TYPE=sqlite" >> .env
echo "SECRET_KEY=$(openssl rand -hex 32)" >> .env

# Start backend
python3 app.py &  # Runs on http://localhost:5001

# Frontend setup (new terminal)
cd frontend
npm install
npm run dev  # Runs on http://localhost:5173
```

**Expected Services**:
- Backend API: http://localhost:5001
- Frontend UI: http://localhost:5173
- Database: SQLite (./instance/informatics_classroom.db)

#### 7. Verify Integrated Platform

```bash
# Check all services
docker ps  # Should show Broadsea, JupyterHub, WintEHR containers

# Test WintEHR FHIR API
curl http://localhost:8888/fhir/Patient?_count=5

# Test Broadsea Atlas
curl http://127.0.0.1/WebAPI/info

# Test JupyterHub
curl http://localhost:8000/hub/api

# Test InformaticsClassroom
curl http://localhost:5001/api/health
```

**Service URLs**:
- WintEHR UI: http://localhost:3000
- WintEHR FHIR: http://localhost:8888/fhir
- Broadsea Atlas: http://127.0.0.1/atlas
- JupyterHub: http://localhost:8000
- InformaticsClassroom: http://localhost:5173

### Port Conflict Resolution

**Issue**: WintEHR backend (8000) conflicts with JupyterHub (8000)

**Solution 1 - Change JupyterHub Port**:
```bash
cd JupyterHubClassroom
echo "JUPYTERHUB_PORT=8001" >> .env
docker compose down && docker compose up -d
# Access JupyterHub at http://localhost:8001
```

**Solution 2 - Use Host Networking (Linux only)**:
```bash
# In JupyterHubClassroom docker-compose.yml
# Change: ports: ["8000:8000"]
# To: network_mode: host
```

**Solution 3 - Reverse Proxy** (Recommended for production):
See [Network Configuration](#network-configuration) section.

---

## Strategy 2: Distributed Multi-Server

**Use Case**: Small production deployment, improved isolation, resource distribution

**Resource Requirements**:
- **Server 1 (Clinical Data)**: 4 CPU, 16GB RAM, 150GB disk
- **Server 2 (Research Platform)**: 8 CPU, 16GB RAM, 150GB disk
- **Server 3 (Education)**: 4 CPU, 8GB RAM, 100GB disk
- **Network**: 10Gbps internal recommended

### Server Distribution

#### Server 1: Clinical Data (WintEHR)

**Purpose**: FHIR EHR system and patient data generation

```bash
# Server 1: wintehr.example.com (4 CPU, 16GB RAM, 150GB disk)

cd ~/ExCITE/WintEHR
cp config.example.yaml config.yaml

# Configure for external access
# Edit config.yaml:
# - Set public_url: https://wintehr.example.com
# - Configure SSL certificates
# - Set FHIR base URL: https://wintehr.example.com:8888/fhir

./deploy.sh

# Configure firewall
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8888/tcp  # FHIR API (consider restricting to internal network)
sudo ufw enable
```

**Exposed Services**:
- UI: https://wintehr.example.com
- FHIR API: https://wintehr.example.com:8888/fhir

#### Server 2: Research Platform (Broadsea)

**Purpose**: OHDSI research tools and OMOP CDM

```bash
# Server 2: broadsea.example.com (8 CPU, 16GB RAM, 150GB disk)

cd ~/ExCITE/Broadsea
cp .env.example .env

# Configure environment
# Edit .env:
# - Set public URLs for Atlas/WebAPI
# - Configure PostgreSQL passwords
# - Optional: Configure LDAP/OAuth authentication

# Deploy with all profiles
docker compose --profile atlas-from-image \
               --profile webapi-from-image \
               --profile cdm-postprocessing \
               --profile solr-vocab-with-import up -d

# Load vocabularies
docker compose --profile omop-vocab-pg-load up

# Configure Traefik for SSL (production)
# See docker-compose.yml traefik service configuration

# Configure firewall
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

**Exposed Services**:
- Atlas: https://broadsea.example.com/atlas
- WebAPI: https://broadsea.example.com/WebAPI

**Network Configuration**:
- For custom FHIR-OMOP integration: Allow Server 2 to access Server 1 FHIR API on port 8888
- Configure connection pooling for database access
- Optional: VPN tunnel between servers for secure communication

#### Server 3: Education Platform (JupyterHub + InformaticsClassroom)

**Purpose**: Interactive learning environment

```bash
# Server 3: jupyter.example.com (4 CPU, 8GB RAM, 100GB disk)

# Deploy JupyterHub
cd ~/ExCITE/JupyterHubClassroom
cp .env.example .env

# Configure for WintEHR access
# Edit .env or jupyterhub_config.py:
# - Set WINTEHR_FHIR_URL=https://wintehr.example.com:8888/fhir
# - Configure resource limits based on expected users
# - Set public domain: jupyter.example.com

./scripts/setup.sh

# Deploy InformaticsClassroom (optional)
cd ~/ExCITE/InformaticsClassroom
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env

# Configure for distributed setup
# Edit .env:
# - Set WINTEHR_API_URL=https://wintehr.example.com:8888/fhir
# - Set BROADSEA_URL=https://broadsea.example.com
# - Configure database (PostgreSQL recommended over SQLite)

python3 app.py

# Configure firewall
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8000/tcp  # JupyterHub (or use reverse proxy)
sudo ufw enable
```

**Exposed Services**:
- JupyterHub: https://jupyter.example.com
- InformaticsClassroom: https://classroom.example.com

### Inter-Server Communication

**Network Requirements**:
- Servers should be on same VPC/VLAN or VPN-connected
- Firewall rules allowing internal traffic on required ports
- DNS resolution between servers (or /etc/hosts entries)

**Example Jupyter Notebook Configuration** (accessing WintEHR from Server 3):
```python
# In notebooks running on Server 3
import requests

# Use public URL or internal IP
fhir_base = "https://wintehr.example.com:8888/fhir"
# Or internal: "http://10.0.1.10:8888/fhir"

response = requests.get(f"{fhir_base}/Patient?_count=10")
```

---

## Strategy 3: Azure Cloud Production

**Use Case**: Enterprise deployment with Azure AD integration, managed services, high availability

**Total Cost Estimate**: $535-910/month

### Azure Architecture

```
Azure Resource Group: excite-platform-rg
Region: East US 2

├── WintEHR Resources
│   ├── VM: Standard_B2ms (2 vCPUs, 8GB RAM)
│   ├── Managed PostgreSQL: Basic tier (1-2 vCores)
│   ├── Public IP + DNS: wintehr.eastus2.cloudapp.azure.com
│   └── SSL: Let's Encrypt automatic
│
├── Broadsea Resources
│   ├── VM: Standard_D4s_v3 (4 vCPUs, 16GB RAM)
│   ├── Managed PostgreSQL: General Purpose (2-4 vCores)
│   └── Public IP + DNS: broadsea.eastus2.cloudapp.azure.com
│
├── JupyterHubClassroom Resources
│   ├── VM: Standard_D4s_v3 (4 vCPUs, 16GB RAM)
│   ├── Managed PostgreSQL: Basic tier
│   └── Public IP + DNS: jupyter.eastus2.cloudapp.azure.com
│
├── InformaticsClassroom Resources
│   ├── App Service: B2 (2 cores, 3.5GB RAM)
│   ├── Cosmos DB: Serverless (or 400-1000 RU/s)
│   ├── Blob Storage: Standard tier
│   ├── Azure AD: App registration
│   └── URL: informatics-classroom.azurewebsites.net
│
└── Shared Resources
    ├── VNet: 10.0.0.0/16 with 4 subnets
    ├── NSG: Security rules for each subnet
    ├── Azure Bastion: Secure SSH access
    └── Log Analytics: Centralized monitoring
```

### Step-by-Step Azure Deployment

#### 1. Prepare Azure Environment

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Create resource group
az group create \
  --name excite-platform-rg \
  --location eastus2

# Create VNet
az network vnet create \
  --resource-group excite-platform-rg \
  --name excite-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name wintehr-subnet \
  --subnet-prefix 10.0.1.0/24

# Create additional subnets
az network vnet subnet create \
  --resource-group excite-platform-rg \
  --vnet-name excite-vnet \
  --name broadsea-subnet \
  --address-prefix 10.0.2.0/24

az network vnet subnet create \
  --resource-group excite-platform-rg \
  --vnet-name excite-vnet \
  --name jupyter-subnet \
  --address-prefix 10.0.3.0/24

az network vnet subnet create \
  --resource-group excite-platform-rg \
  --vnet-name excite-vnet \
  --name classroom-subnet \
  --address-prefix 10.0.4.0/24
```

#### 2. Deploy WintEHR on Azure VM

```bash
# Create public IP
az network public-ip create \
  --resource-group excite-platform-rg \
  --name wintehr-pip \
  --dns-name wintehr-excite \
  --allocation-method Static

# Create NSG
az network nsg create \
  --resource-group excite-platform-rg \
  --name wintehr-nsg

# Add NSG rules
az network nsg rule create \
  --resource-group excite-platform-rg \
  --nsg-name wintehr-nsg \
  --name AllowHTTP \
  --priority 100 \
  --destination-port-ranges 80 \
  --protocol Tcp

az network nsg rule create \
  --resource-group excite-platform-rg \
  --nsg-name wintehr-nsg \
  --name AllowHTTPS \
  --priority 110 \
  --destination-port-ranges 443 \
  --protocol Tcp

az network nsg rule create \
  --resource-group excite-platform-rg \
  --nsg-name wintehr-nsg \
  --name AllowFHIR \
  --priority 120 \
  --destination-port-ranges 8888 \
  --protocol Tcp \
  --source-address-prefixes 10.0.0.0/16  # Internal only

# Create VM
az vm create \
  --resource-group excite-platform-rg \
  --name wintehr-vm \
  --image Ubuntu2204 \
  --size Standard_B2ms \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address wintehr-pip \
  --vnet-name excite-vnet \
  --subnet wintehr-subnet \
  --nsg wintehr-nsg

# SSH into VM and deploy WintEHR
ssh azureuser@wintehr-excite.eastus2.cloudapp.azure.com

# On VM: Install Docker and deploy
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker azureuser
exit

# SSH back in
ssh azureuser@wintehr-excite.eastus2.cloudapp.azure.com

# Clone and deploy
git clone <repo-url> ExCITE
cd ExCITE/WintEHR
cp config.example.yaml config.yaml

# Edit config for Azure
# - Set public_url: https://wintehr-excite.eastus2.cloudapp.azure.com
# - Configure Let's Encrypt for SSL

./deploy.sh
```

#### 3. Deploy Broadsea on Azure VM

```bash
# Create Managed PostgreSQL for Broadsea
az postgres flexible-server create \
  --resource-group excite-platform-rg \
  --name broadsea-postgres \
  --location eastus2 \
  --admin-user broadsea_admin \
  --admin-password '<strong-password>' \
  --sku-name Standard_B2s \
  --tier Burstable \
  --storage-size 128 \
  --version 14

# Create Broadsea VM (similar to WintEHR)
az vm create \
  --resource-group excite-platform-rg \
  --name broadsea-vm \
  --image Ubuntu2204 \
  --size Standard_D4s_v3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name excite-vnet \
  --subnet broadsea-subnet

# SSH and deploy Broadsea
ssh azureuser@<broadsea-vm-ip>

cd ExCITE/Broadsea
cp .env.example .env

# Edit .env to use Managed PostgreSQL
# POSTGRES_HOST=broadsea-postgres.postgres.database.azure.com
# POSTGRES_USER=broadsea_admin
# POSTGRES_PASSWORD=<password>

docker compose --profile atlas-from-image \
               --profile webapi-from-image up -d
```

#### 4. Deploy JupyterHubClassroom on Azure VM

```bash
# Create Managed PostgreSQL for JupyterHub
az postgres flexible-server create \
  --resource-group excite-platform-rg \
  --name jupyter-postgres \
  --location eastus2 \
  --admin-user jupyter_admin \
  --admin-password '<strong-password>' \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --storage-size 32

# Create JupyterHub VM
az vm create \
  --resource-group excite-platform-rg \
  --name jupyter-vm \
  --image Ubuntu2204 \
  --size Standard_D4s_v3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name excite-vnet \
  --subnet jupyter-subnet

# Deploy JupyterHub (similar to above)
```

#### 5. Deploy InformaticsClassroom on Azure App Service

```bash
# Create Cosmos DB
az cosmosdb create \
  --resource-group excite-platform-rg \
  --name informatics-cosmos \
  --kind MongoDB \
  --server-version 4.2

# Create Blob Storage
az storage account create \
  --resource-group excite-platform-rg \
  --name informaticsstorage \
  --location eastus2 \
  --sku Standard_LRS

# Create App Service Plan
az appservice plan create \
  --resource-group excite-platform-rg \
  --name informatics-plan \
  --location eastus2 \
  --sku B2 \
  --is-linux

# Create Web App
az webapp create \
  --resource-group excite-platform-rg \
  --plan informatics-plan \
  --name informatics-classroom \
  --runtime "PYTHON|3.11"

# Configure Azure AD (see PREREQUISITES.md for detailed steps)

# Deploy application
cd InformaticsClassroom
az webapp deploy \
  --resource-group excite-platform-rg \
  --name informatics-classroom \
  --src-path .
```

### Azure Cost Optimization

**Cost Breakdown** (monthly estimates):

| Resource | Configuration | Cost/Month |
|----------|--------------|------------|
| WintEHR VM | Standard_B2ms | ~$60 |
| WintEHR PostgreSQL | Basic 1 vCore | ~$40 |
| Broadsea VM | Standard_D4s_v3 | ~$140 |
| Broadsea PostgreSQL | GP 2 vCores | ~$120 |
| JupyterHub VM | Standard_D4s_v3 | ~$140 |
| JupyterHub PostgreSQL | Basic 1 vCore | ~$30 |
| InformaticsClassroom App | B2 | ~$60 |
| Cosmos DB | Serverless | ~$25-100 |
| Storage/Networking | Standard | ~$50 |
| **Total** | | **$535-910** |

**Optimization Strategies**:
1. **Reserved Instances**: 1-year commitment saves ~30%, 3-year saves ~60%
2. **Auto-Shutdown**: Configure VMs to stop during non-business hours
3. **Spot Instances**: Use for non-critical workloads (up to 90% savings)
4. **Managed Disk Optimization**: Use Standard SSD instead of Premium
5. **Cosmos DB Serverless**: Pay per request instead of provisioned throughput
6. **Azure Hybrid Benefit**: Use existing Windows Server licenses

---

## Module-Specific Deployment

### WintEHR Deployment Details

See [WintEHR/README.md](../WintEHR/README.md) for complete documentation.

**Key Configuration**:
- `config.yaml`: Patient cohort size, API keys, FHIR settings
- `docker-compose.yml`: Service orchestration
- `deploy.sh`: Automated deployment script

**Environment Variables**:
```bash
# Optional AI features
ANTHROPIC_API_KEY=sk-ant-...
TAVILY_API_KEY=tvly-...

# Database
POSTGRES_PASSWORD=<strong-password>

# Redis
REDIS_PASSWORD=<strong-password>
```

### Broadsea Deployment Details

See [Broadsea/README.md](../Broadsea/README.md) for complete documentation.

**Critical Profiles** (MUST use profiles or services won't start):
- `atlas-from-image`: Atlas web UI
- `webapi-from-image`: WebAPI backend
- `cdm-postprocessing`: Achilles analysis
- `solr-vocab-with-import`: Vocabulary search

**Environment Variables**:
```bash
WEBAPI_DATASOURCE_PASSWORD=<strong-password>
POSTGRES_PASSWORD=<strong-password>

# Optional: Authentication
SECURITY_PROVIDER=AtlasRegularSecurity  # Or DisabledSecurity for dev
SECURITY_AUTH_PROVIDER=ad  # ldap, oauth, openid, saml
```

### JupyterHubClassroom Deployment Details

See [JupyterHubClassroom/README.md](../JupyterHubClassroom/README.md) for complete documentation.

**Critical Requirements**:
- `JUPYTERHUB_CRYPT_KEY`: MUST be set (won't start without it)
- Docker socket access: Hub needs `/var/run/docker.sock` mounted
- Resource limits: Configure per-user CPU/RAM limits

**Environment Variables**:
```bash
JUPYTERHUB_CRYPT_KEY=$(openssl rand -hex 32)  # REQUIRED
POSTGRES_PASSWORD=<strong-password>
DOCKER_NOTEBOOK_CPU_LIMIT=2.0
DOCKER_NOTEBOOK_MEM_LIMIT=2G
```

### InformaticsClassroom Deployment Details

See [InformaticsClassroom/README.md](../InformaticsClassroom/README.md) for complete documentation.

**Database Options**:
- **Development**: SQLite (file-based, simple)
- **Production**: PostgreSQL or Cosmos DB

**Azure AD Configuration**:
See [PREREQUISITES.md](PREREQUISITES.md) for Azure AD app registration steps.

---

## Network Configuration

### Reverse Proxy with Nginx

For production deployments, use Nginx as reverse proxy to avoid port conflicts and enable SSL.

**Install Nginx**:
```bash
sudo apt install nginx certbot python3-certbot-nginx -y
```

**Configure Nginx** (`/etc/nginx/sites-available/excite`):
```nginx
# WintEHR
server {
    listen 80;
    server_name wintehr.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /fhir {
        proxy_pass http://localhost:8888;
        proxy_set_header Host $host;
    }
}

# Broadsea
server {
    listen 80;
    server_name broadsea.example.com;

    location / {
        proxy_pass http://localhost:80;
        proxy_set_header Host $host;
    }
}

# JupyterHub
server {
    listen 80;
    server_name jupyter.example.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

**Enable SSL with Let's Encrypt**:
```bash
sudo ln -s /etc/nginx/sites-available/excite /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Obtain certificates
sudo certbot --nginx -d wintehr.example.com
sudo certbot --nginx -d broadsea.example.com
sudo certbot --nginx -d jupyter.example.com
```

### Firewall Configuration

**Ubuntu/Debian (ufw)**:
```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable
```

**RHEL/CentOS (firewalld)**:
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

## Security Hardening

### SSL/TLS Configuration

**Let's Encrypt (Automated)**:
```bash
# For WintEHR, Broadsea, JupyterHub
certbot certonly --standalone -d wintehr.example.com
certbot certonly --standalone -d broadsea.example.com
certbot certonly --standalone -d jupyter.example.com

# Auto-renewal
sudo systemctl enable certbot.timer
```

**Custom Certificates**:
```bash
# Place certificates in each module
# WintEHR: ./ssl/cert.pem, ./ssl/key.pem
# Broadsea: ./secrets/traefik/cert.pem, ./secrets/traefik/key.pem
# JupyterHub: ./secrets/ssl/cert.pem, ./secrets/ssl/key.pem
```

### Secrets Management

**Best Practices**:
1. Never commit `.env` files to version control
2. Use strong passwords: `openssl rand -base64 32`
3. Rotate secrets regularly (every 90 days)
4. Use external secret managers (Azure Key Vault, HashiCorp Vault) for production

**Example: Azure Key Vault Integration**:
```bash
# Store secrets in Key Vault
az keyvault secret set \
  --vault-name excite-keyvault \
  --name wintehr-postgres-password \
  --value '<strong-password>'

# Retrieve in deployment scripts
export POSTGRES_PASSWORD=$(az keyvault secret show \
  --vault-name excite-keyvault \
  --name wintehr-postgres-password \
  --query value -o tsv)
```

### Authentication and Authorization

**Broadsea Security Providers**:
- `DisabledSecurity`: Development only
- `AtlasRegularSecurity`: Password-based authentication
- LDAP/Active Directory: Enterprise SSO
- OAuth: Google, GitHub, Facebook
- SAML: Identity providers

**JupyterHub Authentication**:
- `NativeAuthenticator`: Default (signup + approval)
- `LDAPAuthenticator`: Enterprise LDAP
- `OAuthenticator`: GitHub, Google, Azure AD

**InformaticsClassroom**:
- Development: Local accounts
- Production: Azure AD MSAL

---

## Monitoring and Observability

### Docker Container Monitoring

```bash
# Check all containers
docker ps -a

# View logs
docker compose logs -f <service>

# Resource usage
docker stats

# Inspect container
docker inspect <container-id>
```

### Application-Level Monitoring

**Prometheus + Grafana** (recommended for production):

```bash
# Create monitoring stack
cd ~/excite-monitoring
cat > docker-compose.yml <<EOF
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  prometheus-data:
  grafana-data:
EOF

# Start monitoring
docker compose up -d

# Access Grafana: http://localhost:3001 (admin/admin)
```

### Log Aggregation

**Centralized Logging with ELK Stack**:

```bash
# Install filebeat on each server
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.11.0-amd64.deb
sudo dpkg -i filebeat-8.11.0-amd64.deb

# Configure filebeat to ship logs to Elasticsearch
sudo nano /etc/filebeat/filebeat.yml

# Enable Docker module
sudo filebeat modules enable docker

# Start filebeat
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

---

## Backup and Disaster Recovery

### Database Backups

**PostgreSQL Automated Backups**:

```bash
# Create backup script
cat > ~/backup-postgres.sh <<'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR=/backups/postgres

mkdir -p $BACKUP_DIR

# WintEHR backup
docker exec wintehr-postgres pg_dump -U postgres wintehr > \
  $BACKUP_DIR/wintehr_$DATE.sql

# Broadsea backup
docker exec broadsea-postgres pg_dump -U postgres ohdsi > \
  $BACKUP_DIR/broadsea_$DATE.sql

# JupyterHub backup
docker exec jupyterhub-postgres pg_dump -U postgres jupyterhub > \
  $BACKUP_DIR/jupyterhub_$DATE.sql

# Compress backups
gzip $BACKUP_DIR/*_$DATE.sql

# Retain last 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
EOF

chmod +x ~/backup-postgres.sh

# Schedule daily backups
crontab -e
# Add: 0 2 * * * /home/user/backup-postgres.sh
```

**Azure Managed Database Backups**:
- Automatic: Azure takes daily backups (7-35 day retention)
- Manual: Point-in-time restore available
- Geo-redundant: Cross-region replication

### Disaster Recovery Plan

**RTO (Recovery Time Objective)**: 4 hours
**RPO (Recovery Point Objective)**: 24 hours (daily backups)

**Recovery Steps**:

1. **Database Restore**:
```bash
# Restore from backup
gunzip /backups/postgres/wintehr_20241108.sql.gz
docker exec -i wintehr-postgres psql -U postgres wintehr < \
  /backups/postgres/wintehr_20241108.sql
```

2. **Container Recreation**:
```bash
# Pull latest images
docker compose pull

# Recreate containers
docker compose up -d --force-recreate
```

3. **Verify Services**:
```bash
# Health checks
curl http://localhost:3000/health  # WintEHR
curl http://localhost:8000/hub/api  # JupyterHub
curl http://127.0.0.1/WebAPI/info   # Broadsea
```

---

## Performance Tuning

### PostgreSQL Optimization

**Recommended Settings** (edit `postgresql.conf`):

```ini
# Memory
shared_buffers = 2GB  # 25% of total RAM
effective_cache_size = 6GB  # 75% of total RAM
work_mem = 64MB
maintenance_work_mem = 512MB

# Checkpointing
checkpoint_completion_target = 0.9
wal_buffers = 16MB

# Query Planner
random_page_cost = 1.1  # For SSD
effective_io_concurrency = 200

# Connections
max_connections = 200
```

### Docker Performance

**Resource Limits** (docker-compose.yml):

```yaml
services:
  wintehr-backend:
    image: wintehr-backend:latest
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
```

### JupyterHub Performance

**Per-User Resource Limits**:

```python
# jupyterhub_config.py
c.DockerSpawner.cpu_limit = 2.0  # 2 CPU cores per user
c.DockerSpawner.mem_limit = '2G'  # 2GB RAM per user
c.DockerSpawner.cpu_guarantee = 0.5  # Guaranteed 0.5 cores
c.DockerSpawner.mem_guarantee = '512M'  # Guaranteed 512MB

# Cull idle servers
c.JupyterHub.services = [
    {
        'name': 'idle-culler',
        'admin': True,
        'command': [
            'python3', '-m', 'jupyterhub_idle_culler',
            '--timeout=3600',  # 1 hour
        ],
    }
]
```

---

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for detailed troubleshooting guides.

**Quick Diagnostics**:

```bash
# Check Docker daemon
sudo systemctl status docker

# Check disk space
df -h

# Check memory
free -h

# Check service logs
docker compose logs -f --tail=100 <service>

# Restart all services
docker compose restart
```

---

**Last Updated**: November 2025
**Next Review**: February 2026
