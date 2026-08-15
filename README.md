# Django AWS CI/CD Pipeline

A Django application with an automated CI/CD pipeline for AWS deployment via GitHub Actions.

## What this demonstrates

- Automated testing and deployment pipeline for a Django application
- GitHub Actions workflow triggering on push (build → test → deploy)
- AWS deployment integration
- Django project structure and configuration for cloud deployment

## Stack

- **App:** Django / Python
- **CI/CD:** GitHub Actions
- **Cloud:** AWS EC2 (direct SSH deploy — no PaaS/container layer)
- **App server:** Gunicorn, managed via systemd
- **Testing:** pytest (see `pytest.ini`, `tests/`)

## Pipeline

Two-stage pipeline: `test` must pass before `deploy` runs.

```
Push to main → GitHub Actions:

  test job:
    1. Checkout code
    2. Set up Python 3.12
    3. Install dependencies (pip install -r requirements.txt)
    4. Run test suite (pytest)

  deploy job (runs only if test job succeeds):
    5. SSH into EC2 instance (appleboy/ssh-action)
    6. git pull origin main (hard reset first to discard local drift)
    7. Activate virtualenv, reinstall dependencies
    8. Run Django migrations (python manage.py migrate)
    9. Collect static files (collectstatic --noinput)
    10. Restart Gunicorn via systemctl
```

SSH host and private key are stored as encrypted GitHub Actions secrets (`HOST`, `SSH_PRIVATE_KEY`) — never committed to the repo.

## Repository structure

| Path | Purpose |
|---|---|
| `core/` | Django project settings |
| `main/` | Application logic |
| `templates/` | Django templates |
| `tests/` | Test suite |
| `.github/workflows/` | CI/CD pipeline definition |

## Running locally

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

## Running tests

```bash
pytest
```
