# 🚀 AWS EC2 Web Application Deployment

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-Compute-orange?logo=amazonec2&logoColor=white)](https://aws.amazon.com/ec2/)
[![AWS Systems Manager](https://img.shields.io/badge/AWS%20Systems%20Manager-SSM-blue?logo=amazonaws&logoColor=white)](https://aws.amazon.com/systems-manager/)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-black?logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![OIDC](https://img.shields.io/badge/GitHub%20OIDC-IAM-purple)](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

> A practical AWS deployment project using Amazon EC2, Nginx, AWS Systems Manager, IAM, GitHub Actions, and GitHub OIDC.

This project focuses on deploying a web application to an EC2 instance and automating deployments through GitHub Actions.

The interesting part was not just running a web server. The goal was to understand how a CI/CD pipeline can deploy to an EC2 instance without relying on traditional SSH-based deployment.

---

## 🚀 What I Built

I deployed a web application on an Ubuntu EC2 instance with Nginx and created an automated deployment workflow.

The final architecture looks like this:

```text
Developer
    │
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ▼
GitHub OIDC
    │
    ▼
AWS IAM Role
    │
    ▼
AWS Systems Manager
    │
    ▼
Amazon EC2
    │
    ▼
Nginx
    │
    ▼
Web Application
```

A push to the configured GitHub branch triggers the workflow, which authenticates with AWS and uses Systems Manager to execute the deployment on the EC2 instance.

---

## ✨ Project Highlights

- 🖥️ Web application deployed on Amazon EC2
- 🌐 Nginx configured as the web server
- ⚙️ Automated deployment with GitHub Actions
- 🔐 GitHub OIDC authentication with AWS
- 👤 IAM role-based access
- 🧰 AWS Systems Manager Run Command
- 🚫 No SSH-based deployment
- 🔑 Temporary AWS credentials through OIDC
- 🧪 End-to-end deployment testing
- 📦 Application deployment directly to EC2

---

## 🏗️ Architecture

### Deployment Architecture

```text
┌─────────────────────┐
│  GitHub Repository  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  GitHub Actions     │
└──────────┬──────────┘
           │
           │ OIDC
           ▼
┌─────────────────────┐
│      AWS IAM        │
│   Deployment Role   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ AWS Systems Manager │
│     Run Command     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│      Amazon EC2     │
│      Ubuntu         │
│                     │
│       Nginx         │
│         │           │
│         ▼           │
│   Web Application   │
└─────────────────────┘
```

### Authentication Flow

```text
GitHub Actions
      │
      ▼
GitHub OIDC
      │
      ▼
AWS STS
      │
      ▼
IAM Deployment Role
      │
      ▼
Temporary AWS Credentials
```

---

## 🛠️ AWS Services & Technologies

| Technology             | Purpose                               |
| ---------------------- | ------------------------------------- |
| 🖥️ Amazon EC2          | Host the web application              |
| 🧰 AWS Systems Manager | Execute deployment commands on EC2    |
| 🔐 AWS IAM             | Manage identities and permissions     |
| 🔑 GitHub OIDC         | Authenticate GitHub Actions with AWS  |
| ⚙️ GitHub Actions      | Automate deployments                  |
| 🌐 Nginx               | Serve the web application             |
| 🐧 Ubuntu              | Operating system for the EC2 instance |

---

## 🌐 Web Server

The application runs on an Ubuntu EC2 instance with Nginx configured as the web server.

Nginx receives incoming HTTP requests and serves the deployed application.

```text
Browser
   │
   ▼
Nginx
   │
   ▼
Web Application
```

This gave me hands-on experience with both the AWS infrastructure layer and the application hosting layer.

---

## ⚙️ AWS Systems Manager

Instead of using SSH from GitHub Actions, the deployment uses AWS Systems Manager Run Command.

The deployment flow is:

```text
GitHub Actions
      │
      ▼
AWS Systems Manager
      │
      ▼
SSM Agent
      │
      ▼
EC2 Instance
      │
      ▼
Deployment Commands
```

This was one of the main learning points of the project.

The EC2 instance runs the SSM Agent, allowing Systems Manager to communicate with the instance and execute commands.

---

## 🔐 GitHub OIDC Authentication

The GitHub Actions workflow does not store long-lived AWS access keys.

Instead, it uses GitHub OIDC to authenticate with AWS.

The workflow requests an identity token from GitHub, which AWS validates through the IAM trust relationship.

Once the trust requirements are satisfied, GitHub Actions can assume the deployment IAM role and receive temporary AWS credentials.

```text
GitHub Actions
      │
      ▼
OIDC Token
      │
      ▼
AWS IAM Trust Policy
      │
      ▼
IAM Role
      │
      ▼
Temporary Credentials
```

This keeps long-lived AWS credentials out of the GitHub repository.

---

## 🧩 IAM: Trust vs Permissions

This project gave me practical experience with two different parts of IAM.

### Trust Policy

The trust policy defines:

> Who can assume the role?

For the deployment role, the trusted identity is GitHub through OIDC.

### Permissions Policy

The permissions policy defines:

> What can the role do after it is assumed?

The deployment role was given the permissions required to interact with AWS Systems Manager and execute the deployment.

Understanding this separation was an important part of getting the pipeline working.

---

## 🖥️ EC2 and SSM Setup

The EC2 instance runs Ubuntu and has the SSM Agent installed.

The SSM Agent allows AWS Systems Manager to communicate with the instance.

The deployment process can then execute commands on the EC2 instance without requiring an SSH connection from GitHub Actions.

The setup also involved configuring the required IAM permissions and ensuring that the EC2 instance was available as a managed instance in Systems Manager.

---

## ⚙️ GitHub Actions Deployment

The deployment workflow uses GitHub Actions to authenticate with AWS and send commands through Systems Manager.

A simplified version of the workflow looks like:

```yaml
name: Deploy Web Application

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: <AWS_IAM_ROLE_ARN>
          aws-region: <AWS_REGION>

      - name: Deploy using Systems Manager
        run: |
          aws ssm send-command \
            --document-name "AWS-RunShellScript" \
            --targets "Key=instanceIds,Values=<EC2_INSTANCE_ID>" \
            --parameters 'commands=[
              "<DEPLOYMENT_COMMANDS>"
            ]'
```

The actual project uses environment-specific values that are intentionally not included in this README.

---

## 🔄 Deployment Flow

When code is pushed to the main branch:

```text
1. GitHub receives the change
             │
             ▼
2. GitHub Actions starts
             │
             ▼
3. GitHub provides an OIDC identity token
             │
             ▼
4. AWS IAM validates the trust relationship
             │
             ▼
5. GitHub Actions receives temporary AWS credentials
             │
             ▼
6. Systems Manager receives the deployment command
             │
             ▼
7. SSM Agent executes the command on EC2
             │
             ▼
8. Application is updated
             │
             ▼
9. Nginx serves the updated application
```

---

## 🧪 Testing

The deployment was tested end to end.

The workflow successfully went through:

```text
GitHub
   ↓
GitHub Actions
   ↓
OIDC Authentication
   ↓
AWS IAM
   ↓
Systems Manager
   ↓
EC2
   ↓
Nginx
   ↓
Web Application
```

I also verified the application after deployment to confirm that the updated version was being served by Nginx.

---

## 🧯 Troubleshooting

One of the most valuable parts of this project was troubleshooting the AWS configuration rather than simply following a working example.

### GitHub OIDC Trust

The initial IAM trust configuration did not match the repository's required OIDC subject.

The trust relationship was updated to use the correct repository identity, allowing GitHub Actions to assume the AWS role.

### Systems Manager

The EC2 instance initially required additional configuration before it could be managed correctly through Systems Manager.

This involved understanding the SSM Agent, EC2 IAM configuration, and Systems Manager managed-instance setup.

### IAM Permissions

The GitHub deployment role needed the appropriate Systems Manager permissions before GitHub Actions could send commands successfully.

Working through these permission errors helped reinforce the difference between authentication and authorization in AWS.

---

## 🔒 Security Considerations

Security was kept in mind throughout the project.

The deployment avoids storing long-lived AWS access keys inside GitHub.

Instead:

- 🔐 GitHub OIDC is used for authentication
- 👤 IAM roles are used instead of permanent credentials
- ⏱️ AWS provides temporary credentials
- 🎯 IAM permissions are limited to the deployment requirements
- 🚫 SSH credentials are not required by the CI/CD workflow
- 🔑 No credentials or secrets are stored in the repository

Account-specific identifiers and credentials are intentionally excluded from this README.

---

## 📁 Project Structure

```text
aws-web-project/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── index.html
│
└── README.md
```

The exact application structure can be expanded depending on the web application being deployed.

---

## 💡 What I Learned

### Amazon EC2

- EC2 instance setup
- Ubuntu server administration
- Application hosting
- Nginx configuration
- Basic server deployment

### AWS Systems Manager

- SSM Agent
- Managed instances
- Run Command
- Remote command execution
- Deploying without SSH

### AWS IAM

- IAM roles
- Trust policies
- Permission policies
- Role assumption
- Temporary credentials
- Least-privilege access

### GitHub Actions

- Workflow triggers
- GitHub repository permissions
- AWS credential configuration
- Automated deployment pipelines

### GitHub OIDC

- OIDC identity tokens
- IAM trust relationships
- AWS STS
- Temporary AWS credentials
- Repository-based authentication

---

## 🎯 Why I Built This

I wanted to understand how a real deployment pipeline can interact with an EC2 server without relying on manually connecting to the server every time an application changes.

The project gave me hands-on experience connecting several pieces together:

**GitHub Actions + OIDC + IAM + Systems Manager + EC2 + Nginx**

More importantly, I had to troubleshoot the IAM and Systems Manager configuration myself, which made the architecture much easier to understand.

---

## 🔮 Possible Next Steps

There are several ways this architecture could be extended:

- Add an Application Load Balancer
- Add Auto Scaling
- Use Amazon CloudWatch for monitoring and logs
- Add a proper application process manager
- Introduce blue/green or rolling deployments
- Add CloudFront for content delivery
- Move infrastructure configuration to Infrastructure as Code

These would make the architecture more suitable for a larger production workload.

---

## 📌 Key Takeaway

This project helped me understand how AWS infrastructure, IAM, Systems Manager, and CI/CD fit together.

The final deployment architecture is:

```text
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
GitHub OIDC
   │
   ▼
AWS IAM
   │
   ▼
Systems Manager
   │
   ▼
EC2
   │
   ▼
Nginx
   │
   ▼
Web Application
```

The main takeaway was learning how to automate EC2 deployments while keeping authentication separate from application deployment and avoiding SSH-based CI/CD.

---

## 👨‍💻 Project

Built as part of my hands-on AWS learning journey, with a focus on understanding AWS infrastructure, IAM, Systems Manager, and automated application deployment.
