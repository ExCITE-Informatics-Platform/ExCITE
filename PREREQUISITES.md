# ExCITE Platform Prerequisites

Complete guide to setup requirements, external services, API keys, and configuration for the ExCITE platform.

---

## Table of Contents

- [System Requirements](#system-requirements)
- [Software Dependencies](#software-dependencies)
- [External Services and API Keys](#external-services-and-api-keys)
- [Data Requirements](#data-requirements)
- [Network Requirements](#network-requirements)
- [Azure-Specific Prerequisites](#azure-specific-prerequisites)

---

## System Requirements

### Hardware Requirements

**Single-Server Development**:
- CPU: 12 cores minimum, 24 cores recommended
- RAM: 32GB minimum, 56GB recommended
- Disk: 140GB minimum (SSD recommended), 500GB recommended
- Network: 1Gbps internal, 100Mbps external

**Distributed Deployment** (3 servers):
- **Server 1 (WintEHR)**: 4 CPU cores, 16GB RAM, 150GB disk
- **Server 2 (Broadsea)**: 8 CPU cores, 16GB RAM, 150GB disk
- **Server 3 (JupyterHub)**: 4 CPU cores, 8GB RAM, 100GB disk
- Network: 10Gbps internal recommended

**Azure Cloud**:
- See [DEPLOYMENT.md](DEPLOYMENT.md#strategy-3-azure-cloud-production) for VM sizing details

### Operating System

**Supported Operating Systems**:
- Ubuntu 22.04 LTS (recommended)
- Ubuntu 20.04 LTS
- Debian 11+ (Bullseye)
- RHEL 8+ / CentOS Stream 8+
- macOS 13+ (Ventura) with Docker Desktop
- Windows 10/11 with WSL2 and Docker Desktop

**Kernel Requirements** (Linux):
- Kernel 5.4+ for Docker support
- cgroups v2 for resource limits
- Namespaces support for container isolation

---

## Software Dependencies

### Core Requirements (All Deployments)

#### Docker Engine

**Version**: >= 28.3.2

**Installation (Ubuntu/Debian)**:
```bash
# Add Docker's official GPG key
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
# Expected: Docker version 28.3.2 or higher
```

**Installation (macOS/Windows)**:
- Download Docker Desktop from https://www.docker.com/products/docker-desktop/
- Install and launch Docker Desktop
- Verify in terminal: `docker --version`

#### Docker Compose

**Version**: V2 (included with Docker Desktop and recent Docker Engine)

**Verification**:
```bash
docker compose version
# Expected: Docker Compose version v2.x.x or higher
```

**Note**: Older `docker-compose` (V1, hyphenated) is **not supported**. Use `docker compose` (V2, space-separated).

#### Git

**Version**: Latest stable version

**Installation**:
```bash
# Ubuntu/Debian
sudo apt install git -y

# macOS (via Homebrew)
brew install git

# Verify
git --version
```

### Module-Specific Requirements

#### Python (for InformaticsClassroom)

**Version**: 3.9+ (3.11 recommended)

**Installation**:
```bash
# Ubuntu/Debian
sudo apt install python3.11 python3.11-venv python3-pip -y

# macOS (via Homebrew)
brew install python@3.11

# Verify
python3.11 --version
# Expected: Python 3.11.x
```

**Virtual Environment**:
```bash
# Create venv for InformaticsClassroom
cd InformaticsClassroom
python3.11 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install --upgrade pip
```

#### Node.js (for InformaticsClassroom Frontend)

**Version**: 18+ (20 LTS recommended)

**Installation**:
```bash
# Ubuntu/Debian (via NodeSource)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# macOS (via Homebrew)
brew install node@20

# Verify
node --version  # Expected: v20.x.x
npm --version   # Expected: 9.x.x or 10.x.x
```

---

## External Services and API Keys

### Required Services

#### 1. JupyterHub Crypt Key

**Purpose**: Session encryption and security for JupyterHub

**Generation**:
```bash
openssl rand -hex 32
```

**Configuration**:
```bash
# In JupyterHubClassroom/.env
JUPYTERHUB_CRYPT_KEY=<generated-key>
```

**Critical**: JupyterHub will **not start** without this key.

#### 2. OMOP Vocabularies (Broadsea)

**Purpose**: Required for OHDSI Atlas cohort definition and vocabulary search

**Source**: https://athena.ohdsi.org/

**Setup Process**:
1. Create free account at Athena
2. Select vocabularies:
   - SNOMED CT (required)
   - LOINC (required)
   - RxNorm (required)
   - ICD-10-CM (recommended)
   - CPT-4 (recommended)
   - Additional vocabularies as needed
3. Download bundle (5-10GB compressed)
4. Extract to `Broadsea/omop_vocab/` directory
5. Load via: `docker compose --profile omop-vocab-pg-load up`

**License**: Free for research/education, requires Athena account

**Note**: Vocabulary loading is a one-time operation taking 10-30 minutes.

### Optional Services (Enhanced Features)

#### 3. Azure Active Directory (InformaticsClassroom Production)

**Purpose**: Enterprise single sign-on (SSO) and user management

**Required Information**:
- **Tenant ID**: Azure AD tenant identifier
- **Client ID**: Application (client) ID
- **Client Secret**: Application secret value
- **Redirect URIs**: Configured callback URLs

**Setup Process**:

1. **Register Application** in Azure Portal:
```
Navigate to: Azure Active Directory → App registrations → New registration

Application Name: InformaticsClassroom
Supported Account Types: Single tenant (or Multi-tenant)
Redirect URI:
  - Type: Web
  - URL: https://classroom.example.com/auth/callback
```

2. **Create Client Secret**:
```
App registration → Certificates & secrets → New client secret
Description: InformaticsClassroom Production
Expires: 24 months (or custom)
Copy Value immediately (shown only once)
```

3. **Configure API Permissions**:
```
App registration → API permissions → Add permission
Microsoft Graph → Delegated permissions → Select:
  - User.Read (Sign in and read user profile)
  - Email
  - Profile
  - OpenId

Grant admin consent for organization
```

4. **Note Configuration Values**:
```
Application (client) ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Directory (tenant) ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Client Secret Value: xxx~xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Configuration in InformaticsClassroom**:
```bash
# In InformaticsClassroom/.env
AZURE_TENANT_ID=your-tenant-id
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret
REDIRECT_URI=https://classroom.example.com/auth/callback
```

**Documentation**: https://learn.microsoft.com/azure/active-directory/develop/quickstart-register-app

#### 6. SSL/TLS Certificates (Production Deployments)

**Purpose**: HTTPS encryption for secure communication

**Option A - Let's Encrypt (Free, Automated)**:

**Requirements**:
- Public domain name with DNS configured
- Ports 80 and 443 publicly accessible
- Email address for renewal notifications

**Installation**:
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain certificate (with Nginx)
sudo certbot --nginx -d wintehr.example.com

# Obtain certificate (standalone, no Nginx)
sudo certbot certonly --standalone -d wintehr.example.com

# Auto-renewal (enabled by default)
sudo systemctl enable certbot.timer
```

**Configuration**:
Certbot automatically configures Nginx. For manual configuration, certificates are placed in:
```
/etc/letsencrypt/live/wintehr.example.com/fullchain.pem
/etc/letsencrypt/live/wintehr.example.com/privkey.pem
```

**Option B - Custom Certificates**:

**If you have your own certificates** (purchased or organizational):

```bash
# Place certificates in module-specific locations:

# WintEHR
./WintEHR/ssl/cert.pem
./WintEHR/ssl/key.pem

# Broadsea (Traefik)
./Broadsea/secrets/traefik/cert.pem
./Broadsea/secrets/traefik/key.pem

# JupyterHubClassroom
./JupyterHubClassroom/secrets/ssl/cert.pem
./JupyterHubClassroom/secrets/ssl/key.pem
```

---

## Data Requirements

### Synthetic Patient Data (WintEHR)

**Source**: Synthea synthetic patient generator (auto-downloaded by WintEHR)

**Default Configuration**:
- 122 patients
- 103,000+ FHIR resources
- Massachusetts population demographics
- 10-year medical histories

**Customization** (in `WintEHR/config.yaml`):
```yaml
synthea:
  patient_count: 100  # Adjust number of patients
  state: "Massachusetts"  # Or other US states
  city: "Boston"  # Or other cities
```

**Disk Space**:
- 50 patients: ~500MB
- 100 patients: ~1GB
- 500 patients: ~5GB
- 1000 patients: ~10GB

**Generation Time**:
- 50 patients: ~5 minutes
- 100 patients: ~10 minutes
- 500 patients: ~30 minutes

### OMOP Vocabularies (Broadsea)

**See [Required Services - OMOP Vocabularies](#2-omop-vocabularies-broadsea)**

---

## Network Requirements

### Port Availability

Ensure the following ports are not in use:

| Module | Port | Protocol | Purpose |
|--------|------|----------|---------|
| WintEHR | 3000 | HTTP | Frontend UI |
| WintEHR | 8000 | HTTP | Backend API |
| WintEHR | 8888 | HTTP | FHIR Server (HAPI) |
| WintEHR | 5432 | TCP | PostgreSQL |
| WintEHR | 6379 | TCP | Redis |
| Broadsea | 80 | HTTP | Traefik (HTTP) |
| Broadsea | 443 | HTTPS | Traefik (HTTPS) |
| Broadsea | 5432 | TCP | PostgreSQL |
| Broadsea | 8983 | HTTP | SOLR |
| JupyterHub | 8000 | HTTP | JupyterHub UI |
| JupyterHub | 5433 | TCP | PostgreSQL (alt port) |
| InformaticsClassroom | 5001 | HTTP | Backend API |
| InformaticsClassroom | 5173 | HTTP | Frontend (Vite dev) |

**Port Conflict Resolution**:

If WintEHR backend (8000) conflicts with JupyterHub (8000):

```bash
# Option 1: Change JupyterHub port
echo "JUPYTERHUB_PORT=8001" >> JupyterHubClassroom/.env

# Option 2: Change WintEHR port
# Edit WintEHR docker-compose.yml or config.yaml
```

### Firewall Configuration

**Development** (localhost only):
- No firewall configuration needed
- All services accessible on localhost

**Production** (external access):

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

**Azure Network Security Groups (NSG)**:
See [Azure-Specific Prerequisites](#azure-specific-prerequisites).

### DNS Configuration (Production)

**If deploying with public access**:

1. Register domain names (or subdomains):
   - wintehr.example.com
   - broadsea.example.com
   - jupyter.example.com
   - classroom.example.com

2. Configure A records pointing to server IPs:
```
wintehr.example.com    A    203.0.113.10
broadsea.example.com   A    203.0.113.20
jupyter.example.com    A    203.0.113.30
classroom.example.com  A    203.0.113.40
```

3. For Azure: Use auto-assigned DNS names:
```
wintehr.eastus2.cloudapp.azure.com
broadsea.eastus2.cloudapp.azure.com
```

---

## Azure-Specific Prerequisites

### Azure CLI

**Installation**:
```bash
# Ubuntu/Debian
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# macOS
brew install azure-cli

# Verify
az --version
```

**Login**:
```bash
az login
# Opens browser for authentication
```

### Azure Subscription

**Requirements**:
- Active Azure subscription
- Sufficient quota for VMs and managed services
- Billing configured

**Verify Quota**:
```bash
# Check VM quota in region
az vm list-usage --location eastus2 --output table
```

### Azure Resource Group

**Create Resource Group**:
```bash
az group create \
  --name excite-platform-rg \
  --location eastus2
```

### Azure Virtual Network

**Create VNet** (if not using default):
```bash
az network vnet create \
  --resource-group excite-platform-rg \
  --name excite-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name default-subnet \
  --subnet-prefix 10.0.1.0/24
```

### Azure Active Directory

**See [Optional Services - Azure Active Directory](#5-azure-active-directory-informaticsclassroom-production)**

---

## Verification Checklist

Before starting deployment, verify:

### System Verification

- [ ] Operating system is supported version
- [ ] Sufficient disk space available
- [ ] Sufficient RAM available
- [ ] CPU meets minimum requirements

### Software Verification

- [ ] Docker Engine >= 28.3.2 installed
- [ ] Docker Compose V2 available
- [ ] Git installed
- [ ] Python 3.11 installed (if using InformaticsClassroom)
- [ ] Node.js 18+ installed (if using InformaticsClassroom)

### Service Verification

- [ ] JUPYTERHUB_CRYPT_KEY generated
- [ ] OMOP Vocabularies downloaded (for Broadsea)
- [ ] Anthropic API key obtained (if using AI features)
- [ ] Tavily API key obtained (if using web research)
- [ ] Azure AD app registered (if using Azure AD auth)
- [ ] SSL certificates obtained or Let's Encrypt configured

### Network Verification

- [ ] Required ports available
- [ ] Firewall rules configured (for production)
- [ ] DNS records configured (for production)
- [ ] Network connectivity between servers (distributed)

### Security Verification

- [ ] Strong passwords generated for databases
- [ ] Secrets not committed to version control
- [ ] SSL/TLS configured for production
- [ ] Backup strategy in place

---

## Quick Setup Scripts

### Generate All Secrets

```bash
#!/bin/bash
# generate-secrets.sh - Generate all required secrets

echo "Generating ExCITE platform secrets..."

# JupyterHub
echo "JUPYTERHUB_CRYPT_KEY=$(openssl rand -hex 32)" > JupyterHubClassroom/.env

# Databases
echo "POSTGRES_PASSWORD=$(openssl rand -base64 32)" >> JupyterHubClassroom/.env
echo "POSTGRES_PASSWORD=$(openssl rand -base64 32)" > Broadsea/secrets/WEBAPI_DATASOURCE_PASSWORD
echo "POSTGRES_PASSWORD=$(openssl rand -base64 32)" > WintEHR/.env

# InformaticsClassroom
echo "SECRET_KEY=$(openssl rand -hex 32)" > InformaticsClassroom/.env

echo "Secrets generated! Store these files securely and do not commit to Git."
```

### Verify Prerequisites

```bash
#!/bin/bash
# verify-prerequisites.sh - Check all prerequisites

echo "=== ExCITE Platform Prerequisites Check ==="

# Docker
if command -v docker &> /dev/null; then
    DOCKER_VERSION=$(docker --version | cut -d' ' -f3 | cut -d',' -f1)
    echo "✓ Docker: $DOCKER_VERSION"
else
    echo "✗ Docker not found"
fi

# Docker Compose
if command -v docker compose &> /dev/null; then
    COMPOSE_VERSION=$(docker compose version | cut -d' ' -f4)
    echo "✓ Docker Compose: $COMPOSE_VERSION"
else
    echo "✗ Docker Compose not found"
fi

# Git
if command -v git &> /dev/null; then
    GIT_VERSION=$(git --version | cut -d' ' -f3)
    echo "✓ Git: $GIT_VERSION"
else
    echo "✗ Git not found"
fi

# Python
if command -v python3.11 &> /dev/null; then
    PYTHON_VERSION=$(python3.11 --version | cut -d' ' -f2)
    echo "✓ Python: $PYTHON_VERSION"
else
    echo "⚠ Python 3.11 not found (required for InformaticsClassroom)"
fi

# Node.js
if command -v node &> /dev/null; then
    NODE_VERSION=$(node --version)
    echo "✓ Node.js: $NODE_VERSION"
else
    echo "⚠ Node.js not found (required for InformaticsClassroom)"
fi

# Check disk space
AVAILABLE_SPACE=$(df -h . | tail -1 | awk '{print $4}')
echo "Available Disk Space: $AVAILABLE_SPACE"

# Check RAM
TOTAL_RAM=$(free -h | grep Mem | awk '{print $2}')
echo "Total RAM: $TOTAL_RAM"

echo "=== Prerequisites check complete ==="
```

---

**Last Updated**: November 2025
**Next Review**: February 2026
