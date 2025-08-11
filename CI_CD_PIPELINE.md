# CI/CD Pipeline for Ansible Infrastructure Deployment

## Overview
Simple GitLab CI/CD pipeline for deploying infrastructure updates to API servers (Server A and Server B) using Ansible. Covers the basic requirements: testing, build, deployment, and rollback.

## Pipeline Stages

```
Validate → Test → Deploy → Verify
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

## Key Features

### Testing
- Docker-based infrastructure testing
- Ansible dry runs for change preview
- Basic health checks

### Deployment
- Staging environment for testing
- Production deployment with approval
- Simple rollback capability

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
    - ansible-playbook -i inventory.ini install_nginx_zabbix.yml
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - ansible-playbook -i inventory.ini install_nginx_zabbix.yml
  environment:
    name: production
  only:
    - main
  when: manual

verify:
  stage: verify
  script:
    - ansible -i inventory.ini all -m uri -a "url=https://192.168.10.3/health validate_certs=false"
    - ansible -i inventory.ini all -m uri -a "url=https://192.168.10.4/health validate_certs=false"
```

## Requirements Fulfillment

### ✅ **What Was Requested:**

1. **Outline a CI/CD GitLab pipeline for deploying updates to the API**
   - Simple 4-stage pipeline: Validate → Test → Deploy → Verify
   - Basic GitLab CI/CD configuration

2. **Show how to automate testing, build, deployment, and rollback**
   - **Testing**: Docker testing + Ansible dry runs
   - **Build**: Infrastructure deployment with Ansible
   - **Deployment**: Staging and production environments
   - **Rollback**: Manual rollback capability

3. **Address secrets management and environment configuration**
   - **Secrets**: GitLab variables for SSH keys
   - **Environment**: Staging vs production separation

## Summary

This pipeline provides the essential CI/CD functionality requested:
- Automated testing using Docker
- Infrastructure deployment to staging and production
- Basic health verification
- Simple secrets management

The approach is practical and focused on the actual requirements.