# ExCITE - Educators’ Clinical Informatics Training Environment​

**A comprehensive platform for biomedical informatics education and research**

**Last Updated**: November 2025

---

## Overview

The **ExCITE** ecosystem is an integrated collection of biomedical informatics platforms designed to support education, research, and clinical decision support. The system combines FHIR-native electronic health records, OHDSI observational research tools, interactive learning environments, and comprehensive educational management into a cohesive platform.

### Core Capabilities

- 🏥 **Electronic Health Records** - Complete FHIR R4-native EHR system with clinical workflows
- 🔬 **Observational Research** - OHDSI platform for population health analytics
- 📚 **Interactive Education** - Multi-user Jupyter environments with course materials
- 🎓 **Learning Management** - Role-based classroom platform with quizzes and assignments
- 🔗 **Integrated Workflows** - Seamless connections between EHR, research, and education modules

---

## Ecosystem Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        ExCITE Ecosystem                         │
│                   Integrated Platform Components                │
└─────────────────────────────────────────────────────────────────┘

        CLINICAL DATA SOURCE                  EDUCATIONAL PLATFORMS
┌──────────────────────────────────┐   ┌──────────────────────────────┐
│          WintEHR                 │   │   InformaticsClassroom       │
│   FHIR-Native EHR System         │   │  Learning Management System  │
│   • 122 Synthetic Patients       │   │  • Role-Based Access         │
│   • 103K+ FHIR Resources         │   │  • Interactive Quizzes       │
│   • Clinical Workflows           │   │  • Azure AD Integration      │
│   • CDS Hooks 2.0                │   │  • Progress Tracking         │
│   • Medical Imaging (DICOM)      │   │                              │
│   Port: 3000 (UI), 8888 (FHIR)   │   │  Port: 5001 (API), 5173 (UI) │
└──────────────────────────────────┘   └──────────────────────────────┘
            │                                       │
            │ FHIR Data                             │ Assignment
            │ Queries                               │ Management
            ↓                                       ↓
┌──────────────────────────────────┐   ┌──────────────────────────────┐
│          Broadsea                │   │    JupyterHubClassroom       │
│   OHDSI Research Platform        │   │  Multi-User Jupyter Platform │
│   • Atlas Cohort Definition      │   │  • NativeAuthenticator       │
│   • Achilles Data Quality        │   │  • Per-User Containers       │
│   • HADES R Analytics            │   │  • Resource Management       │
│   • WebAPI Services              │   │  • nbgitpuller Integration   │
│   • OMOP CDM Database            │   │                              │
│   • Vocabulary Search (SOLR)     │   │  Port: 8000                  │
│                                  │   └──────────────────────────────┘
│   Port: 80/443 (Traefik)         │               │
└──────────────────────────────────┘               │ Course
            ↑                                      │ Materials
            │ OMOP Queries                         ↓
            │ (Optional)               ┌──────────────────────────────┐
            └──────────────────────────│    InformaticsCourses        │
                                       │  Course Materials Repository │
                                       │  • FHIR Assessments          │
                                       │  • CDS Assignments           │
                                       │  • Jupyter Notebooks         │
                                       │                              │
                                       │  Git: github.com/ultraub     │
                                       └──────────────────────────────┘

        RESEARCH WORKFLOWS                    EDUCATIONAL WORKFLOWS
```

---

## Platform Components

### 1. WintEHR - FHIR-Native Electronic Health Record

**Purpose**: Complete FHIR R4-native EHR system for clinical informatics education

**Key Features**:
- Clinical workflows (chart review, orders, results, pharmacy, imaging)
- 38+ FHIR resource types with full CRUD operations
- Medical imaging with DICOM viewer (CT, MR, XR, US, DX, CR, MG)
- Clinical Decision Support (CDS Hooks 2.0)
- Real-time WebSocket updates
- 122 synthetic patients with 103K+ resources

**Technology**: React + FastAPI + HAPI FHIR + PostgreSQL + Redis

**Deployment**: Docker Compose (5 containers)

**Resource Requirements**:
- **Minimum**: 2 CPU cores, 8GB RAM, 20GB disk
- **Recommended**: 4 CPU cores, 16GB RAM, 100GB SSD

**Access**: http://localhost:3000 (UI), http://localhost:8888/fhir (FHIR server)

**Documentation**: [WintEHR/README.md](WintEHR/README.md)

---

### 2. Broadsea - OHDSI Observational Research Platform

**Purpose**: OHDSI platform for population health analytics and cohort studies

**Key Features**:
- Atlas web UI for cohort definition
- WebAPI REST services for OHDSI tools
- RStudio Server with HADES R packages
- Achilles data characterization
- Data Quality Dashboard (DQD)
- OMOP Common Data Model (CDM) integration
- SOLR vocabulary search

**Technology**: Docker Compose + Traefik + PostgreSQL + SOLR + Java Spring Boot

**Deployment**: Docker Compose with profile-based activation (6-13 containers)

**Resource Requirements**:
- **Minimum**: 2 CPU cores, 4GB RAM, 10GB disk
- **Recommended**: 4 CPU cores, 8GB RAM, 50GB SSD (with vocabularies)

**Access**: http://127.0.0.1/atlas (Atlas UI), http://127.0.0.1/WebAPI (API)

**Documentation**: [Broadsea/README.md](Broadsea/README.md)

---

### 3. JupyterHubClassroom - Multi-User Jupyter Environment

**Purpose**: Production-ready JupyterHub for educational notebook environments

**Key Features**:
- Self-service student registration with instructor approval
- Per-user isolated Docker containers
- Persistent workspaces (per-user volumes)
- nbgitpuller course material distribution
- Resource limits (CPU, memory) per user
- Pre-configured for InformaticsCourses

**Technology**: JupyterHub + DockerSpawner + PostgreSQL + NativeAuthenticator

**Deployment**: Docker Compose (2 core services + dynamic user containers)

**Resource Requirements**:
- **Minimum**: 4 CPU cores, 8GB RAM, 50GB disk (5-8 concurrent users)
- **Recommended**: 8 CPU cores, 16GB RAM, 200GB SSD (10-15 concurrent users)
- **Capacity Planning**: Total RAM / 2GB per user = Max concurrent users

**Access**: http://localhost:8000

**Documentation**: [JupyterHubClassroom/README.md](JupyterHubClassroom/README.md)

---

### 4. InformaticsClassroom - Learning Management System

**Purpose**: Educational platform for teaching biomedical informatics courses

**Key Features**:
- Role-based access (Student, TA, Instructor, Admin)
- Interactive quizzes with real-time feedback
- Class and module management
- Azure AD authentication (production)
- Permission matrix for fine-grained control
- Audit trail for compliance

**Technology**: Flask (backend) + React (frontend) + PostgreSQL/Cosmos DB

**Deployment**: Local development or Azure App Service

**Resource Requirements**:
- **Minimum**: 2 CPU cores, 4GB RAM, 10GB disk
- **Recommended**: 4 CPU cores, 8GB RAM, 50GB SSD
- **Azure**: B2 App Service, Serverless Cosmos DB

**Access**: http://localhost:5001 (API), http://localhost:5173 (React UI)

**Documentation**: [InformaticsClassroom/README.md](InformaticsClassroom/README.md)

---

### 5. InformaticsCourses - Course Materials Repository

**Purpose**: Educational content repository for medical informatics courses

**Key Features**:
- Jupyter notebooks with FHIR assessments
- CDS (Clinical Decision Support) assignments
- Version-controlled lesson materials
- nbgitpuller integration with JupyterHub

**Technology**: Jupyter Notebooks (.ipynb) + Git

**Deployment**: Git repository distribution via nbgitpuller

**Resource Requirements**: Minimal (cloned repository ~50MB)

**Current Lessons**:
- FHIR Assessment 1k, 2q, 3k (FHIR basics, queries, advanced)
- CDS Assessment 1h, 2d, 3e (CDS introduction, development, evaluation)

**Access**: https://github.com/ultraub/InformaticsCourses

**Documentation**: [InformaticsCourses/README.md](InformaticsCourses/README.md)

---

## Integration Patterns

### 1. Educational Assignment Flow

**Workflow**: InformaticsClassroom → JupyterHubClassroom → InformaticsCourses → WintEHR/Broadsea

**Data Flow**:
1. **InformaticsClassroom** instructor assigns FHIR lesson
2. Student logs into **JupyterHubClassroom**
3. Clicks nbgitpuller link for **InformaticsCourses**
4. Materials cloned to student workspace
5. Student queries **WintEHR** FHIR API from notebook
6. Completes assignment, downloads for submission
7. **InformaticsClassroom** tracks grades

**Use Cases**:
- FHIR API training
- Clinical data analysis exercises
- Hands-on informatics education

---

### 2. Course Material Distribution

**Workflow**: InformaticsCourses → JupyterHubClassroom

**Data Flow**:
1. **InformaticsCourses** GitHub repository updated
2. Instructor generates nbgitpuller link
3. Students access **JupyterHubClassroom**
4. Click link → materials auto-cloned to workspace
5. Students work on lessons (changes persist)
6. Instructor updates materials
7. Students click link again → smart merge (preserves student changes)

**Use Cases**:
- Seamless lesson distribution
- Version-controlled course materials
- Automatic content updates

---

## Deployment Strategies

### Strategy 1: Single-Server Development (All Components)

**Use Case**: Local development, testing, small classroom (<10 students)

**Requirements**:
- **Minimum**: 16GB RAM, 8 CPU cores, 200GB SSD
- **Recommended**: 32GB RAM, 16 CPU cores, 500GB SSD

**Deployment Order**:
```bash
# 1. Start WintEHR (generates patient data)
cd WintEHR && ./deploy.sh

# 2. Start Broadsea (OHDSI platform)
cd ../Broadsea
cp .env.example .env
# Edit .env: Configure database, secrets
docker compose --profile atlas-from-image --profile webapi-from-image up -d

# 3. Start JupyterHubClassroom
cd ../JupyterHubClassroom
cp .env.example .env
# Edit .env: Set passwords, generate CRYPT_KEY
./scripts/setup.sh

# 4. Start InformaticsClassroom (optional)
cd ../InformaticsClassroom
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python3 app.py
```

**Port Allocation**:
- 3000: WintEHR UI
- 8000: JupyterHub, WintEHR Backend
- 8888: WintEHR HAPI FHIR
- 80/443: Broadsea (Traefik)
- 5001: InformaticsClassroom API
- 5173: InformaticsClassroom React UI

**⚠️ Port Conflicts**: WintEHR backend (8000) conflicts with JupyterHub (8000)
- **Solution**: Run on different machines or change JupyterHub port to 8001

---

### Strategy 2: Distributed Deployment (Recommended for Production)

**Use Case**: Production classroom (>10 students), research deployment

**Server Distribution**:

**Server 1: Clinical Data (WintEHR)**
- **Resources**: 4 CPU cores, 16GB RAM, 100GB SSD
- **Services**: WintEHR (all 5 containers)
- **Ports**: 3000 (UI), 8000 (backend), 8888 (FHIR)

**Server 2: Research Platform (Broadsea)**
- **Resources**: 4 CPU cores, 8GB RAM, 100GB SSD
- **Services**: Broadsea (6-13 containers)
- **Ports**: 80/443 (Traefik)

**Server 3: Education (JupyterHub + InformaticsClassroom)**
- **Resources**: 8 CPU cores, 16GB RAM, 200GB SSD
- **Services**: JupyterHub, InformaticsClassroom
- **Ports**: 8000 (JupyterHub), 5001/5173 (Classroom)

**Network Configuration**:
- Server 1 FHIR endpoint accessible to Server 3 (http://server1:8888/fhir)
- Server 3 JupyterHub references Server 1 for FHIR lessons
- Server 2 Broadsea can optionally connect to Server 1 for FHIR-based research (requires custom integration)

---

### Strategy 3: Azure Cloud Deployment (Fully Managed)

**Use Case**: Production deployment with Azure AD integration

**Azure Resources**:

**WintEHR**:
- VM: Standard_B2ms (2 vCPUs, 8GB RAM)
- Managed PostgreSQL: Basic tier (1-2 vCores)
- Public IP with DNS: wintehr.eastus2.cloudapp.azure.com
- SSL: Let's Encrypt automatic

**Broadsea**:
- VM: Standard_D4s_v3 (4 vCPUs, 16GB RAM)
- Managed PostgreSQL: General Purpose (2-4 vCores)
- Public IP with DNS: broadsea.eastus2.cloudapp.azure.com

**JupyterHubClassroom**:
- VM: Standard_D4s_v3 (4 vCPUs, 16GB RAM) for 10-15 users
- Managed PostgreSQL: Basic tier
- Public IP with DNS: jupyter.eastus2.cloudapp.azure.com

**InformaticsClassroom**:
- App Service: B2 (2 cores, 3.5GB RAM)
- Cosmos DB: Serverless or 400-1000 RU/s
- Blob Storage: Standard tier
- Azure AD: App registration for authentication
- URL: informatics-classroom.azurewebsites.net

**Networking**:
- VNet peering between all resources
- NSG rules: Ports 80, 443 (public), 5432 (VNet only)
- Azure Bastion for SSH access

**Cost Estimate** (monthly):
- VMs: ~$250-400
- Managed Databases: ~$150-300
- App Service: ~$60
- Cosmos DB: ~$25-100
- Storage/Networking: ~$50
- **Total**: ~$535-910/month

---

## Resource Planning

### Aggregate Resource Requirements

| Deployment | CPU Cores | RAM | Disk | Max Users |
|------------|-----------|-----|------|-----------|
| **Development (all)** | 12 | 32GB | 140GB | 5-8 |
| **Recommended (all)** | 24 | 56GB | 500GB | 15-20 |
| **Distributed (3 servers)** | 16 | 40GB | 400GB | 20-30 |
| **Azure Production** | 14 (vCPUs) | 40GB | 400GB | 30-50 |

### Individual Module Resources

| Module | Min CPU | Min RAM | Min Disk | Recommended CPU | Recommended RAM | Recommended Disk |
|--------|---------|---------|----------|-----------------|-----------------|------------------|
| **Broadsea** | 2 | 4GB | 10GB | 4 | 8GB | 50GB |
| **InformaticsClassroom** | 2 | 4GB | 10GB | 4 | 8GB | 50GB |
| **InformaticsCourses** | - | - | 50MB | - | - | 50MB |
| **JupyterHubClassroom** | 4 | 8GB | 50GB | 8 | 16GB | 200GB |
| **WintEHR** | 2 | 8GB | 20GB | 4 | 16GB | 100GB |

---

## Setup Prerequisites

### Required Software

**All Deployments**:
- **Docker**: >= 28.3.2
- **Docker Compose**: V2 (included with Docker Desktop)
- **Git**: Latest version
- **Operating System**: Linux, macOS, or Windows with WSL2

**Python Modules** (InformaticsClassroom backend):
- **Python**: 3.9+ (3.11 recommended)
- **pip**: Latest version
- **venv**: Virtual environment management

**Node.js** (InformaticsClassroom frontend):
- **Node.js**: 18+ (20 recommended)
- **npm**: 9+ (10 recommended)

### API Keys and External Services

**Required**:

1. **Azure Active Directory** (InformaticsClassroom production):
   - **Tenant ID**: Azure AD tenant
   - **App Registration**: Client ID and secret
   - **Redirect URIs**: Configured for MSAL authentication
   - **Microsoft Graph API**: User.Read permission
   - **Setup Guide**: https://learn.microsoft.com/azure/active-directory/develop/quickstart-register-app

2. **JUPYTERHUB_CRYPT_KEY** (JupyterHubClassroom):
   - **Generate**: `openssl rand -hex 32`
   - **Purpose**: Session encryption and security
   - **Required**: Yes (JupyterHub won't start without it)

**Optional**:

1. **Anthropic Claude API** (WintEHR AI features):
   - **API Key**: From https://console.anthropic.com/
   - **Purpose**: AI-powered clinical notes, decision support
   - **Required**: No (AI features disabled without key)

2. **Tavily API** (WintEHR web research):
   - **API Key**: From https://tavily.com/
   - **Purpose**: Web research for clinical references
   - **Required**: No

3. **Let's Encrypt SSL** (production deployments):
   - **Email**: For certificate notifications
   - **Domain**: Public DNS required (Azure: auto-assigned)
   - **Port 80/443**: Must be publicly accessible
   - **Required**: No (dev uses HTTP)

4. **Authentication Providers** (Broadsea optional):
   - **LDAP/Active Directory**: Enterprise auth
   - **OAuth**: Google, GitHub, Facebook
   - **OpenID Connect**: SSO providers
   - **SAML**: Identity providers
   - **Required**: No (default: DisabledSecurity)

### External Data Requirements

1. **OMOP Vocabularies** (Broadsea - Required for research):
   - **Source**: https://athena.ohdsi.org/
   - **Size**: ~5-10GB uncompressed
   - **Contents**: SNOMED CT, LOINC, RxNorm, ICD-10, CPT-4
   - **License**: Athena account required (free)
   - **Setup**: Download, extract to ./omop_vocab, load via Broadsea scripts

2. **Synthea Synthetic Patients** (WintEHR - Auto-generated):
   - **Source**: Synthea JAR (auto-downloaded)
   - **Size**: ~1-5GB for 50-100 patients
   - **Contents**: Realistic patient cohorts with encounters, medications, observations
   - **Setup**: Automatic via WintEHR deploy.sh

---

## Quick Start Guide

### Minimal Educational Setup (No Research)

For teaching FHIR basics without OMOP research:

```bash
# 1. WintEHR (FHIR data source)
cd WintEHR
cp config.example.yaml config.yaml
./deploy.sh
# Access: http://localhost:3000

# 2. JupyterHubClassroom (student notebooks)
cd ../JupyterHubClassroom
cp .env.example .env
echo "JUPYTERHUB_CRYPT_KEY=$(openssl rand -hex 32)" >> .env
echo "POSTGRES_PASSWORD=$(openssl rand -base64 32)" >> .env
./scripts/setup.sh
# Access: http://localhost:8000

# 3. Verify
# - WintEHR: http://localhost:3000 (should show EHR login)
# - JupyterHub: http://localhost:8000 (should show signup page)
# - InformaticsCourses: Pre-configured in JupyterHub
```

**Students can now**:
- Login to JupyterHub
- Access FHIR lessons via nbgitpuller
- Query WintEHR at http://host.docker.internal:8888/fhir

---

### Complete Platform Setup (All Modules)

For full integrated platform deployment:

```bash
# 1. WintEHR (clinical data)
cd WintEHR && ./deploy.sh

# 2. Broadsea (OHDSI platform)
cd ../Broadsea
cp .env.example .env
# Edit .env: Set WEBAPI_DATASOURCE_PASSWORD
docker compose --profile atlas-from-image --profile webapi-from-image \
  --profile cdm-postprocessing --profile solr-vocab-with-import up -d

# 3. Load OMOP Vocabularies
# Download from https://athena.ohdsi.org/ → extract to ./omop_vocab
docker compose --profile omop-vocab-pg-load up

# 4. JupyterHubClassroom
cd ../JupyterHubClassroom
./scripts/setup.sh

# 5. InformaticsClassroom (optional)
cd ../InformaticsClassroom
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python3 app.py

# 6. Verify
# - WintEHR: http://localhost:3000
# - Broadsea Atlas: http://127.0.0.1/atlas
# - JupyterHub: http://localhost:8000
# - Classroom: http://localhost:5001
```

**Note**: For custom FHIR to OMOP integration workflows, you can develop custom ETL pipelines based on your specific research needs using the FHIR APIs from WintEHR and the OMOP CDM schemas in Broadsea.

---

## Common Issues and Solutions

### Port Conflicts

**Problem**: WintEHR backend (8000) conflicts with JupyterHub (8000)

**Solution**:
```bash
# Option 1: Change JupyterHub port
cd JupyterHubClassroom
echo "JUPYTERHUB_PORT=8001" >> .env
docker compose down && docker compose up -d

# Option 2: Run on different machines (recommended for production)
```

---

### WintEHR ↔ JupyterHub Networking

**Problem**: Notebooks can't reach http://localhost:8888/fhir from inside containers

**Solution**:
```python
# In Jupyter notebooks, use Docker host networking
import requests

# Docker for Mac/Windows
fhir_base = "http://host.docker.internal:8888/fhir"

# Docker on Linux
fhir_base = "http://172.17.0.1:8888/fhir"

# Or use external IP/hostname
fhir_base = "http://<server-ip>:8888/fhir"
```

---

### OMOP Vocabulary Not Found

**Problem**: Broadsea Atlas shows "Vocabulary not loaded"

**Solution**:
```bash
cd Broadsea
# Download vocabularies from https://athena.ohdsi.org/
# Extract to ./omop_vocab directory
docker compose --profile omop-vocab-pg-load up
# Wait for load to complete (10-30 minutes depending on vocabulary size)
```

---

### JupyterHub Spawner Fails

**Problem**: "Server failed to start" when launching notebook

**Solution**:
```bash
cd JupyterHubClassroom

# Check Docker socket access
docker compose exec hub ls -la /var/run/docker.sock

# Rebuild notebook image
docker build -f Dockerfile.notebook -t jupyterhub-notebook:latest .

# Check logs
docker compose logs -f hub | grep -i spawn

# Verify resource availability
docker stats  # Ensure enough RAM/CPU
```

---

## Documentation Index

### Platform-Level Documentation

Comprehensive guides for the integrated ExCITE platform:

- **[Deployment Guide](ExCITE/DEPLOYMENT.md)** - Detailed deployment strategies for single-server, distributed, and Azure cloud
- **[Features Guide](ExCITE/FEATURES.md)** - Complete platform capabilities and features across all modules
- **[Integration Guide](ExCITE/INTEGRATION.md)** - Inter-module integration patterns and data flows
- **[Prerequisites Guide](ExCITE/PREREQUISITES.md)** - Setup requirements, API keys, and external services
- **[Contributing Guide](ExCITE/CONTRIBUTING.md)** - How to contribute to the ExCITE project

### Module-Specific Documentation

Individual module documentation and configuration:

- **[Broadsea](Broadsea/README.md)** - OHDSI platform deployment and configuration
- **[InformaticsClassroom](InformaticsClassroom/README.md)** - Learning management system
- **[InformaticsCourses](InformaticsCourses/README.md)** - Course materials repository
- **[JupyterHubClassroom](JupyterHubClassroom/README.md)** - JupyterHub deployment and user management
- **[WintEHR](WintEHR/README.md)** - FHIR EHR system and synthetic patient data

---

## Support and Community

### Getting Help

1. **Check module-specific documentation** in each subdirectory
2. **Review troubleshooting guides** for common issues
3. **Check logs** with `docker compose logs -f <service>`
4. **Verify prerequisites** (Docker, API keys, ports)

### External Resources

- **OHDSI Forums**: https://forums.ohdsi.org/
- **FHIR Documentation**: https://hl7.org/fhir/
- **JupyterHub Documentation**: https://jupyterhub.readthedocs.io/
- **Docker Documentation**: https://docs.docker.com/

---

## License

This integrated platform combines multiple open-source projects, each with their own licenses:

- **JupyterHub**: BSD 3-Clause License
- **OHDSI Broadsea**: Apache 2.0 License
- **HAPI FHIR**: Apache 2.0 License
- **Docker**: Apache 2.0 License
- **ExCITE Platform Integration**: See individual module licenses

---

## Acknowledgments

The ExCITE platform integrates work from:
- OHDSI Collaborative (Observational Health Data Sciences and Informatics)
- HL7 FHIR Community (Fast Healthcare Interoperability Resources)
- Project Jupyter (Interactive computing)
- Synthea (Synthetic patient generation)

---

**Last Updated**: November 2025
**Maintained by**: ExCITE Project Team
**Version**: 1.0.0
