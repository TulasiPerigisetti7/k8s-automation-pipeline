# K8s Automation Pipeline

This repository provides an automated setup for a Kubernetes-based microservices platform using Ansible for infrastructure provisioning and Helm for application deployment.

## Repository Structure

```
k8s-automation-pipeline/
├── infra-automation/          # Ansible playbooks for DevOps environment setup
│   ├── site.yml              # Main playbook
│   ├── group_vars/
│   │   └── all.yml           # Global variables (versions, configurations)
│   └── roles/                # Ansible roles for different tools
│       ├── common/           # Common setup tasks
│       ├── docker/           # Docker installation
│       ├── kubectl/          # Kubernetes CLI installation
│       ├── minikube/         # Minikube setup
│       └── helm/             # Helm package manager installation
├── platform-chart/           # Helm charts for microservices deployment
│   └── microservices-platform/
│       ├── Chart.yaml        # Umbrella chart definition
│       └── values.yaml       # Default values for all microservices
└── .github/
    └── workflows/            # GitHub Actions workflows
        ├── platform-pipeline.yml          # Full deployment pipeline
        ├── setup-devops-environment.yml   # Environment setup only
        └── download-charts-from-jfrog.yml # Chart download only
        └── download-charts-from-jfrog.yml
```

## Infra Automation (Ansible Playbooks)

The `infra-automation/` directory contains Ansible playbooks designed to automate the setup of a complete DevOps environment on a local machine or server. This includes installing and configuring essential tools required for Kubernetes development and deployment.

### Main Playbook: `site.yml`

The primary playbook (`site.yml`) orchestrates the installation of:

- **Common Tools**: Basic system utilities and dependencies
- **Docker**: Container runtime for building and running containers
- **kubectl**: Kubernetes command-line tool for cluster management
- **Minikube**: Local Kubernetes cluster for development
- **Helm**: Package manager for Kubernetes applications

### Configuration

Global variables are defined in `group_vars/all.yml`, including:

- Minikube driver (docker)
- Specific versions for minikube (v1.33.0), kubectl (stable), and Helm (v3.15.0)

### Roles

Each role handles the installation and configuration of a specific tool:

- `common`: Sets up base system requirements
- `docker`: Installs Docker and configures it for use with Minikube
- `kubectl`: Downloads and installs the kubectl binary
- `minikube`: Installs Minikube and configures it to use Docker as the driver
- `helm`: Installs Helm v3 for chart management

## Platform Chart (Helm Umbrella Chart)

The `platform-chart/microservices-platform/` directory contains a Helm umbrella chart that manages the deployment of an entire microservices platform.

### Chart Details

- **Name**: microservices-platform
- **Type**: Application (umbrella chart)
- **Version**: 0.1.0
- **Description**: Umbrella chart for deploying all microservices

### Included Microservices

The umbrella chart deploys the following microservices:

- **auth-api**: Authentication service
- **frontend**: User interface application
- **redis-queue**: Message queue using Redis
- **todos-api**: Todo management API
- **users-api**: User management API
- **zipkin**: Distributed tracing system

### Configuration

Default values for each microservice are defined in `values.yaml`. These can be overridden during deployment to customize the platform configuration.

## CI/CD Workflows

The repository includes GitHub Actions workflows for automated setup, chart management, and full platform deployment.

### Full Platform Deployment Pipeline (`platform-pipeline.yml`)

This comprehensive workflow orchestrates the complete deployment process from environment setup to platform deployment:

1. **Trigger**: Manual dispatch with version input
2. **Input**: Chart version (default: 0.1.0)
3. **Jobs**:
   - **setup-devops-environment**: Installs Ansible and runs the playbook to set up Docker, kubectl, Minikube, and Helm
   - **download-charts** (depends on setup-devops-environment): Downloads microservice charts from JFrog Artifactory
   - **deploy-platform** (depends on download-charts): Updates Helm dependencies and deploys the umbrella chart

**Required Secrets**:
- `ARTIFACTORY_USER_NAME`: JFrog Artifactory username
- `ARTIFACTORY_PASSWORD`: JFrog Artifactory password
- `ARTIFACTORY_URL`: Base URL for JFrog Artifactory

### Setup DevOps Environment (`setup-devops-environment.yml`)

This standalone workflow automates the provisioning of a DevOps environment on self-hosted runners:

1. **Trigger**: Manual dispatch (`workflow_dispatch`)
2. **Runner**: Self-hosted
3. **Steps**:
   - Checks out the repository
   - Installs Ansible on the runner
   - Executes the Ansible playbook to set up Docker, kubectl, Minikube, and Helm

This workflow ensures that CI/CD runners have all necessary tools pre-installed for Kubernetes operations.

### Download Charts from JFrog (`download-charts-from-jfrog.yml`)

This standalone workflow handles the retrieval of dependency charts from JFrog Artifactory:

1. **Trigger**: Manual dispatch with version input
2. **Input**: Chart version (default: 0.1.0)
3. **Runner**: Self-hosted
4. **Steps**:
   - Downloads specified versions of microservice charts from JFrog Artifactory
   - Stores charts in the `microservices-platform/charts/` directory
   - Lists downloaded charts for verification

**Required Secrets**:
- `ARTIFACTORY_USER_NAME`: JFrog Artifactory username
- `ARTIFACTORY_PASSWORD`: JFrog Artifactory password
- `ARTIFACTORY_URL`: Base URL for JFrog Artifactory

## Prerequisites

- Ubuntu/Debian-based system (for Ansible playbooks)
- GitHub repository with self-hosted runners (for workflows)
- JFrog Artifactory access (for chart downloads)

## Usage

### Setting Up DevOps Environment

1. Ensure you have a self-hosted GitHub runner configured
2. Trigger the "Setup DevOps Environment" workflow manually
3. The workflow will install Ansible and run the playbook to set up all tools

### Deploying the Platform

#### Option 1: Full Automated Pipeline
1. Ensure you have a self-hosted GitHub runner configured
2. Trigger the "Full Platform Deployment Pipeline" workflow with the desired chart version
3. The workflow will automatically set up the environment, download charts, and deploy the platform

#### Option 2: Manual Steps
1. Set up the DevOps environment using the "Setup DevOps Environment" workflow
2. Download dependency charts using the "Download Charts from JFrog" workflow
3. Deploy the umbrella chart:

```bash
helm dependency update platform-chart/microservices-platform
helm upgrade --install platform platform-chart/microservices-platform
```

### Local Development

For local development:

1. Run the Ansible playbook locally:

```bash
ansible-playbook -i "localhost," -c local infra-automation/site.yml
```

2. Start Minikube:

```bash
minikube start --driver=docker
```

3. Deploy the platform:

```bash
helm install microservices-platform ./platform-chart/microservices-platform
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally using the provided Ansible playbooks and Helm charts
5. Submit a pull request

## License

[Add license information here]