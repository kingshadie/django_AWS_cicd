# Django AWS CI/CD Pipeline

A Django application deployed to **Amazon EC2** through an automated **GitHub Actions CI/CD pipeline**.

This project demonstrates automated testing, AWS deployment, Linux application operations, secure CI/CD configuration, and a repeatable application delivery workflow.

## What This Project Demonstrates

- Automated testing and deployment of a Django application
- GitHub Actions CI/CD triggered by pushes to `main`
- AWS EC2 application deployment
- Direct SSH-based deployment to EC2
- Gunicorn application serving with systemd process management
- Automated Django migrations and static-file collection
- Secure handling of deployment credentials with GitHub Actions Secrets
- Configuration-drift control on a persistent EC2 host

## Architecture

```text
Developer
    │
    ▼
  GitHub
    │
    ▼
GitHub Actions
    │
    ├── Test Job
    │      ├── Checkout code
    │      ├── Setup Python 3.12
    │      ├── Install dependencies
    │      └── Run pytest
    │
    ▼
Deploy Job
    │
    ├── SSH into EC2
    ├── Synchronize repository
    ├── Install dependencies
    ├── Run migrations
    ├── Collect static files
    └── Restart Gunicorn
    │
    ▼
Amazon EC2
    │
    ├── Django
    ├── Gunicorn
    └── systemd
```

## Technology Stack

| Area | Technology |
|---|---|
| Application | Django / Python |
| Cloud | AWS |
| Compute | Amazon EC2 |
| CI/CD | GitHub Actions |
| Application Server | Gunicorn |
| Process Management | systemd |
| Testing | pytest |
| Deployment | SSH |
| Package Management | pip |
| Operating System | Linux |

## CI/CD Pipeline

The pipeline uses a two-stage workflow:

```text
Push to main
     │
     ▼
┌───────────────┐
│   Test Job    │
├───────────────┤
│ Checkout      │
│ Python 3.12   │
│ Install deps  │
│ Run pytest    │
└───────┬───────┘
        │
     Tests pass
        │
        ▼
┌───────────────┐
│  Deploy Job   │
├───────────────┤
│ SSH to EC2    │
│ Sync code     │
│ Install deps  │
│ Django migrate│
│ Collect static│
│ Restart Gunicorn
└───────────────┘
```

The deployment stage runs only when the test stage succeeds.

---

# Deployment

## Prerequisites

Before using the deployment workflow, you need:

- An AWS EC2 instance
- SSH access to the instance
- Python 3.12
- Git
- A Python virtual environment
- Gunicorn
- systemd service configuration
- A GitHub repository containing the Django application
- GitHub Actions enabled for the repository

## GitHub Actions Secrets

The deployment workflow uses encrypted GitHub Actions secrets for the EC2 connection:

```text
HOST
SSH_PRIVATE_KEY
```

These credentials must **not** be committed to the repository. The current implementation stores the SSH host and private key as GitHub Actions secrets.

## CI Test Steps

The test job performs the following operations:

### 1. Checkout the repository

The GitHub Actions workflow checks out the application source code.

### 2. Configure Python

Python 3.12 is configured for the workflow.

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Run automated tests

```bash
pytest
```

The deployment stage does not run unless the test stage succeeds.

## EC2 Deployment Steps

After successful tests, GitHub Actions connects to the EC2 instance using SSH.

### 1. Synchronize the repository

The deployment workflow first restores the working tree to the version-controlled state:

```bash
git reset --hard HEAD
git pull origin main
```

The `git reset --hard HEAD` step is deliberate. It ensures that the EC2 working tree returns to the version-controlled state before pulling the latest code, reducing the risk of local configuration drift interfering with deployment.

### 2. Activate the virtual environment

```bash
source venv/bin/activate
```

### 3. Install application dependencies

```bash
pip install -r requirements.txt
```

### 4. Apply Django database migrations

```bash
python manage.py migrate
```

### 5. Collect static files

```bash
python manage.py collectstatic --noinput
```

### 6. Restart Gunicorn

```bash
sudo systemctl restart gunicorn
```

These deployment operations are executed automatically by the GitHub Actions deployment job.

---

# Running Locally

### 1. Create a virtual environment

```bash
python -m venv venv
```

### 2. Activate the virtual environment

```bash
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Apply migrations

```bash
python manage.py migrate
```

### 5. Start the Django development server

```bash
python manage.py runserver
```

The original project uses these local setup commands.

---

# Running Tests

Run the test suite with:

```bash
pytest
```

The repository includes both `pytest.ini` and a dedicated `tests/` directory.

---

# Repository Structure

```text
.
├── .github/
│   └── workflows/
├── core/
├── main/
├── djangoawsenv/
├── templates/
├── tests/
├── .gitignore
├── manage.py
├── pytest.ini
├── requirements.txt
└── README.md
```

### Key Components

**`.github/workflows/`**  
Contains the CI/CD workflow.

**`core/`**  
Django project configuration and settings.

**`main/`**  
Application logic.

**`templates/`**  
Django templates.

**`tests/`**  
Automated test suite.

**`manage.py`**  
Django command-line interface.

---

# Deployment Design Decision: Configuration Drift

One of the most important operational lessons from this project was managing configuration drift on a persistent EC2 instance.

The deployment workflow intentionally runs:

```bash
git reset --hard HEAD
```

before:

```bash
git pull origin main
```

The purpose is to ensure that the EC2 working tree reflects the version-controlled application state before the next deployment.

This was a deliberate operational decision rather than an incidental command. It addresses a common problem in manually modified, always-on application servers: changes made directly on the server can conflict with the next automated deployment.

---

# Engineering Lessons

This project provided practical experience with:

- AWS EC2 application hosting
- CI/CD pipeline design
- Automated testing gates
- SSH-based deployment
- Linux application operations
- Gunicorn and systemd
- Python/Django application deployment
- Secure CI/CD secret management
- Database migration automation
- Static asset deployment
- Configuration-drift control

---

# Future Improvements

Potential improvements include:

- Provisioning the EC2 infrastructure with Terraform
- Introducing an AWS Application Load Balancer
- Adding HTTPS and automated certificate management
- Adding centralized logging
- Adding Prometheus and Grafana monitoring
- Implementing automated security and dependency scanning
- Adding deployment rollback mechanisms
- Implementing blue/green or rolling deployments
- Migrating the database to Amazon RDS
- Introducing containerized deployment with Docker
- Migrating the application to Amazon ECS or Amazon EKS

---

# Key Takeaway

This project demonstrates practical **AWS application deployment, CI/CD automation, automated testing, Linux operations, secure deployment configuration, and repeatable software delivery**.

It represents the transition from managing application code alone toward managing the **complete application delivery lifecycle**:

```text
Code
  ↓
Test
  ↓
Deploy
  ↓
Operate
  ↓
Monitor
```

**Focus areas:** AWS • EC2 • Django • Python • GitHub Actions • CI/CD • Linux • Automation • Deployment Engineering
