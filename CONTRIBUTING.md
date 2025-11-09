# Contributing to ExCITE Platform

Thank you for your interest in contributing to the ExCITE (Educational and Clinical Information Technology Environment) platform! This guide will help you get started.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Environment Setup](#development-environment-setup)
- [Contributing Guidelines](#contributing-guidelines)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)
- [Community](#community)

---

## Code of Conduct

### Our Pledge

We pledge to make participation in the ExCITE project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

**Positive behaviors include**:
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behaviors include**:
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Other conduct which could reasonably be considered inappropriate in a professional setting

### Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported to the project maintainers. All complaints will be reviewed and investigated promptly and fairly.

---

## Getting Started

### Prerequisites

Before you begin contributing, ensure you have:

1. **Git** installed and configured
2. **Docker** and **Docker Compose** (for local testing)
3. **Python 3.11+** (for InformaticsClassroom development)
4. **Node.js 18+** (for InformaticsClassroom frontend)
5. Familiarity with the relevant technologies:
   - **WintEHR**: React, FastAPI, FHIR, PostgreSQL
   - **Broadsea**: Docker Compose, OHDSI tools
   - **JupyterHubClassroom**: JupyterHub, Docker
   - **InformaticsClassroom**: Flask, React
   - **InformaticsLessons**: Jupyter Notebooks, FHIR

### Find an Issue

1. Browse the [issues](link-to-issues) for each module
2. Look for issues labeled `good first issue` or `help wanted`
3. Comment on the issue to express interest and get assigned
4. If you want to work on something not listed, create a new issue first

---

## Development Environment Setup

### 1. Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR-USERNAME/ExCITE.git
cd ExCITE
git remote add upstream https://github.com/ORIGINAL-ORG/ExCITE.git
```

### 2. Set Up Module-Specific Development

#### WintEHR

```bash
cd WintEHR

# Copy configuration
cp config.example.yaml config.yaml

# Deploy for development
./deploy.sh

# For frontend development
cd frontend
npm install
npm run dev  # Runs on http://localhost:5173 with hot reload

# For backend development
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Development dependencies
uvicorn app:app --reload  # Hot reload for FastAPI
```

#### Broadsea

```bash
cd Broadsea

# Copy environment
cp .env.example .env

# Deploy with profiles
docker compose --profile atlas-from-image --profile webapi-from-image up -d

# View logs
docker compose logs -f atlas webapi
```

#### JupyterHubClassroom

```bash
cd JupyterHubClassroom

# Setup for development
cp .env.example .env
echo "JUPYTERHUB_CRYPT_KEY=$(openssl rand -hex 32)" >> .env

# Deploy
./scripts/setup.sh

# Rebuild notebook image after changes
docker build -f Dockerfile.notebook -t jupyterhub-notebook:latest .
docker compose restart hub
```

#### InformaticsClassroom

```bash
cd InformaticsClassroom

# Backend setup
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Includes pytest, black, flake8

# Copy environment
cp .env.example .env

# Run backend (development mode with hot reload)
export FLASK_ENV=development
python3 app.py

# Frontend setup (separate terminal)
cd frontend
npm install
npm run dev  # Vite dev server with hot reload
```

#### InformaticsLessons

```bash
cd InformaticsLessons

# Install Jupyter
pip install jupyter jupyterlab

# Test notebooks locally
jupyter lab
```

---

## Contributing Guidelines

### Types of Contributions

We welcome various types of contributions:

1. **Bug Fixes**: Fix existing issues
2. **Features**: Add new functionality
3. **Documentation**: Improve guides, add examples
4. **Tests**: Increase test coverage
5. **Performance**: Optimize existing code
6. **Refactoring**: Improve code quality
7. **Course Materials**: Add/improve educational content (InformaticsLessons)

### Branch Naming Convention

```
feature/<brief-description>     # New features
bugfix/<issue-number>-<brief>   # Bug fixes
docs/<what-docs>                # Documentation
refactor/<what-refactor>        # Code refactoring
test/<what-test>                # Test additions

Examples:
feature/add-medication-alerts
bugfix/123-fix-patient-search
docs/deployment-guide-azure
refactor/fhir-client-module
test/patient-api-endpoints
```

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks
- `perf`: Performance improvements

**Examples**:

```
feat(wintehr): add medication interaction checking

Implements drug-drug interaction checking using clinical decision support rules.
Uses RxNorm for medication coding and checks against interaction database.

Closes #45
```

```
fix(jupyterhub): resolve notebook spawn timeout

Increased container spawn timeout from 30s to 60s to accommodate larger images.
Also improved error messaging when spawn fails.

Fixes #67
```

```
docs(readme): update deployment instructions for Azure

Added step-by-step guide for Azure VM deployment including NSG configuration
and managed database setup.
```

---

## Pull Request Process

### 1. Create Feature Branch

```bash
git checkout -b feature/your-feature-name
```

### 2. Make Changes

- Write clean, well-documented code
- Follow coding standards (see below)
- Add/update tests as needed
- Update documentation if needed

### 3. Test Your Changes

```bash
# Run tests for your module
# WintEHR
pytest

# InformaticsClassroom
pytest
npm test  # Frontend tests

# JupyterHub
# Manual testing - spawn notebook, verify functionality
```

### 4. Commit Changes

```bash
git add .
git commit -m "feat(module): brief description"
```

### 5. Push to Your Fork

```bash
git push origin feature/your-feature-name
```

### 6. Create Pull Request

1. Go to your fork on GitHub
2. Click "Pull Request"
3. Select your feature branch
4. Fill out PR template:
   - Description of changes
   - Issue number (if applicable)
   - Testing performed
   - Screenshots (if UI changes)
5. Submit PR

### 7. Code Review Process

- Maintainers will review your PR
- Address any requested changes
- Once approved, your PR will be merged

### PR Checklist

Before submitting, ensure:

- [ ] Code follows project coding standards
- [ ] Tests pass locally
- [ ] New tests added for new features
- [ ] Documentation updated (if needed)
- [ ] Commit messages follow convention
- [ ] No merge conflicts with main branch
- [ ] PR description clearly explains changes
- [ ] Linked to relevant issue(s)

---

## Coding Standards

### Python (WintEHR Backend, InformaticsClassroom)

**Style**: Follow [PEP 8](https://pep8.org/)

**Tools**:
```bash
# Format code
black .

# Check style
flake8

# Sort imports
isort .

# Type checking
mypy .
```

**Code Examples**:

```python
# Good
def get_patient_observations(
    patient_id: str,
    code: Optional[str] = None,
    start_date: Optional[datetime] = None
) -> List[Observation]:
    """
    Retrieve observations for a patient.

    Args:
        patient_id: FHIR Patient resource ID
        code: Optional LOINC code to filter observations
        start_date: Optional start date for filtering

    Returns:
        List of Observation resources matching criteria
    """
    query = db.query(Observation).filter(Observation.patient_id == patient_id)

    if code:
        query = query.filter(Observation.code == code)

    if start_date:
        query = query.filter(Observation.effective_date >= start_date)

    return query.all()


# Bad
def getobs(pid,c=None,d=None):
    q=db.query(Observation).filter(Observation.patient_id==pid)
    if c:q=query.filter(Observation.code==c)
    if d:q=q.filter(Observation.effective_date>=d)
    return q.all()
```

### JavaScript/React (WintEHR Frontend, InformaticsClassroom Frontend)

**Style**: Follow [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)

**Tools**:
```bash
# Format code
npx prettier --write .

# Lint
npx eslint .
```

**Code Examples**:

```javascript
// Good
import React, { useState, useEffect } from 'react';
import PropTypes from 'prop-types';

const PatientList = ({ onPatientSelect }) => {
  const [patients, setPatients] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchPatients = async () => {
      try {
        const response = await fetch('/api/patients');
        const data = await response.json();
        setPatients(data);
      } catch (error) {
        console.error('Error fetching patients:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchPatients();
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <ul>
      {patients.map((patient) => (
        <li key={patient.id} onClick={() => onPatientSelect(patient)}>
          {patient.name}
        </li>
      ))}
    </ul>
  );
};

PatientList.propTypes = {
  onPatientSelect: PropTypes.func.isRequired,
};

export default PatientList;
```

### Jupyter Notebooks (InformaticsLessons)

**Best Practices**:

1. **Clear Markdown**: Explain concepts before code
2. **Self-Contained**: Each notebook should be runnable independently
3. **Progressive**: Build complexity gradually
4. **Commented Code**: Explain non-obvious logic
5. **Expected Output**: Show what students should see
6. **Error Handling**: Include examples of common errors and how to fix them

**Structure**:

```markdown
# Lesson Title

## Learning Objectives
- Objective 1
- Objective 2

## Introduction
[Explain the concept]

## Code Example 1
```python
# Comment explaining what this does
import requests

fhir_base = "http://host.docker.internal:8888/fhir"
response = requests.get(f"{fhir_base}/Patient?_count=5")
patients = response.json()

# Expected output: Bundle with 5 Patient resources
print(f"Found {len(patients['entry'])} patients")
```

## Exercise 1
**Task**: Query for patients with diabetes

**Hint**: Use the `condition` search parameter

## Solution
[Code cell with solution]

## Common Errors
[Examples of mistakes and how to fix them]
```

---

## Testing Requirements

### WintEHR

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app --cov-report=html

# Run specific test file
pytest tests/test_patient_api.py

# Run specific test
pytest tests/test_patient_api.py::test_get_patient
```

**Minimum Coverage**: 80% for new code

### InformaticsClassroom

**Backend**:
```bash
# Run pytest
pytest

# With coverage
pytest --cov=app --cov-report=html
```

**Frontend**:
```bash
# Run Jest tests
npm test

# With coverage
npm test -- --coverage
```

### Manual Testing

For modules without comprehensive automated tests, perform manual testing:

**JupyterHub**:
1. Spawn notebook successfully
2. Access FHIR API from notebook
3. nbgitpuller clones repository correctly
4. User approval workflow functions

**Broadsea**:
1. Atlas loads without errors
2. Cohort definition saves
3. Vocabulary search works
4. WebAPI endpoints respond correctly

---

## Documentation

### Code Documentation

**Python (docstrings)**:
```python
def create_patient(data: PatientCreate) -> Patient:
    """
    Create a new patient resource.

    Args:
        data: Patient creation data including demographics

    Returns:
        Created Patient resource with generated ID

    Raises:
        ValueError: If required fields are missing
        DatabaseError: If database operation fails

    Example:
        >>> patient_data = PatientCreate(name="John Doe", birthDate="1980-01-01")
        >>> patient = create_patient(patient_data)
        >>> print(patient.id)
        "12345"
    """
    # Implementation
```

**JavaScript (JSDoc)**:
```javascript
/**
 * Fetch patient data from FHIR server
 *
 * @param {string} patientId - FHIR Patient resource ID
 * @param {Object} options - Optional fetch options
 * @param {boolean} options.includeObservations - Include observations
 * @returns {Promise<Patient>} Patient resource with optional observations
 * @throws {FetchError} If FHIR server request fails
 *
 * @example
 * const patient = await fetchPatient('123', { includeObservations: true });
 * console.log(patient.name);
 */
async function fetchPatient(patientId, options = {}) {
  // Implementation
}
```

### Documentation Updates

When adding features or making changes:

1. Update relevant README files in each module
2. Update this CONTRIBUTING.md if contributing process changes
3. Add examples to documentation
4. Update API documentation (if applicable)
5. Create/update diagrams if architecture changes

---

## Community

### Communication Channels

- **GitHub Issues**: Bug reports, feature requests
- **GitHub Discussions**: Questions, general discussion
- **Email**: [Maintainer email for private inquiries]

### Getting Help

- Review existing documentation
- Search closed issues for similar problems
- Ask in GitHub Discussions
- Tag maintainers in issues if urgent

### Recognition

Contributors will be recognized in:
- CONTRIBUTORS.md file
- Release notes
- Project README (significant contributions)

---

## License

By contributing to ExCITE, you agree that your contributions will be licensed under the same license as the project (see LICENSE files in each module).

---

## Questions?

If you have questions about contributing, please:

1. Check this guide and other documentation
2. Search existing issues
3. Create a new issue with the `question` label
4. Contact maintainers

Thank you for contributing to ExCITE! Your efforts help improve healthcare informatics education and research.

---

**Last Updated**: November 2025
**Maintained by**: ExCITE Project Team
