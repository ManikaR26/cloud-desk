# CloudDesk — Automated Cloud Service Request Management

CloudDesk is a Python Flask-based service request management application integrated with Terraform and AWS.

The application allows employees to submit IT service requests and administrators to manage them. I extended the project to automate AWS VPC creation after administrator approval.

The complete workflow was successfully tested by provisioning a real AWS VPC using Terraform and verifying it through the AWS Management Console.

## Tech Stack

- **Programming Language:** Python
- **Backend:** Flask
- **Database:** SQLite
- **ORM:** SQLAlchemy
- **Frontend:** HTML, CSS, Jinja2
- **Cloud Platform:** AWS
- **Infrastructure as Code:** Terraform
- **AWS Services:** VPC, Subnets, Route Tables
- **Tools:** AWS CLI, Git, GitHub, VS Code
- **Authentication:** Flask-Login, Flask-WTF

---

## Project Features

### Employee Features

- User registration and login
- Create and track service requests
- Select request category and priority
- Request AWS VPC provisioning
- Add comments and view request history
- View request status

### Administrator Features

- View and manage employee requests
- Assign tickets to administrators
- Update request priority and status
- Approve AWS infrastructure requests
- Trigger automatic Terraform provisioning
- Track provisioning jobs
- View AWS-generated VPC IDs

---

## Project Architecture

```text
Employee
   |
   v
CloudDesk Web Application
   |
   v
Flask Backend
   |
   v
SQLAlchemy + SQLite
   |
   v
Administrator Approval
   |
   v
Terraform Provisioning Job
   |
   v
AWS Provider
   |
   v
AWS VPC + Private Subnets
   |
   v
Provisioning Status Updated
```

---

## How I Built and Tested the Project

### Step 1: Developed the Flask Application

I built the service request management application using Python and Flask.

SQLAlchemy was used to interact with the SQLite database and manage users, tickets, assignments, comments and provisioning records.

The application was tested locally at:

http://127.0.0.1:5000

### Step 2: Added Terraform Automation

I integrated Terraform with the Flask backend to automate AWS VPC provisioning.

When an administrator approves a VPC request, CloudDesk creates a Terraform provisioning job.

The application generates two configuration files:

- `main.tf`
- `request.auto.tfvars.json`

The Terraform configuration defines a VPC, two private subnets, a private route table and route table associations.

### Step 3: Created an AWS Account

I created an AWS account using the AWS Free plan.

The AWS project was assigned to the Asia Pacific (Sydney) Region.

```text
AWS Region: ap-southeast-2
```

I confirmed the Region through the AWS project settings and configured CloudDesk to use the same Region.

### Step 4: Installed AWS CLI and Terraform

I installed AWS CLI and Terraform on my Windows laptop.

Verified the installation using:

```cmd
aws --version
terraform version
```

### Step 5: Connected AWS CLI to My AWS Account

I authenticated using AWS CLI's browser-based login.

```cmd
aws login --profile clouddesk --region ap-southeast-2
```

Configured the AWS Region:

```cmd
aws configure set region ap-southeast-2 --profile clouddesk
```

Verified authentication:

```cmd
aws sts get-caller-identity --profile clouddesk
```

I also used the `describe-vpcs` command to retrieve existing AWS VPC information.

This confirmed that my laptop could communicate with AWS.

### Step 6: Tested Terraform Preview Mode

Initially, CloudDesk was configured in Preview Mode.

```env
PROVISIONING_MODE=preview
PROVISIONING_REGION=ap-southeast-2
AUTO_RUN_PROVISIONING=true
```

I submitted a VPC request using an employee account and approved it through the administrator dashboard.

CloudDesk automatically generated Terraform configuration files.

The provisioning status changed to:

```text
Preview Ready
```

No actual AWS resources were created during this stage.

### Step 7: Initialized and Tested Terraform

I used Terraform commands to inspect the infrastructure configuration.

```cmd
terraform init
terraform plan
```

The execution plan displayed:

```text
Plan: 6 to add, 0 to change, 0 to destroy.
```

The six planned resources were:

- 1 AWS VPC
- 2 Private Subnets
- 1 Route Table
- 2 Route Table Associations

### Step 8: Resolved AWS Authentication Issues

During the first live provisioning attempt, Terraform failed to retrieve valid AWS credentials.

The error was:

```text
Error: No valid credential sources found
```

I inspected the Flask application logs to identify the issue.

I configured an AWS credential-process profile that could retrieve temporary credentials from AWS CLI.

```cmd
aws configure set credential_process "aws configure export-credentials --profile clouddesk --format process" --profile clouddesk-terraform
```

Configured the Region:

```cmd
aws configure set region ap-southeast-2 --profile clouddesk-terraform
```

Verified the new profile:

```cmd
aws sts get-caller-identity --profile clouddesk-terraform
```

I also resolved a local disk-space issue that affected Terraform provider installation and AWS credential caching.

After these changes, Terraform Plan executed successfully.

### Step 9: Enabled Live AWS Provisioning

After verifying AWS authentication and Terraform configuration, I enabled live provisioning.

```env
PROVISIONING_MODE=aws
PROVISIONING_REGION=ap-southeast-2
AUTO_RUN_PROVISIONING=true
ENABLE_AWS_PROVISIONING=I_UNDERSTAND_AWS_CHARGES
```

I restarted CloudDesk using the working AWS profile.

```cmd
set "AWS_PROFILE=clouddesk-terraform"
START_WINDOWS.bat
```

### Step 10: Successfully Created an AWS VPC

I submitted a new AWS VPC request through the employee dashboard.

The administrator reviewed and approved the request.

CloudDesk automatically executed Terraform to provision the infrastructure.

The job status changed to:

```text
Succeeded
```

The successfully provisioned VPC had the following configuration:

| Property | Value |
|----------|-------|
| Request | REQ-0004 |
| AWS Region | ap-southeast-2 |
| VPC Name | clouddesk-ticket-000004 |
| VPC CIDR | 10.80.0.0/16 |
| Private Subnets | 2 |
| VPC State | Available |

I verified the created VPC through:

**AWS Console → VPC → Your VPCs**

This confirmed that CloudDesk successfully automated real AWS infrastructure provisioning.

---

## Running the Project Locally

### Prerequisites

- Python
- Terraform CLI
- AWS CLI
- AWS account (only required for live provisioning)

### 1. Start the Application

Open the CloudDesk project directory and run:

```cmd
START_WINDOWS.bat
```

The Flask application starts at:

http://127.0.0.1:5000

### 2. Configure Environment Variables

For local testing, use Preview Mode:

```env
SECRET_KEY=YOUR_PRIVATE_RANDOM_SECRET_KEY
DATABASE_URL=sqlite:///cloud_desk.db
APP_ENV=development
COOKIE_SECURE=false

PROVISIONING_MODE=preview
PROVISIONING_REGION=ap-southeast-2
AUTO_RUN_PROVISIONING=true
```

Preview Mode does not create AWS resources.

For live provisioning, AWS credentials and the Terraform executable must be correctly configured, and AWS mode must be explicitly enabled.

### 3. Test the Workflow

1. Register or log in as an employee.
2. Create an AWS VPC request.
3. Log in as an administrator.
4. Review and approve the request.
5. Open the Automation dashboard.
6. Check the provisioning status.
7. In AWS mode, verify the created VPC in AWS Console.

---

## Project Structure

```text
CloudDesk_v2/
|
|-- app/
|   |-- models.py
|   |-- routes.py
|   |-- forms.py
|   |-- provisioning.py
|   |-- templates/
|   |-- static/
|
|-- provisioning/
|   |-- aws-vpc/
|       |-- main.tf
|
|-- infra/
|
|-- instance/
|   |-- provisioning/
|
|-- tests/
|-- docs/
|-- scripts/
|
|-- requirements.txt
|-- START_WINDOWS.bat
|-- Dockerfile
|-- wsgi.py
```

The `infra/` folder contains separate AWS hosting configurations. The tested automatic VPC provisioning workflow uses `provisioning/aws-vpc/`.

---

## Testing and Validation

The following features were tested:

- Employee and administrator authentication
- Service request creation and management
- Administrator approval workflow
- Terraform preview generation
- AWS CLI authentication
- Terraform initialization and planning
- Automatic Terraform execution
- Real AWS VPC creation
- Provisioning status tracking
- AWS Console verification

The updated application also includes automated tests for application and provisioning workflows.

---

## Current Limitations

- CloudDesk currently automates AWS VPC provisioning, not all AWS services.
- Employees are not automatically granted AWS IAM access to created resources.
- The Flask application was run locally, not deployed on EC2.
- EC2, RDS MySQL, CloudWatch and SNS configurations are included separately but were not deployed and validated end-to-end.
- Terraform state is stored locally for this project demonstration.
- The application is a working portfolio prototype rather than a production-ready enterprise system.

---

## Project Outcome

CloudDesk combines backend development, database management, Infrastructure as Code and AWS automation.

Through this project, I gained practical experience with:

- Flask backend development
- SQLAlchemy and SQLite
- Role-based access control
- AWS CLI authentication
- Terraform configuration and execution
- AWS VPC networking
- Automated infrastructure provisioning
- Troubleshooting cloud authentication and deployment errors

The project successfully demonstrates how an employee's infrastructure request can be processed through administrator approval and automatically provisioned on AWS using Terraform.

---

## Author

**Manika Rattan**

B.E. Computer Science and Engineering  
Thapar Institute of Engineering & Technology

GitHub: https://github.com/ManikaR26
