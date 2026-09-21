# Ansible Infrastructure

<p align="center">
  <strong>Infrastructure as Code · Source of Truth · Reproducible Fedora Systems</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Ansible-Infrastructure-black?logo=ansible" alt="Ansible">
  <img src="https://img.shields.io/badge/Fedora-Server-black?logo=fedora" alt="Fedora">
  <img src="https://img.shields.io/badge/Podman-Containers-black?logo=podman" alt="Podman">
  <img src="https://img.shields.io/badge/License-AGPL--3.0-black" alt="License">
</p>

---

## Overview

This repository is the **source of truth for the infrastructure**.

It contains the Ansible configuration used to provision, configure, maintain, and verify Fedora-based systems.

> **A server should be reproducible from a clean operating-system installation using version-controlled infrastructure code.**

Instead of treating a server as a collection of manual changes, the system is defined as code.

```text
                         ┌──────────────────────────┐
                         │       Git Repository     │
                         │                          │
                         │   Infrastructure Code    │
                         │   Configuration          │
                         │   Documentation          │
                         └────────────┬─────────────┘
                                      │
                                      │
                                      ▼
                         ┌──────────────────────────┐
                         │        Ansible           │
                         │       Controller         │
                         └────────────┬─────────────┘
                                      │
                                      │ SSH
                                      ▼
                 ┌─────────────────────────────────────────┐
                 │             Managed Systems             │
                 │                                         │
                 │  Fedora                                 │
                 │  ├── Base configuration                 │
                 │  ├── Security                           │
                 │  ├── Packages                           │
                 │  ├── Services                           │
                 │  ├── Podman                             │
                 │  ├── Quadlet                            │
                 │  └── Monitoring                         │
                 └─────────────────────────────────────────┘
```

---

## Architecture

The infrastructure is designed in layers.

```text
┌────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                       │
│                                                            │
│       Gitea · PostgreSQL · Open WebUI · Web Services       │
├────────────────────────────────────────────────────────────┤
│                    CONTAINER LAYER                         │
│                                                            │
│             Podman · Quadlet · Networks · Volumes          │
├────────────────────────────────────────────────────────────┤
│                   OBSERVABILITY LAYER                      │
│                                                            │
│          Prometheus · Node Exporter · Grafana              │
├────────────────────────────────────────────────────────────┤
│                    SYSTEM LAYER                            │
│                                                            │
│     Users · SSH · Firewall · Packages · systemd            │
├────────────────────────────────────────────────────────────┤
│                   OPERATING SYSTEM                         │
│                                                            │
│                         Fedora                             │
└────────────────────────────────────────────────────────────┘
```

---

## Managed Infrastructure

| Component         | Purpose                | Management       |
| ----------------- | ---------------------- | ---------------- |
| **Fedora**        | Base operating system  | Ansible          |
| **SSH**           | Remote administration  | Ansible          |
| **Firewalld**     | Network security       | Ansible          |
| **systemd**       | Service management     | Ansible          |
| **Podman**        | Container runtime      | Ansible          |
| **Quadlet**       | Declarative containers | Ansible          |
| **Gitea**         | Git hosting            | Podman / Quadlet |
| **PostgreSQL**    | Database               | Podman / Quadlet |
| **Open WebUI**    | AI interface           | Podman / Quadlet |
| **Prometheus**    | Metrics collection     | Podman / Quadlet |
| **Node Exporter** | Host metrics           | Podman / Quadlet |
| **Grafana**       | Metrics visualization  | Podman / Quadlet |

---

## Repository Structure

```text
.
├── README.md
├── LICENSE
├── .gitignore
│
├── ansible.cfg
├── inventory.ini
├── site.yml
│
├── group_vars/
│   └── all.yml
│
├── host_vars/
│   └── ...
│
├── roles/
│   ├── base/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   ├── templates/
│   │   └── defaults/
│   │
│   ├── security/
│   │   ├── tasks/
│   │   └── templates/
│   │
│   ├── podman/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   └── templates/
│   │
│   └── monitoring/
│       ├── tasks/
│       └── templates/
│
└── files/
    └── ...
```

---

# Quick Start

### 1. Clone the repository

```bash
git clone <repository-url>
cd <repository-directory>
```

### 2. Verify Ansible

```bash
ansible --version
```

### 3. Verify the inventory

```bash
ansible-inventory -i inventory.ini --graph
```

Example:

```text
@all:
  |--@servers:
  |    |--z10
```

### 4. Test connectivity

```bash
ansible all -i inventory.ini -m ping
```

Expected:

```text
z10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 5. Review planned changes

```bash
ansible-playbook \
  -i inventory.ini \
  site.yml \
  --check \
  --diff
```

### 6. Apply the configuration

```bash
ansible-playbook \
  -i inventory.ini \
  site.yml \
  --ask-become-pass
```

---

# Configuration Lifecycle

Infrastructure changes follow a controlled lifecycle.

```text
       ┌───────────────┐
       │    Change     │
       │ Infrastructure│
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │      Git      │
       │     diff      │
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │   Validate    │
       │   YAML/Lint   │
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │     Check     │
       │ --check/diff  │
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │     Apply     │
       │    Ansible    │
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │    Verify     │
       │ Actual State  │
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │    Commit     │
       │    History    │
       └───────────────┘
```

---

# Safe Execution

Before applying changes:

```bash
git status
```

Review modifications:

```bash
git diff
```

Validate the playbook:

```bash
ansible-playbook \
  -i inventory.ini \
  site.yml \
  --syntax-check
```

Perform a dry run:

```bash
ansible-playbook \
  -i inventory.ini \
  site.yml \
  --check \
  --diff
```

Only then apply:

```bash
ansible-playbook \
  -i inventory.ini \
  site.yml \
  --ask-become-pass
```

---

# Idempotency

Ansible configuration should describe **desired state**, not a sequence of manual shell commands.

### Preferred

```yaml
- name: Ensure Podman is installed
  ansible.builtin.dnf:
    name: podman
    state: present
```

### Avoid

```yaml
- name: Install Podman
  ansible.builtin.command:
    cmd: dnf install -y podman
```

The first task describes the desired state.

The second describes an action.

The objective is that repeated execution converges toward the same state.

```text
First run:

    changed = many
    failed  = 0

Second run:

    changed = 0
    failed  = 0
```

---

# Infrastructure as Code Principles

### Source of Truth

The Git repository is the authoritative definition of the managed infrastructure.

Manual changes made directly on servers are considered **drift** unless they are subsequently represented in Ansible.

### Reproducibility

A clean Fedora installation should be capable of becoming an equivalent managed system through this repository.

### Idempotency

Repeated execution should converge toward the desired state without introducing unnecessary changes.

### Declarative Configuration

Configuration should describe **what the system should be**, rather than manually describing every step required to reach that state.

### Minimal Manual Configuration

If a configuration matters to the system, it should eventually be represented as code.

### Auditability

Every infrastructure change should be traceable through Git history.

```bash
git log --oneline --decorate
```

---

# Podman & Quadlet

Containerized services are managed declaratively using **Podman** and **Quadlet**.

```text
                         Ansible
                            │
                            ▼
                 /etc/containers/systemd/
                            │
                            ▼
                         Quadlet
                            │
                            ▼
                         systemd
                            │
                            ▼
                          Podman
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
           Gitea        PostgreSQL     Monitoring
```

Each layer has a defined responsibility:

| Layer       | Responsibility       |
| ----------- | -------------------- |
| **Ansible** | Deploy configuration |
| **Quadlet** | Describe containers  |
| **systemd** | Manage lifecycle     |
| **Podman**  | Run containers       |

---

# Security

Sensitive information must **never** be committed to Git.

Never commit:

```text
Passwords
Private keys
API tokens
Database credentials
Authentication secrets
TLS private keys
```

Use appropriate secret-management mechanisms such as:

* Ansible Vault
* External secret stores
* Environment-specific secret management

Before committing:

```bash
git diff --cached
```

Always review staged changes.

---

# Validation

The repository should progressively adopt automated validation.

```text
YAML validation
       │
       ▼
Ansible syntax check
       │
       ▼
ansible-lint
       │
       ▼
Check mode
       │
       ▼
Integration tests
       │
       ▼
Deployment
```

Example:

```bash
ansible-playbook \
  -i inventory.ini \
  site.yml \
  --syntax-check
```

Future validation:

```bash
ansible-lint
```

---

# CI/CD

The long-term objective is to validate infrastructure changes automatically.

```text
Git Push
   │
   ▼
CI Pipeline
   │
   ├── YAML validation
   ├── Ansible syntax check
   ├── ansible-lint
   └── Automated tests
   │
   ▼
Approved Change
   │
   ▼
Deployment
```

Infrastructure should fail validation **before** it reaches a managed system.

---

# Rebuild Strategy

One of the primary goals of this repository is the ability to rebuild infrastructure from a clean operating-system installation.

```text
                    Clean Fedora
                         │
                         ▼
                  Bootstrap Access
                         │
                         ▼
                    Run Ansible
                         │
                         ▼
                 Base Configuration
                         │
                         ▼
                   Security Layer
                         │
                         ▼
                Container Platform
                         │
                         ▼
                    Applications
                         │
                         ▼
                   Observability
                         │
                         ▼
                  Managed System
```

A server should not depend on undocumented steps performed months earlier.

> **If the infrastructure cannot be reconstructed, the source of truth is incomplete.**

---

# Drift Management

The desired state and actual state can diverge.

```text
┌──────────────────┐
│   Desired State  │
│                  │
│ Git + Ansible    │
└────────┬─────────┘
         │
         │
         ▼
┌──────────────────┐
│   Actual State   │
│                  │
│     Server       │
└──────────────────┘
```

The difference between these states is **configuration drift**.

Ansible should be used to detect and correct drift where appropriate.

Future automation may periodically perform compliance checks without applying changes.

---

# Git Workflow

Check the working tree:

```bash
git status
```

Review changes:

```bash
git diff
```

Stage changes:

```bash
git add .
```

Review staged changes:

```bash
git diff --cached
```

Commit:

```bash
git commit -m "Configure Podman baseline"
```

Review history:

```bash
git log --oneline --decorate
```

Prefer meaningful infrastructure commits:

```text
Configure Podman baseline
Add firewall configuration
Configure Cockpit service
Add monitoring stack
```

Avoid vague messages:

```text
Changed stuff
Update files
Fix things
```

---

# Roadmap

* [ ] Structured Ansible roles
* [ ] `ansible.cfg`
* [ ] Ansible Vault
* [ ] `ansible-lint`
* [ ] Automated syntax validation
* [ ] CI pipeline
* [ ] Integration testing
* [ ] Multi-host inventory
* [ ] Environment separation
* [ ] Automated compliance checks
* [ ] Configuration drift detection
* [ ] Full clean-install rebuild procedure
* [ ] Automated deployment pipeline

---

# Operational Philosophy

> **Infrastructure is code.**

> **Git records what should exist.**

> **Ansible enforces the desired state.**

> **Automation should be repeatable.**

> **Manual configuration should be minimized.**

> **If it cannot be reproduced, it is not fully documented.**

---

## License

This project is licensed under the **GNU Affero General Public License v3.0**.

See [`LICENSE`](LICENSE) for the complete license text.

---

<p align="center">
  <strong>Reproducible · Observable · Auditable · Explainable</strong>
</p>
