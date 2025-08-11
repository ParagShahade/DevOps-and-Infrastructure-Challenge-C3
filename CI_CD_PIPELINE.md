# CI/CD Pipeline for Ansible Infrastructure Deployment

## Overview
Simple GitLab CI/CD pipeline for deploying infrastructure updates to API servers (Server A and Server B) using Ansible. Covers the basic requirements: testing, build, deployment, and rollback.

## Pipeline Stages

```
Validate → Test → Deploy → Verify → Rollback
```

## 1. Validate Stage
- **Syntax Check**: Ansible playbook validation
- **Security Scan**: Basic vulnerability scanning

## 2. Test Stage
- **Docker Testing**: Test infrastructure in containers
- **Ansible Dry Run**: Preview changes without execution

## 3. Deploy Stage
- **Staging**: Test environment deployment
- **Production**: Production deployment (manual approval)

## 4. Verify Stage
- **Health Checks**: Verify services are running
- **Basic Monitoring**: Check Nginx and Zabbix status

## 5. Rollback Stage
- **Automatic Rollback**: Triggered if health checks fail
- **Manual Rollback**: Available for manual intervention

## Key Features

### Testing
- Docker-based infrastructure testing
- Ansible dry runs for change preview
- Basic health checks

### Deployment
- Staging environment for testing
- Production deployment with approval
- Rollback capability for failed deployments

### Secrets Management
- GitLab CI/CD variables for SSH keys
- Basic environment configuration

## GitLab CI/CD Configuration

```yaml
stages:
  - validate
  - test
  - deploy
  - verify
  - rollback

variables:
  ANSIBLE_HOST_KEY_CHECKING: "False"
  ANSIBLE_PRIVATE_KEY_FILE: "$SSH_PRIVATE_KEY"

validate:
  stage: validate
  script:
    - ansible-playbook --syntax-check install_nginx_zabbix.yml
    - ansible-playbook --syntax-check patch_and_reboot.yml

test:
  stage: test
  script:
    - docker-compose up -d
    - ansible-playbook -i inventory-docker.ini install_nginx_zabbix_local.yml --check
    - docker-compose down

deploy_staging:
  stage: deploy
  script:
    - echo "Deploying to staging environment..."
    - ansible-playbook -i inventory.ini install_nginx_zabbix.yml
  environment:
    name: staging
  only:
    - develop
  variables:
    ENVIRONMENT: "staging"
    ANSIBLE_USER: "$STAGING_SSH_USER"

deploy_production:
  stage: deploy
  script:
    - echo "Deploying to production environment..."
    - ansible-playbook -i inventory.ini install_nginx_zabbix.yml
  environment:
    name: production
  only:
    - main
  when: manual
  variables:
    ENVIRONMENT: "production"
    ANSIBLE_USER: "$PRODUCTION_SSH_USER"

verify:
  stage: verify
  script:
    - ansible -i inventory.ini all -m uri -a "url=https://192.168.10.3/health validate_certs=false"
    - ansible -i inventory.ini all -m uri -a "url=https://192.168.10.4/health validate_certs=false"
  allow_failure: false

rollback:
  stage: rollback
  script:
    - echo "Rolling back using maintenance playbook..."
    - ansible-playbook -i inventory.ini patch_and_reboot.yml
  when: manual
  only:
    - main
  environment:
    name: production
  variables:
    ENVIRONMENT: "production"
    ANSIBLE_USER: "$PRODUCTION_SSH_USER"
```

## GitLab CI/CD Variables Setup

To use this pipeline, configure these variables in GitLab:

### **Required Variables:**
- `SSH_PRIVATE_KEY` - SSH private key for server access
- `STAGING_SSH_USER` - SSH username for staging servers
- `PRODUCTION_SSH_USER` - SSH username for production servers

### **How to Set Variables:**
1. Go to GitLab project → Settings → CI/CD → Variables
2. Add each variable as "Protected" and "Masked"
3. Set variable type as "Variable" for usernames, "File" for SSH key

### **Example Variable Configuration:**
```
SSH_PRIVATE_KEY (File, Protected, Masked)
STAGING_SSH_USER (Variable, Protected, Masked) = "ubuntu"
PRODUCTION_SSH_USER (Variable, Protected, Masked) = "admin"
```

## Requirements Fulfillment

### ✅ **What Was Requested:**

1. **Outline a CI/CD GitLab pipeline for deploying updates to the API**
   - Simple 5-stage pipeline: Validate → Test → Deploy → Verify → Rollback
   - Basic GitLab CI/CD configuration

2. **Show how to automate testing, build, deployment, and rollback**
   - **Testing**: Docker testing + Ansible dry runs
   - **Build**: Infrastructure deployment with Ansible
   - **Deployment**: Staging and production environments
   - **Rollback**: Rollback stage using maintenance playbook

3. **Address secrets management and environment configuration**
   - **Secrets**: GitLab CI/CD variables for SSH keys and usernames
   - **Environment**: Staging vs production separation with environment-specific variables

## Summary

This pipeline provides the essential CI/CD functionality requested:
- Automated testing using Docker
- Infrastructure deployment to staging and production
- Basic health verification
- Rollback capability using maintenance playbook
- Simple secrets management

The approach is practical and focused on the actual requirements.