# ExCITE Platform Features

Comprehensive guide to capabilities and features across the integrated ExCITE healthcare informatics platform.

---

## Table of Contents

- [Platform Overview](#platform-overview)
- [WintEHR Features](#wintehr-features)
- [Broadsea Features](#broadsea-features)
- [JupyterHubClassroom Features](#jupyterhubclassroom-features)
- [InformaticsClassroom Features](#informaticsclassroom-features)
- [InformaticsLessons Features](#informaticslessons-features)
- [Integration Features](#integration-features)
- [Educational Features](#educational-features)
- [Research Features](#research-features)
- [Technical Features](#technical-features)

---

## Platform Overview

The ExCITE platform provides an integrated ecosystem for healthcare informatics education and research:

### Core Capabilities

🏥 **Clinical Data Management**
- FHIR R4-native electronic health record system
- Synthetic patient data generation (Synthea)
- 122 realistic patient cohorts with complete medical histories
- 103,000+ FHIR resources across 50+ resource types

🔬 **Observational Research**
- OMOP Common Data Model (CDM) integration
- OHDSI research tools (Atlas, Achilles, HADES)
- Cohort definition and characterization
- Data quality assessment and visualization

📚 **Interactive Education**
- Multi-user Jupyter notebook environment
- FHIR and CDS Hooks course materials
- Learning management system with quizzes
- Real-time progress tracking and assessment

🔗 **Integrated Workflows**
- Seamless data flow between EHR and research platforms
- Course material distribution via nbgitpuller
- FHIR query integration with notebooks
- Unified authentication and access control

---

## WintEHR Features

**FHIR-Native Electronic Health Record System**

### Patient Management

**Synthetic Patient Generation**:
- Powered by Synthea open-source patient generator
- Realistic demographic distributions (age, gender, race, ethnicity)
- Complete medical histories with encounters, conditions, medications
- Configurable cohort sizes (10-1000+ patients)
- Geographic diversity (ZIP codes, addresses)

**Patient Demographics**:
- Full FHIR Patient resources with identifiers
- Contact information (phone, email, address)
- Emergency contacts and relationships
- Preferred language and communication preferences
- Multiple identifier systems (MRN, SSN, driver's license)

**Patient Search and Filtering**:
- Search by name, DOB, identifier
- Filter by age range, gender, condition
- Sort by various demographics
- Pagination for large patient lists

### Clinical Workflows

**Encounter Management**:
- Ambulatory visits (office, telehealth)
- Emergency department encounters
- Inpatient hospitalizations
- Encounter history and timeline
- Provider assignments and notes

**Condition/Problem List**:
- Active and resolved conditions
- ICD-10-CM coded diagnoses
- SNOMED CT clinical concepts
- Onset dates and verification status
- Clinical status tracking (active, recurrence, remission, resolved)

**Medication Management**:
- Active medications list
- Medication history and timeline
- RxNorm coded prescriptions
- Dosage instructions (sig)
- Medication reconciliation
- Drug-drug interaction checking (planned)

**Allergy and Intolerance Tracking**:
- Drug allergies
- Food allergies
- Environmental allergies
- Severity classification (mild, moderate, severe, life-threatening)
- Reaction manifestations
- Verification status

**Laboratory Results**:
- LOINC-coded lab orders and results
- Reference ranges and interpretations
- Trend visualization over time
- Critical value flagging
- Specimen tracking

**Vital Signs Monitoring**:
- Blood pressure, heart rate, temperature
- Respiratory rate, oxygen saturation
- Height, weight, BMI calculation
- Vital signs trends and charting
- Pediatric growth charts

**Imaging and Diagnostics**:
- Radiology orders and reports
- DICOM integration (planned)
- Imaging study metadata
- Diagnostic report narratives

### FHIR API Capabilities

**FHIR R4 Server** (HAPI FHIR):
- Full FHIR R4 specification compliance
- RESTful API with JSON/XML support
- 50+ FHIR resource types supported
- Search parameters for all resources
- _include and _revinclude support for resource graph queries
- CapabilityStatement for API discovery

**Search Capabilities**:
```
# Example searches
GET /fhir/Patient?name=Smith
GET /fhir/Observation?code=http://loinc.org|8867-4&date=ge2023-01-01
GET /fhir/Condition?patient=Patient/123&clinical-status=active
GET /fhir/MedicationRequest?patient=Patient/123&status=active
GET /fhir/Encounter?patient=Patient/123&_sort=-date
```

**Resource Operations**:
- Create (POST): Add new resources
- Read (GET): Retrieve by ID
- Update (PUT): Modify existing resources
- Delete (DELETE): Remove resources
- Patch (PATCH): Partial updates
- Search (GET): Query resources

**Bulk Data Export** (FHIR Bulk Data):
- Patient-level export
- Group-level export
- System-level export
- NDJSON format for large datasets

### Clinical Decision Support

**CDS Hooks 2.0 Integration**:
- Patient view hook
- Order select hook
- Order sign hook
- Appointment book hook

**AI-Powered Features** (with Anthropic Claude API):
- Clinical note generation and summarization
- Differential diagnosis suggestions
- Treatment plan recommendations
- Patient education material generation
- Medical literature search integration (with Tavily API)

**Drug Interaction Checking**:
- Real-time medication interaction alerts
- Severity classification
- Alternative medication suggestions
- Evidence-based recommendations

### Medical Imaging

**DICOM Support** (Planned):
- Image upload and storage
- DICOM metadata extraction
- Image viewing (Orthanc integration)
- PACS integration capabilities

### Security and Privacy

**HIPAA Compliance Features**:
- Role-based access control (RBAC)
- Audit logging for all data access
- Data encryption at rest and in transit
- Patient consent management
- Access control lists per resource

**Authentication**:
- JWT-based authentication
- Session management
- Password policies
- Multi-factor authentication (planned)

### User Interface

**Modern React Frontend**:
- Responsive design (desktop, tablet, mobile)
- Material-UI components
- Real-time updates
- Accessibility (WCAG 2.1 AA)
- Dark mode support

**Dashboard Views**:
- Patient summary dashboard
- Clinical timeline visualization
- Medication and allergy lists
- Vital signs trends
- Upcoming appointments

### Technical Specifications

**Technology Stack**:
- Frontend: React 18, Material-UI, React Router
- Backend: FastAPI (Python), Pydantic validation
- FHIR Server: HAPI FHIR 6.x
- Database: PostgreSQL 14+
- Cache: Redis 7+
- AI: Anthropic Claude 3.5 Sonnet

**Performance**:
- <200ms API response time (95th percentile)
- 10,000+ concurrent users supported (with appropriate scaling)
- 1M+ FHIR resources capacity
- Real-time search with sub-second latency

**Data Volume**:
- 122 synthetic patients (default)
- 103,000+ FHIR resources
- 50+ resource types
- Scalable to millions of resources

---

## Broadsea Features

**OHDSI Observational Research Platform**

### Atlas - Cohort Definition Tool

**Cohort Building**:
- Drag-and-drop interface for cohort criteria
- Complex inclusion/exclusion logic
- Temporal relationships between events
- Drug exposure criteria (ingredients, classes, specific drugs)
- Condition occurrence (diagnoses, clinical findings)
- Procedure occurrence (surgeries, interventions)
- Measurement criteria (lab values, vital signs)
- Visit context (inpatient, outpatient, ED)

**Cohort Characterization**:
- Demographic distributions (age, gender, race)
- Clinical feature analysis
- Medication usage patterns
- Comorbidity analysis
- Healthcare utilization metrics

**Cohort Comparison**:
- Side-by-side cohort visualization
- Feature comparison tables
- Statistical significance testing
- Propensity score matching

### WebAPI - Backend Services

**Cohort Generation**:
- Execute cohort definitions across CDMs
- Generate cohort membership tables
- Track generation status and errors
- Store generated cohort statistics

**Vocabulary Services**:
- Concept search across vocabularies
- Ancestor/descendant concept relationships
- Concept set management
- Concept mapping between vocabularies

**Source Management**:
- Multiple CDM source configuration
- Source priority for queries
- Source-specific vocabulary versions
- Data quality metrics per source

### Achilles - Data Quality Assessment

**Descriptive Statistics**:
- Person demographics summary
- Observation period coverage
- Death statistics
- Care site and provider counts

**Data Density Visualizations**:
- Record counts over time
- Data quality dashboard
- Completeness heatmaps
- Temporal data availability

**Data Quality Rules**:
- Plausibility checks
- Conformance validation
- Completeness assessment
- Threshold-based quality gates

### HADES - R Analytics

**CohortMethod**:
- Large-scale propensity score analysis
- Comparative effectiveness studies
- Safety surveillance
- Outcome prediction

**PatientLevelPrediction**:
- Machine learning model development
- Risk prediction models
- Model validation and evaluation
- External validation support

**FeatureExtraction**:
- Automated covariate construction
- Temporal feature engineering
- Aggregated feature generation
- Feature selection utilities

### Vocabulary Management

**Supported Vocabularies**:
- SNOMED CT: Clinical terminology
- LOINC: Laboratory observations
- RxNorm: Medications
- ICD-10-CM: Diagnoses (US)
- ICD-10-PCS: Procedures (US)
- CPT-4: Procedural terminology
- NDC: National Drug Codes
- ATC: Anatomical Therapeutic Chemical
- Read: UK clinical codes
- OPCS4: UK procedure codes

**Vocabulary Features**:
- SOLR-powered fast search
- Concept relationships (is-a, maps-to, etc.)
- Standard vs source concept mapping
- Vocabulary version management
- Concept synonyms and translations

### Security and Access Control

**Authentication Options**:
- DisabledSecurity (development only)
- AtlasRegularSecurity (username/password)
- LDAP/Active Directory integration
- OAuth 2.0 (Google, GitHub, Facebook)
- OpenID Connect
- SAML 2.0 identity providers

**Authorization**:
- Role-based permissions
- Cohort sharing and privacy
- Source-level access control
- Feature-level permissions

### Technical Specifications

**Technology Stack**:
- Frontend: AngularJS (Atlas)
- Backend: Java Spring Boot (WebAPI)
- Database: PostgreSQL with OMOP CDM schema
- Search: Apache SOLR
- Reverse Proxy: Traefik

**OMOP CDM Support**:
- CDM v5.3, v5.4
- 39 standardized tables
- Person, Death, Observation Period
- Condition, Drug, Procedure, Measurement
- Visit, Provider, Care Site
- Payer Plan, Cost

**Performance**:
- 100M+ person database capacity
- Complex cohort generation in minutes
- Real-time vocabulary search
- Scalable R analytics

---

## JupyterHubClassroom Features

**Multi-User Interactive Computing Environment**

### User Management

**NativeAuthenticator**:
- Self-service user signup
- Admin approval workflow
- Password strength enforcement
- User enable/disable controls
- Bulk user management

**Admin Dashboard**:
- User list with status
- Pending signup approvals
- Server management per user
- Resource usage monitoring
- User activity logs

**Authentication Options**:
- NativeAuthenticator (default)
- LDAPAuthenticator (enterprise)
- OAuthenticator (GitHub, Google, Azure AD)
- FirstUseAuthenticator (simple auto-approve)
- Custom authenticators via plugins

### Notebook Environment

**DockerSpawner - Per-User Isolation**:
- Individual containers per user
- Persistent user volumes
- Resource limits per user (CPU, RAM)
- Custom Docker images per user/group
- Network isolation between users

**Pre-Installed Packages**:
- **Data Science**: pandas, numpy, scipy, scikit-learn
- **Visualization**: matplotlib, seaborn, plotly
- **Healthcare**: fhirclient, fhir.resources
- **Utilities**: requests, beautifulsoup4, lxml
- **Notebooks**: jupyter, jupyterlab, ipywidgets

**Jupyter Lab Interface**:
- Code editor with syntax highlighting
- Terminal access
- File browser with upload/download
- Extension manager
- Dark mode theme

### Course Material Distribution

**nbgitpuller Integration**:
- One-click course material distribution
- Automatic Git repository cloning
- Smart conflict resolution (student work preserved)
- Branch switching and updates
- Custom subpath navigation

**InformaticsLessons Pre-Configuration**:
- Lessons auto-available on first login
- Organized directory structure
- README with instructions
- Sample FHIR query notebooks
- CDS Hooks assignment templates

**nbgitpuller URL Generator**:
```
http://localhost:8000/hub/user-redirect/git-pull?repo=<repo-url>&branch=<branch>&subPath=<path>
```

### Resource Management

**Per-User Limits**:
```python
CPU Limit: 2.0 cores (configurable)
Memory Limit: 2GB (configurable)
Disk Limit: 10GB (configurable)
Concurrent Notebooks: 5
```

**Server Culling**:
- Idle server shutdown (configurable timeout)
- Resource reclamation
- Automatic cleanup
- User notification before shutdown

**Resource Monitoring**:
- Real-time CPU/RAM usage per user
- Disk space tracking
- Active notebook counts
- Total resource consumption

### Collaboration Features

**Shared Notebooks**:
- Share via public URL
- nbviewer integration
- Export to HTML/PDF
- Version control with Git

**Group Workspaces**:
- Shared volumes for teams
- Group-specific environments
- Collaborative editing (via extensions)

### Integration Features

**WintEHR FHIR Access**:
- Pre-configured FHIR client
- Network routing to WintEHR
- Sample query notebooks
- Helper functions for common queries

**Data Persistence**:
- User work auto-saved
- Persistent volumes across sessions
- Backup and restore capabilities
- Export notebooks and data

### Security

**Container Isolation**:
- User namespaces
- Limited capabilities (no privileged operations)
- Resource quotas enforced
- Network segmentation

**Authentication Security**:
- Secure cookie storage
- HTTPS enforcement (production)
- CSRF protection
- Session timeout controls

### Technical Specifications

**Technology Stack**:
- JupyterHub 4.x
- DockerSpawner
- NativeAuthenticator
- PostgreSQL (user database)
- Docker API for container management

**Scalability**:
- 10-15 users (single server - 4CPU/8GB)
- 20-50 users (single server - 8CPU/16GB)
- 100+ users (Kubernetes deployment)

**Performance**:
- <30s container spawn time
- <5s notebook kernel start
- Concurrent user support based on resources

---

## InformaticsClassroom Features

**Learning Management System for Healthcare Informatics**

### Course Management

**Course Creation**:
- Rich text editor for content
- Multi-week course structure
- Module organization
- Prerequisites and dependencies
- Publish/draft status control

**Content Types**:
- Video embeds (YouTube, Vimeo)
- Reading materials (PDF, links)
- Interactive quizzes
- Coding assignments (Jupyter notebooks)
- Discussion forums (planned)

**Course Catalog**:
- Browse available courses
- Search and filter
- Course descriptions and syllabi
- Instructor information
- Enrollment statistics

### Assessment and Grading

**Quiz Features**:
- Multiple choice questions
- True/false questions
- Short answer (auto-graded via keywords)
- Code submission (manual grading)
- Randomized question order
- Time limits per quiz
- Attempt limits and retries

**Grading**:
- Automatic grading for MCQ
- Manual grading interface for instructors
- Grade distribution visualization
- Individual student progress tracking
- Grade export (CSV, Excel)

**Progress Tracking**:
- Course completion percentage
- Module completion status
- Quiz scores and history
- Time spent on materials
- Learning analytics dashboard

### User Roles

**Student**:
- Course enrollment
- Material access
- Quiz submission
- Progress viewing
- Certificate download (planned)

**Instructor**:
- Course creation and editing
- Student management
- Grade management
- Content publishing
- Analytics viewing

**Admin**:
- User management
- System configuration
- Course approval workflow
- Platform analytics
- Backup and maintenance

### Azure AD Integration

**Microsoft Authentication**:
- MSAL (Microsoft Authentication Library)
- Single sign-on (SSO)
- Organizational account support
- Azure AD groups mapping to roles
- Multi-tenant support (planned)

**User Provisioning**:
- Automatic account creation from Azure AD
- Profile sync (name, email, photo)
- Group membership sync
- License assignment integration

### Analytics and Reporting

**Student Analytics**:
- Course completion rates
- Average quiz scores
- Time to completion
- Drop-off points
- Learning path visualization

**Instructor Analytics**:
- Enrollment trends
- Content engagement metrics
- Assessment difficulty analysis
- Student cohort performance
- Intervention recommendations

**Admin Analytics**:
- Platform usage statistics
- Storage and resource utilization
- User growth metrics
- Course popularity
- System health monitoring

### Technical Specifications

**Technology Stack**:
- Backend: Flask (Python), SQLAlchemy
- Frontend: React, Material-UI
- Database: PostgreSQL (production), Cosmos DB (Azure), SQLite (dev)
- Storage: Azure Blob Storage (production), local filesystem (dev)
- Auth: Azure AD MSAL, JWT

**Deployment Options**:
- Local (Flask dev server)
- Docker container
- Azure App Service
- Kubernetes (planned)

---

## InformaticsLessons Features

**Healthcare Informatics Course Materials Repository**

### Course Content

**FHIR Fundamentals**:
- Introduction to FHIR R4
- Resource types and structure
- RESTful API concepts
- Search parameters
- Patient, Observation, Condition, MedicationRequest resources
- Bundle construction
- References and contained resources

**FHIR Query Assignments**:
- Patient search and filtering
- Observation data extraction
- Medication history analysis
- Condition timeline construction
- Multi-resource queries with _include
- Pagination and sorting

**Clinical Decision Support**:
- CDS Hooks 2.0 specification
- Hook types (patient-view, order-select, order-sign)
- Card construction and actions
- Prefetch optimization
- Testing CDS services

**Advanced Topics**:
- SMART on FHIR app development
- Bulk Data export
- FHIR subscriptions
- Custom operations
- Terminology services (ValueSet, CodeSystem)

### Notebook Structure

**Self-Contained Learning**:
- Markdown explanations
- Code examples
- Exercises with solutions
- Expected output samples
- Common error troubleshooting

**Progressive Difficulty**:
- Beginner: Basic FHIR queries
- Intermediate: Complex searches, data transformation
- Advanced: CDS Hooks, SMART apps, analytics

**Interactive Elements**:
- Widgets for parameter adjustment
- Visualization of FHIR resources
- Interactive diagrams
- Quiz-style validation

### Git Integration

**Version Control**:
- Git repository on GitHub
- Branch for each course version
- Pull requests for updates
- Community contributions welcome
- Issue tracker for bugs/improvements

**nbgitpuller Compatibility**:
- Optimized for classroom distribution
- Conflict-free updates (student work preserved)
- One-click access via JupyterHub
- Automatic latest version pulling

### Assessment Integration

**InformaticsClassroom Linking**:
- Notebooks linked to LMS assignments
- Auto-submission from notebooks (planned)
- Grade passback (planned)
- Progress tracking

**Autograding Support**:
- nbgrader compatibility (planned)
- Test cases embedded in notebooks
- Automatic scoring
- Feedback generation

---

## Integration Features

### FHIR Query Integration (WintEHR ↔ JupyterHub)

**Network Routing**:
- JupyterHub containers access WintEHR FHIR API
- Docker host networking (host.docker.internal)
- Configurable FHIR base URL
- API key authentication (planned)

**Sample Query Notebook**:
```python
import requests

fhir_base = "http://host.docker.internal:8888/fhir"

# Search patients
patients = requests.get(f"{fhir_base}/Patient?_count=10").json()

# Get specific patient
patient = requests.get(f"{fhir_base}/Patient/123").json()

# Search observations
obs = requests.get(f"{fhir_base}/Observation?patient=Patient/123&code=8867-4").json()
```

### Educational Assignment Flow (InformaticsClassroom ↔ JupyterHub)

**Workflow**:
1. Instructor creates assignment in InformaticsClassroom
2. Assignment links to Jupyter notebook in InformaticsLessons
3. Student clicks assignment link → nbgitpuller URL
4. JupyterHub opens notebook with course materials
5. Student completes assignment in notebook
6. (Planned) Submit back to InformaticsClassroom for grading

### Course Material Distribution (InformaticsLessons ↔ JupyterHub)

**nbgitpuller Workflow**:
1. InformaticsLessons updated on GitHub
2. Instructor shares nbgitpuller link
3. Students click link → JupyterHub pulls latest materials
4. Student work merged intelligently (no overwrites)
5. New materials appear in student workspace

---

## Educational Features

### Hands-On FHIR Learning

**Real EHR Data Access**:
- Query live FHIR server (WintEHR)
- Realistic patient cohorts (Synthea-generated)
- Complete medical records
- Production-like API responses

**Structured Curriculum**:
- Week-by-week progression
- Prerequisite tracking
- Scaffolded assignments
- Increasing complexity

**Immediate Feedback**:
- Run code, see results instantly
- Error messages guide learning
- Expected output comparisons
- Validation cells in notebooks

### Research Skills Development

**OMOP CDM Exposure**:
- Broadsea Atlas for cohort building
- OHDSI research methodologies
- Observational study design
- Data quality assessment

**Analytics Tools**:
- R analytics in Jupyter notebooks
- Python data science libraries
- Visualization techniques
- Statistical analysis methods

### Collaborative Learning

**Peer Interaction**:
- Shared JupyterHub environment
- Code sharing and review
- Discussion forums (planned)
- Group projects support

**Instructor Support**:
- View student progress
- Access student notebooks for help
- Provide inline feedback
- Monitor completion rates

---

## Research Features

### Cohort-Based Studies

**Atlas Cohort Definition**:
- Complex inclusion/exclusion criteria
- Temporal event relationships
- Drug exposure patterns
- Condition occurrence timing
- Measurement thresholds

**Cohort Validation**:
- Achilles data quality checks
- Cohort characterization reports
- Sample size estimation
- Feasibility assessment

### Comparative Effectiveness Research

**Propensity Score Matching**:
- CohortMethod package in HADES
- Large-scale PS analysis
- Covariate balance assessment
- Treatment effect estimation

**Outcome Analysis**:
- Survival analysis
- Time-to-event analysis
- Incidence rate calculations
- Risk difference estimation

### Data Quality and Governance

**Achilles Validation**:
- Plausibility checks
- Conformance to CDM
- Completeness assessment
- Temporal consistency

**Quality Dashboards**:
- Real-time data quality metrics
- Trend monitoring
- Threshold alerts
- Continuous improvement tracking

---

## Technical Features

### Containerization and Orchestration

**Docker Compose**:
- Multi-container orchestration
- Service dependencies
- Volume management
- Network isolation
- Profile-based deployments (Broadsea)

**Scalability**:
- Horizontal scaling via load balancing
- Kubernetes deployment support (planned)
- Auto-scaling based on demand
- Resource optimization

### Database Technologies

**PostgreSQL**:
- Relational data storage
- ACID compliance
- Advanced query optimization
- Replication and high availability

**OMOP CDM Schema**:
- 39 standardized tables
- Referential integrity
- Indexing strategies
- Partition support for large datasets

**Redis**:
- Session caching
- Query result caching
- Real-time data updates
- Pub/sub messaging (planned)

### API and Integration

**RESTful APIs**:
- WintEHR FHIR API (HAPI FHIR)
- Broadsea WebAPI (Java Spring)
- InformaticsClassroom API (Flask)
- Standardized error responses
- API versioning

**Bulk Data Operations**:
- FHIR Bulk Data export
- Large cohort extraction
- NDJSON format support
- Asynchronous processing

### Security and Compliance

**Data Encryption**:
- TLS/SSL for data in transit
- Database encryption at rest
- Secret management (Azure Key Vault)
- Encrypted backups

**Audit Logging**:
- User access logs
- Data modification tracking
- Admin action logging
- Security event monitoring

**Compliance**:
- HIPAA compliance features
- GDPR data protection (planned)
- 21 CFR Part 11 (planned)
- SOC 2 readiness (planned)

---

**Last Updated**: November 2025
**Next Review**: February 2026
