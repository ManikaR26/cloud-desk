# CloudDesk — Cloud-Based Service Request Management

A Flask service desk built as a hands-on cloud engineering portfolio project.
Run locally with SQLite, then deploy on EC2 with a private RDS MySQL database.

**Start here:** [Windows setup](docs/LOCAL_SETUP.md). Then follow [AWS deployment](docs/AWS_DEPLOYMENT.md).

## Included
- Registration and login, hashed passwords, requester/admin access.
- Ticket creation, category/priority, assignment to an administrator, status changes.
- Requesters only see their own tickets; administrators see and manage all tickets.
- Comments, timestamped activity history, dashboard counts, search, filters and pagination.
- CSV export with spreadsheet-formula escaping.
- CSRF protection, input validation, login throttling, security headers and optimistic update locking.
- Database readiness/liveness endpoints; application logging.
- Docker support; GitHub Actions test workflow.
- Terraform infrastructure: VPC, public application subnet, two private DB subnets, security groups,
  EC2, encrypted RDS MySQL, seven-day automated backup retention, IAM log permissions,
  CloudWatch log group, CPU/instance-health alarms and SNS email subscription.
- EC2 installation scripts, systemd service, HTTPS nginx configuration and recovery runbook.

## Quick local start (Windows PowerShell, Python 3.12 recommended)
```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe scripts/bootstrap.py
.\.venv\Scripts\python.exe -m flask --app wsgi init-db
.\.venv\Scripts\python.exe -m flask --app wsgi create-admin
.\.venv\Scripts\python.exe -m flask --app wsgi seed-demo
.\.venv\Scripts\python.exe -m flask --app wsgi run
```
Open http://127.0.0.1:5000. Sign in using the admin account you created. There are no default passwords.
Use Register to create a requester account, then compare the two roles.

## Project structure
- `app/`: application factory, models, forms, routes, templates and styles.
- `tests/`: functional, authorization, security and local recovery tests.
- `infra/`: Terraform AWS resources. Review before applying; AWS charges may apply.
- `scripts/`: local configuration, EC2 installation, RDS setup, service and logging files.
- `docs/`: local/AWS setup, architecture, backup/restore, demo and interview notes.

## Validation and limits
See `docs/VALIDATION.md` for the actual checks performed. Cloud resources have NOT been provisioned
from this package. AWS account-specific behavior, MySQL integration, email delivery, Docker runtime
and RDS restore must be validated in your account/environment.

This is a learning/portfolio application, not an enterprise-ready service. It uses one EC2 instance,
single-AZ RDS and one Gunicorn worker with threads. The rate limiter is process-local; multiple workers
or replicas require shared storage such as Redis. Activity history is application-level, not tamper-proof.
No email verification, password-reset flow, SLA clock, attachments or SSO is implemented.
`init-db` creates initial tables; it is not a schema-migration system. Add Alembic for future schema changes.
The AWS demo uses a self-signed HTTPS certificate and IP-restricted access. A public deployment needs
a trusted certificate/domain and an appropriate access/registration policy.

## Learning sequence
1. Run locally and read `app/routes.py` with the UI open.
2. Run tests and inspect the role checks.
3. Draw the architecture in `docs/ARCHITECTURE.md` yourself.
4. Deploy the infrastructure and app using the runbook.
5. Test log collection, notifications and recovery. Keep screenshots of real results.
6. Use the resume bullets only after completing the corresponding work.
