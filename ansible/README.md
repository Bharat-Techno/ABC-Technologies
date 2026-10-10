# Ansible Automation

## Overview

This directory contains the Ansible automation used by the **ABC-Technologies** project.

Ansible is responsible for validating the Kubernetes environment, deploying the application, verifying the deployment, performing automatic rollback when required, and cleaning up temporary deployment artifacts.

The playbooks are executed from **Jenkins Dynamic Kubernetes Agents** as part of the CI/CD pipeline.

---

# Architecture

```text
GitHub
    │
    ▼
Jenkins Pipeline
    │
    ▼
Dynamic Kubernetes Agent
    │
    ▼
Ansible
    │
    ▼
SSH
    │
    ▼
Kubernetes Control Plane
    │
    ▼
kubectl
    │
    ▼
Kubernetes Cluster
```

---

# Directory Structure

```text
ansible/
│
├── ansible.cfg
├── inventory.ini
│
├── group_vars/
│   └── all.yml
│
├── host_vars/
│   └── k8s-master.yml
│
├── playbooks/
│   ├── auto-fix.yml
│   ├── precheck.yml
│   ├── deploy.yml
│   ├── healthcheck.yml
│   ├── rollback.yml
│   └── cleanup.yml
│
└── README.md
```

---

# Prerequisites

Before running the playbooks ensure the following are available.

- Kubernetes Cluster
- kubectl
- Ansible
- SSH Access to Kubernetes Master
- Jenkins
- Docker Registry (Nexus)
- Container Image
- Kubernetes Manifests

---

# Inventory

The inventory contains the Kubernetes Control Plane.

```
k8s-master
```

Application deployment is performed through the Kubernetes API.

Worker nodes are managed automatically by Kubernetes Scheduler.

---

# Global Variables

Global variables are located in

```
group_vars/all.yml
```

Examples

- Application Name
- Namespace
- Docker Registry
- Image
- Ingress
- Monitoring
- Resource Limits
- HPA Configuration

---

# Host Variables

Host-specific variables are located in

```
host_vars/k8s-master.yml
```

Examples

- kubectl path
- kubeconfig
- Workspace
- Manifest directory
- API Server

---

# Playbooks

## auto-fix.yml

Performs safe environment preparation.

Responsibilities

- Create required directories
- Verify workspace
- Verify kubectl
- Create namespace if missing
- Fix kubeconfig permissions

---

## precheck.yml

Validates the Kubernetes environment before deployment.

Checks

- Kubernetes API
- Cluster Nodes
- Namespace
- Docker Registry
- StorageClass
- Istio
- Monitoring Stack
- Kubernetes Manifests

---

## deploy.yml

Deploys the application.

Tasks

- Update image
- Apply Kubernetes manifests
- Deploy application
- Deploy Service
- Deploy Ingress
- Deploy HPA
- Deploy ServiceMonitor
- Wait for rollout

---

## healthcheck.yml

Verifies the deployment.

Checks

- Deployment
- Pods
- Service
- Endpoints
- Ingress
- HPA
- ServiceMonitor
- Application URL

---

## rollback.yml

Automatically restores the previous deployment when required.

Tasks

- Display rollout history
- Rollback deployment
- Wait for rollout
- Verify pods
- Display running image

---

## cleanup.yml

Removes temporary deployment artifacts.

Tasks

- Remove temporary files
- Remove retry files
- Remove old logs
- Display deployment summary

---

# Running Playbooks

Run Auto Fix

```bash
ansible-playbook playbooks/auto-fix.yml
```

Run Precheck

```bash
ansible-playbook playbooks/precheck.yml
```

Deploy Application

```bash
ansible-playbook playbooks/deploy.yml \
  -e image_tag=v1
```

Health Check

```bash
ansible-playbook playbooks/healthcheck.yml
```

Rollback

```bash
ansible-playbook playbooks/rollback.yml
```

Cleanup

```bash
ansible-playbook playbooks/cleanup.yml
```

---

# Jenkins Pipeline Flow

```text
Clone Repository
        │
        ▼
Build (Maven)
        │
        ▼
Upload Artifact (Nexus)
        │
        ▼
Build Image (Kaniko)
        │
        ▼
Security Scan (Trivy)
        │
        ▼
Auto Fix
        │
        ▼
Precheck
        │
        ▼
Deploy
        │
        ▼
Health Check
        │
        ├───────────────┐
        │               │
        ▼               ▼
Success             Failure
        │               │
        ▼               ▼
Cleanup          Rollback
        │               │
        └──────► Cleanup
```

---

# Technologies Used

- Ansible
- Kubernetes
- Jenkins
- Docker
- Kaniko
- Nexus Repository
- Trivy
- Istio
- NGINX Ingress
- MetalLB
- Prometheus
- Grafana
- Loki
- Tempo
- OpenTelemetry

---

# Notes

- Application deployment is performed through the Kubernetes Control Plane.
- Worker nodes are managed automatically by Kubernetes Scheduler.
- The Jenkins pipeline injects the SSH private key during runtime.
- The image tag is provided by Jenkins using the current build number.
- Rollback uses Kubernetes Deployment revision history.

---

# Author

**Bharat Dasa**

ABC-Technologies DevSecOps Project