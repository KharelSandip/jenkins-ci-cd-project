# Jenkins CI/CD Pipeline with Docker

A containerized Jenkins setup demonstrating CI/CD fundamentals using Docker Compose. This project showcases automated pipeline configuration, SSH-enabled build agents, and database integration for enterprise-grade continuous integration workflows.

## Project Overview

This repository implements a local Jenkins CI/CD environment using Docker containers. It demonstrates infrastructure-as-code practices by containerizing Jenkins with persistent storage, custom build agents with SSH access, and MySQL database integration for build artifacts or application data.

**Key Capabilities:**
- Fully containerized Jenkins instance with Docker Compose
- Custom Fedora 41-based SSH build agent
- Persistent Jenkins configuration and job data
- Network isolation with custom Docker networks
- MySQL database for application state management

## Architecture
```
┌─────────────────────────────────────────────────────────┐
│                    Docker Host                          │
│                                                         │
│  ┌─────────────────┐         ┌──────────────────┐       │
│  │  Jenkins Master │◄───────►│  Fedora41 Agent  │       │
│  │  (port 8080)    │   SSH   │  (MySQL + SSH)   │       │
│  │                 │         │                  │       │
│  └────────┬────────┘         └──────────────────┘       │
│           │                                             │
│           │ Volume Mount                                │
│           ▼                                             │
│  ┌─────────────────┐                                    │
│  │ jenkins_home/   │ (Persistent Storage)               │
│  └─────────────────┘                                    │
│                                                         │
│  Network: project-net                                   │
└─────────────────────────────────────────────────────────┘
```

The architecture consists of:
- **Jenkins Master**: Orchestrates builds and manages pipeline execution
- **SSH Build Agent**: Fedora 41 container with MySQL, configured for remote SSH builds
- **Persistent Storage**: Volume-mounted Jenkins home for job configurations and build history
- **Isolated Network**: Custom Docker bridge network for inter-container communication

## Technologies Used

- **Jenkins** - CI/CD automation server
- **Docker** - Container runtime
- **Docker Compose** - Multi-container orchestration
- **Fedora 41** - Base image for build agent
- **MySQL** - Database server for application data
- **OpenSSH** - Remote build agent connectivity
- **Bash** - Shell scripting for automation

## Repository Structure
```
jenkins-ci-cd-project/
├── docker-compose.yaml          # Docker Compose configuration
├── fedora41/
│   ├── Dockerfile               # Custom build agent image
│   ├── remote-key               # SSH private key (for agent auth)
│   └── remote-key.pub           # SSH public key
├── jenkins_home/                # Jenkins persistent data (gitignored)
│   ├── jobs/                    # Pipeline job definitions
│   ├── plugins/                 # Installed Jenkins plugins
│   ├── secrets/                 # Jenkins credentials store
│   ├── workspace/               # Build workspaces
│   └── ...
├── .gitignore                   # Git exclusions
└── README.md                    # This file
```

## CI/CD Pipeline Workflow

### Pipeline Trigger
- Pipelines can be triggered manually via Jenkins UI
- Supports SCM polling for automated builds on code commits
- Webhook integration ready for GitHub/GitLab push events

### Build Stages
1. **Checkout** - Clone source code from version control
2. **Build** - Compile application or prepare artifacts
3. **Test** - Execute unit/integration tests
4. **Deploy** - Deploy to target environment or registry

### Execution Flow
1. Jenkins master receives build trigger
2. Job dispatched to SSH build agent (Fedora 41 container)
3. Agent executes pipeline stages in isolated workspace
4. Build artifacts stored in Jenkins workspace
5. Results reported back to Jenkins master dashboard

## Jenkins Setup

### Prerequisites
- Docker Engine 20.10+
- Docker Compose 2.0+
- 2GB+ available RAM
- Ports 8080 available on host

### Installation Steps

1. **Clone the repository**
```bash
   git clone https://github.com/KharelSandip/jenkins-ci-cd-project.git
   cd jenkins-ci-cd-project
```

2. **Start Jenkins with Docker Compose**
```bash
   docker-compose up -d
```

3. **Verify containers are running**
```bash
   docker-compose ps
```
   Expected output:
```
   NAME       IMAGE             STATUS    PORTS
   jenkins    jenkins/jenkins   Up        0.0.0.0:8080->8080/tcp
```

4. **Retrieve initial admin password**
```bash
   docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

5. **Access Jenkins web interface**
   - Open browser: `http://localhost:8080`
   - Enter admin password from step 4
   - Complete setup wizard
   - Install recommended plugins

6. **Build custom SSH agent (optional)**
```bash
   cd fedora41
   docker build -t fedora41-jenkins-agent .
```

## Security Considerations

### Secrets Management
- **Initial admin password**: Auto-generated, stored in `/var/jenkins_home/secrets/`
- **SSH keys**: Located in `fedora41/remote-key*` - these are for demonstration only
- **Credentials**: Stored encrypted in Jenkins credentials store
- **Database passwords**: Hardcoded in Dockerfile (demonstration only - use environment variables in production)

### What Is NOT Committed
The `.gitignore` file explicitly excludes:
- `jenkins_home/` - Contains sensitive credentials, job configs, and build history
- SSH private keys (production keys should never be in VCS)
- Database files and build artifacts
- Temporary files and logs

### Production Recommendations
- Rotate all default passwords immediately
- Use Jenkins Credentials Plugin for secret management
- Store SSH keys in Jenkins credential store, not filesystem
- Enable HTTPS/TLS for Jenkins UI
- Implement RBAC with Jenkins role-based access control
- Use Docker secrets or external secret managers (Vault, AWS Secrets Manager)
- Never commit `jenkins_home/` or sensitive credentials to version control

## Running Locally

### Quick Start
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f jenkins

# Stop services
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v
```

### Accessing Services
- **Jenkins UI**: http://localhost:8080
- **Jenkins Container Shell**: `docker exec -it jenkins bash`
- **Build Agent SSH**: Configure in Jenkins as SSH build agent using `remote-key`

### Common Operations
```bash
# Restart Jenkins without losing data
docker-compose restart jenkins

# View Jenkins logs in real-time
docker logs -f jenkins

# Check container resource usage
docker stats jenkins

# Backup Jenkins configuration
docker cp jenkins:/var/jenkins_home ./jenkins_backup
```

## Skills

This project showcases proficiency in:

- **CI/CD Pipeline Design** - End-to-end automation workflow architecture
- **Containerization** - Docker image creation and multi-container orchestration
- **Infrastructure as Code** - Declarative infrastructure with Docker Compose
- **SSH Configuration** - Secure remote build agent setup and authentication
- **Linux System Administration** - User management, permissions, and service configuration
- **DevOps Practices** - Separation of concerns, persistent storage, network isolation
- **Security Awareness** - Credential management, least privilege, secret exclusion from VCS

##  Improvements

1. **Jenkins Pipeline as Code** - Implement declarative Jenkinsfile for version-controlled pipelines
2. **Multi-Stage Dockerfile Optimization** - Reduce build agent image size with multi-stage builds
3. **Dynamic Agent Provisioning** - Kubernetes or Docker plugin for on-demand agent scaling
4. **Automated Backups** - Scheduled Jenkins configuration and job backups to S3/NFS
5. **SSL/TLS Termination** - Nginx reverse proxy with Let's Encrypt certificates
6. **Monitoring Integration** - Prometheus + Grafana for Jenkins performance metrics
7. **Automated Testing** - Add sample application with unit/integration test stages
8. **Blue-Green Deployments** - Implement zero-downtime deployment strategy


## Author

**Sandip Kharel**
- GitHub: [@KharelSandip](https://github.com/KharelSandip)
- Project Link: [jenkins-ci-cd-project](https://github.com/KharelSandip/jenkins-ci-cd-project)

---

**Note**: This is a demonstration project for learning and portfolio purposes. Default credentials and configurations are intentionally simplified and should be hardened before being used in production.# jenkins-ci-cd-project
