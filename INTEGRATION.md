# ExCITE Platform Integration Guide

Inter-module integration patterns, data flows, and technical implementation details for the ExCITE platform.

---

## Table of Contents

- [Integration Overview](#integration-overview)
- [Integration Pattern 1: Educational Assignment Flow](#integration-pattern-1-educational-assignment-flow)
- [Integration Pattern 2: Course Material Distribution](#integration-pattern-2-course-material-distribution)
- [Network Architecture](#network-architecture)
- [API Integration Details](#api-integration-details)
- [Authentication and Authorization](#authentication-and-authorization)
- [Data Flow Examples](#data-flow-examples)

---

## Integration Overview

The ExCITE platform is composed of five integrated modules that work together to provide a comprehensive healthcare informatics research and education environment.

### Module Dependency Graph

```
InformaticsLessons
      ↓ (git clone via nbgitpuller)
JupyterHubClassroom ←→ InformaticsClassroom (assignment links)
      ↓ (FHIR queries)
   WintEHR ←→ Broadsea (optional custom integration for research)
```

### Integration Technologies

| Integration | Technology | Purpose |
|-------------|-----------|---------|
| Notebook Distribution | nbgitpuller (Git + HTTP) | Clone/update course materials |
| FHIR Queries | REST API (HTTP/JSON) | Access patient data from notebooks |
| Assignment Management | HTTP links + redirects | Link LMS assignments to notebooks |
| Research Data** | Custom ETL (optional) | FHIR to OMOP integration if needed |

** Note: For custom FHIR-based research workflows, you can develop ETL pipelines using the FHIR APIs from WintEHR and OMOP CDM schemas in Broadsea based on your specific research needs.

---

## Integration Pattern 1: Educational Assignment Flow

**Purpose**: Link assignments in InformaticsClassroom to interactive Jupyter notebooks

### Workflow

```
1. Instructor creates assignment in InformaticsClassroom
   ↓
2. Assignment includes nbgitpuller URL to specific notebook
   ↓
3. Student clicks assignment link in InformaticsClassroom
   ↓
4. Browser redirects to JupyterHub with nbgitpuller parameters
   ↓
5. JupyterHub authenticates student (or redirects to login)
   ↓
6. nbgitpuller clones/updates InformaticsLessons repository
   ↓
7. JupyterHub opens specific notebook in student's workspace
   ↓
8. Student completes assignment in Jupyter notebook
   ↓
9. (Planned) Student submits notebook back to InformaticsClassroom
```

### Technical Implementation

#### Step 1: Create Assignment in InformaticsClassroom

**Instructor creates assignment** with metadata:

```python
# InformaticsClassroom backend (Flask)
assignment = Assignment(
    title="FHIR Patient Query Exercise",
    description="Query WintEHR for patients with diabetes",
    course_id=course.id,
    module_id=module.id,
    due_date=datetime(2024, 12, 31),
    points=100,
    # Link to Jupyter notebook
    notebook_url="http://jupyter.example.com/hub/user-redirect/git-pull?repo=https://github.com/UltraUB/InformaticsLessons&branch=main&subPath=lessons/week3/patient_queries.ipynb"
)
```

#### Step 2: Generate nbgitpuller URL

**nbgitpuller URL structure**:

```
http://<jupyterhub-url>/hub/user-redirect/git-pull?
  repo=<git-repo-url>&
  branch=<branch-name>&
  subPath=<path-to-notebook>&
  app=notebook  # or lab
```

**Example**:

```
http://jupyter.example.com/hub/user-redirect/git-pull?repo=https://github.com/UltraUB/InformaticsLessons&branch=main&subPath=lessons/week3/patient_queries.ipynb&app=lab
```

**URL Generator Tool**:

```python
def generate_nbgitpuller_url(
    jupyterhub_url,
    repo_url,
    branch="main",
    subpath="",
    app="lab"
):
    """Generate nbgitpuller URL for assignment linking"""
    from urllib.parse import urlencode

    params = {
        "repo": repo_url,
        "branch": branch,
        "subPath": subpath,
        "app": app
    }

    base_url = f"{jupyterhub_url}/hub/user-redirect/git-pull"
    return f"{base_url}?{urlencode(params)}"

# Usage
assignment_url = generate_nbgitpuller_url(
    jupyterhub_url="http://jupyter.example.com",
    repo_url="https://github.com/UltraUB/InformaticsLessons",
    branch="main",
    subpath="lessons/week3/patient_queries.ipynb",
    app="lab"
)
```

#### Step 3: Student Clicks Assignment

**Browser flow**:

1. Student in InformaticsClassroom clicks assignment link
2. InformaticsClassroom redirects to nbgitpuller URL
3. JupyterHub receives request at `/hub/user-redirect/git-pull`
4. If not authenticated, JupyterHub redirects to login
5. After auth, JupyterHub triggers nbgitpuller

#### Step 4: nbgitpuller Execution

**What nbgitpuller does**:

1. Check if repository exists in user workspace
2. If not, clone: `git clone <repo> InformaticsLessons/`
3. If exists, pull updates with smart merge:
   - Student changes preserved (never overwritten)
   - New instructor files added
   - Conflicts resolved automatically (student version wins)
4. Navigate to specified `subPath`
5. Open notebook in Jupyter Lab (or Notebook)

**Repository location in user workspace**:

```
/home/jovyan/InformaticsLessons/
  ├── lessons/
  │   ├── week1/
  │   ├── week2/
  │   ├── week3/
  │   │   └── patient_queries.ipynb  ← Opens this
  │   └── ...
  └── README.md
```

---

## Integration Pattern 2: Course Material Distribution

**Purpose**: Distribute and update course materials to all students efficiently

### Workflow

```
1. Instructor updates InformaticsLessons on GitHub
   ↓
2. Instructor shares nbgitpuller link with students
   ↓
3. Students click link (first time or for updates)
   ↓
4. nbgitpuller clones or updates repository
   ↓
5. Student work preserved, new materials added
   ↓
6. Students access latest course content
```

### Git Repository Structure

**InformaticsLessons Repository**:

```
InformaticsLessons/
├── README.md              # Course overview
├── lessons/
│   ├── week1_intro/
│   │   ├── 01_fhir_basics.ipynb
│   │   ├── 02_patient_resources.ipynb
│   │   └── README.md
│   ├── week2_search/
│   │   ├── 01_search_parameters.ipynb
│   │   ├── 02_advanced_queries.ipynb
│   │   └── exercises.ipynb
│   ├── week3_observations/
│   │   ├── 01_lab_results.ipynb
│   │   ├── 02_vital_signs.ipynb
│   │   └── patient_queries.ipynb
│   └── ...
├── assignments/
│   ├── assignment1.ipynb
│   ├── assignment2.ipynb
│   └── ...
├── solutions/             # Hidden from students
│   └── ...
└── .gitignore
```

### Conflict Resolution Strategy

**nbgitpuller Smart Merge Algorithm**:

1. **Student added new files**: Keep student files, add instructor files
2. **Both modified same file**: Keep student version (they win)
3. **Instructor added new files**: Add to student workspace
4. **Instructor deleted files**: Keep in student workspace (don't delete)

**Example Scenario**:

```
Student modified: lessons/week2/exercises.ipynb (student version kept)
Instructor added: lessons/week4/new_lesson.ipynb (added to workspace)
Instructor updated: lessons/week2/01_search_parameters.ipynb (student has old version, instructor's update added as conflict file)
```

**Result**:

```
lessons/week2/
├── exercises.ipynb (student's modified version)
├── 01_search_parameters.ipynb (student's version)
├── 01_search_parameters.ipynb.nbpuller-conflict (instructor's new version)
└── ...
lessons/week4/
└── new_lesson.ipynb (new from instructor)
```

---

## Network Architecture

### Single-Server Deployment

```
┌─────────────────────────────────────────┐
│         Docker Host (localhost)         │
│                                         │
│  ┌──────────────┐   ┌──────────────┐   │
│  │   WintEHR    │   │   Broadsea   │   │
│  │ :3000, :8888 │   │   :80, :443  │   │
│  └──────────────┘   └──────────────┘   │
│         ↑                               │
│         │ FHIR queries                  │
│         │ (host.docker.internal:8888)   │
│  ┌──────────────┐                       │
│  │ JupyterHub   │                       │
│  │    :8000     │                       │
│  │              │                       │
│  │ Student      │                       │
│  │ Containers   │                       │
│  └──────────────┘                       │
└─────────────────────────────────────────┘
```

**Key Network Configurations**:

1. **JupyterHub containers → WintEHR FHIR API**:
```python
# In notebooks
fhir_base = "http://host.docker.internal:8888/fhir"  # macOS/Windows
fhir_base = "http://172.17.0.1:8888/fhir"            # Linux
```

2. **All services → localhost** (from host machine)

### Distributed Deployment

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│  Server 1    │       │  Server 2    │       │  Server 3    │
│  WintEHR     │       │  Broadsea    │       │  JupyterHub  │
│ 10.0.1.10    │       │ 10.0.1.20    │       │ 10.0.1.30    │
└──────────────┘       └──────────────┘       └──────────────┘
       │                                              │
       │                                              │
       └──────────────────────────────────────────────┘
                 FHIR API (http://10.0.1.10:8888/fhir)
```

**Network Requirements**:

1. **VPC/VLAN**: All servers on same private network
2. **Internal DNS** (or `/etc/hosts`):
```
10.0.1.10  wintehr.local
10.0.1.20  broadsea.local
10.0.1.30  jupyter.local
```

3. **Firewall Rules**:
   - Allow 8888/tcp from JupyterHub server to WintEHR server
   - Allow 80/tcp, 443/tcp from public internet to all servers
   - Block direct database access (5432/tcp) from public

---

## API Integration Details

### WintEHR FHIR API Integration

**Accessing from Jupyter Notebooks**:

```python
# In JupyterHub notebook
import requests
import json

# FHIR server base URL
# - Single-server: host.docker.internal (Docker for Mac/Win) or 172.17.0.1 (Linux)
# - Distributed: http://wintehr.example.com:8888/fhir or http://10.0.1.10:8888/fhir
fhir_base = "http://host.docker.internal:8888/fhir"

# Search for patients
response = requests.get(f"{fhir_base}/Patient?_count=10")
patients = response.json()

# Get specific patient
patient_id = patients['entry'][0]['resource']['id']
patient = requests.get(f"{fhir_base}/Patient/{patient_id}").json()

# Search observations for patient
observations = requests.get(
    f"{fhir_base}/Observation",
    params={
        'patient': f'Patient/{patient_id}',
        'code': 'http://loinc.org|8867-4',  # Heart rate
        '_sort': '-date',
        '_count': 100
    }
).json()

# Extract values
for entry in observations.get('entry', []):
    obs = entry['resource']
    date = obs['effectiveDateTime']
    value = obs['valueQuantity']['value']
    unit = obs['valueQuantity']['unit']
    print(f"{date}: {value} {unit}")
```

**Helper Module for Students**:

```python
# fhir_helper.py (included in InformaticsLessons)
import requests
import os

class FHIRClient:
    def __init__(self, base_url=None):
        self.base_url = base_url or os.getenv('FHIR_BASE_URL', 'http://host.docker.internal:8888/fhir')

    def search(self, resource_type, params=None):
        """Search for FHIR resources"""
        url = f"{self.base_url}/{resource_type}"
        response = requests.get(url, params=params or {})
        response.raise_for_status()
        return response.json()

    def read(self, resource_type, resource_id):
        """Read a specific FHIR resource"""
        url = f"{self.base_url}/{resource_type}/{resource_id}"
        response = requests.get(url)
        response.raise_for_status()
        return response.json()

# Usage in notebooks
from fhir_helper import FHIRClient

client = FHIRClient()
patients = client.search('Patient', {'_count': 10})
```

### Broadsea WebAPI Integration (Optional)

**For advanced research workflows**:

```python
# Access Broadsea WebAPI from notebooks
import requests

webapi_base = "http://broadsea.example.com/WebAPI"

# Get source information
sources = requests.get(f"{webapi_base}/source/sources").json()

# Get concept information
concept_id = 313217  # Atrial fibrillation
concept = requests.get(f"{webapi_base}/vocabulary/{source_key}/concept/{concept_id}").json()
```

---

## Authentication and Authorization

### JupyterHub Authentication

**Current**: NativeAuthenticator (self-signup + admin approval)

**Integration with InformaticsClassroom** (Planned):

1. **Shared User Database**:
   - Both systems use same PostgreSQL user table
   - Username/password synchronized
   - User creation in InformaticsClassroom auto-creates JupyterHub account

2. **SSO via OAuth** (Future):
   - InformaticsClassroom as OAuth provider
   - JupyterHub OAuthenticator configured
   - Single login for both systems

### WintEHR API Authentication (Planned)

**Current**: Open access (development only)

**Future**: JWT-based authentication

```python
# Future API access pattern
import requests

# Login to get token
response = requests.post('http://wintehr.example.com:8000/auth/login', json={
    'username': 'student1',
    'password': 'password'
})
token = response.json()['access_token']

# Use token for FHIR queries
headers = {'Authorization': f'Bearer {token}'}
patients = requests.get('http://wintehr.example.com:8888/fhir/Patient', headers=headers).json()
```

---

## Data Flow Examples

### Example 1: Student Completes FHIR Assignment

```
1. Student opens InformaticsClassroom
   URL: http://classroom.example.com

2. Student navigates to Week 3 Assignment
   Clicks: "FHIR Patient Query Exercise"

3. Browser redirects to JupyterHub nbgitpuller URL
   URL: http://jupyter.example.com/hub/user-redirect/git-pull?repo=...

4. JupyterHub authenticates student
   NativeAuthenticator checks credentials

5. JupyterHub spawns user container
   DockerSpawner creates isolated container

6. nbgitpuller clones/updates InformaticsLessons
   Git operations preserve student work

7. Jupyter Lab opens specific notebook
   URL: http://jupyter.example.com/user/student1/lab/tree/InformaticsLessons/lessons/week3/patient_queries.ipynb

8. Student runs code cells in notebook
   Python kernel executes code

9. Notebook queries WintEHR FHIR API
   HTTP GET http://host.docker.internal:8888/fhir/Patient?condition=44054006

10. WintEHR HAPI FHIR processes query
    Returns JSON bundle with matching patients

11. Notebook processes and displays results
    Pandas DataFrame, matplotlib visualizations

12. Student completes assignment
    Saves notebook (auto-saved every 2 minutes)

13. (Planned) Student submits to InformaticsClassroom
    HTTP POST to classroom API with notebook content
```

### Example 2: Instructor Updates Course Materials

```
1. Instructor updates InformaticsLessons repository
   Adds new notebook: lessons/week5/medication_analysis.ipynb
   Commits and pushes to GitHub

2. Instructor shares update link with students
   Sends nbgitpuller URL via email or InformaticsClassroom announcement

3. Student clicks link
   Browser navigates to nbgitpuller URL

4. nbgitpuller pulls latest changes
   Git operations:
   - Student's modified files preserved
   - New week5 folder added
   - Updated notebooks merged intelligently

5. Student accesses new materials
   New notebook appears in file browser
```

### Example 3: Research Workflow (Optional Custom Integration)

```
1. Researcher generates cohort in Broadsea Atlas
   Defines inclusion criteria for diabetes patients

2. (Custom ETL) Export FHIR resources from WintEHR
   Query patients matching criteria
   Extract FHIR resources (Patient, Condition, Observation, etc.)

3. (Custom ETL) Transform to OMOP CDM
   Map FHIR resources to OMOP tables
   Load into Broadsea PostgreSQL

4. Run analysis in Jupyter notebook
   Connect to both WintEHR FHIR API and Broadsea database
   Compare FHIR and OMOP representations

5. Generate research findings
   Statistical analysis, visualizations
   Export results for publication
```

**Note**: Custom FHIR to OMOP integration requires development based on specific research needs. You can use FHIR APIs from WintEHR and OMOP CDM schemas in Broadsea to build tailored ETL pipelines.

---

## Troubleshooting Integration Issues

### Issue 1: Notebooks Can't Reach WintEHR FHIR API

**Symptoms**:
- `Connection refused` or `Name or service not known`
- HTTP request timeouts

**Solutions**:

```python
# Try different base URLs

# Option 1: host.docker.internal (Docker for Mac/Windows)
fhir_base = "http://host.docker.internal:8888/fhir"

# Option 2: Docker gateway IP (Linux)
fhir_base = "http://172.17.0.1:8888/fhir"

# Option 3: Host IP address
import socket
host_ip = socket.gethostbyname(socket.gethostname())
fhir_base = f"http://{host_ip}:8888/fhir"

# Option 4: External URL (distributed deployment)
fhir_base = "http://wintehr.example.com:8888/fhir"
```

### Issue 2: nbgitpuller Conflicts

**Symptoms**:
- `.nbpuller-conflict` files appearing
- Student work overwritten (shouldn't happen)

**Solutions**:

1. **Conflicts are normal**: Instructor updates create conflict files, student versions preserved
2. **Review conflicts**: Compare `.ipynb` and `.ipynb.nbpuller-conflict`
3. **Manual merge**: Copy desired content from conflict file if needed

### Issue 3: Assignment Links Not Working

**Symptoms**:
- 404 errors after clicking assignment link
- JupyterHub doesn't open notebook

**Debugging**:

```bash
# Check nbgitpuller URL structure
# Should be: /hub/user-redirect/git-pull?repo=<url>&branch=<branch>&subPath=<path>

# Verify repository URL is accessible
git clone <repo-url>

# Check branch name is correct
git ls-remote <repo-url> | grep <branch-name>

# Verify subPath exists in repository
ls -la <repo-path>/<subPath>
```

---

**Last Updated**: November 2025
**Next Review**: February 2026
