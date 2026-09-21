# 🔴 Ansible Infrastructure

<p align="center">
  <strong>Infrastructure as Code for Fedora Server</strong><br>
  Reproducible · Declarative · Idempotent · Auditable
</p>

<p align="center">

![Ansible](https://img.shields.io/badge/Ansible-Infrastructure-EE0000?logo=ansible\&logoColor=white)
![Fedora](https://img.shields.io/badge/Fedora-Server-51A2DA?logo=fedora\&logoColor=white)
![Podman](https://img.shields.io/badge/Podman-Containers-892CA0?logo=podman\&logoColor=white)
![Quadlet](https://img.shields.io/badge/Quadlet-systemd-333333)
![YAML](https://img.shields.io/badge/YAML-Configuration-CB171E?logo=yaml\&logoColor=white)
![Git](https://img.shields.io/badge/Git-Version_Control-F05032?logo=git\&logoColor=white)
![License](https://img.shields.io/badge/License-AGPL--3.0-blue)

</p>

> [!NOTE]
> This repository is the **source of truth** for the server infrastructure.

---

## 🔧 Overview

This repository defines and automates the configuration of a Fedora Server environment using **Infrastructure as Code (IaC)**.

The goal is simple:

> **A server should be reproducible from the repository, not rebuilt manually from memory.**

The infrastructure is managed through a layered approach:

```text
                    ┌──────────────────────┐
                    │       🔴 Ansible     │
                    │   Source of Truth    │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
       🟦 Fedora          🔐 Security       ⚙️ Services
       Operating System   firewalld/SSH      systemd
              │                                 │
              │                                 ▼
              │                         ⚫ Quadlet
              │                                 │
              │                                 ▼
              │                         🟣 Podman
              │                                 │
              │                    ┌────────────┼────────────┐
              │                    ▼            ▼            ▼
              │                  Gitea      PostgreSQL   Applications
              │
              └─────────────────────────────────────────────
```

---

## 🎨 Technology Color Language

The project uses a consistent visual language to make the infrastructure easier to understand.

| Technology              | Color  | Responsibility               |
| ----------------------- | ------ | ---------------------------- |
| 🔴 **Ansible**          | Red    | Automation and configuration |
| 🟦 **Fedora**           | Blue   | Operating system             |
| 🟣 **Podman**           | Purple | Containers                   |
| ⚫ **Quadlet / systemd** | Dark   | Service lifecycle            |
| 🟡 **YAML**             | Gold   | Configuration                |
| 🟢 **Git**              | Green  | Version control              |
| 🔵 **GitHub**           | Blue   | Repository and collaboration |
| 🟠 **Prometheus**       | Orange | Metrics and monitoring       |
| 🟣 **Grafana**          | Purple | Visualization                |
| 🟢 **nginx**            | Green  | Web and reverse proxy        |
| 🐘 **PostgreSQL**       | Blue   | Database                     |
| 🔵 **Open WebUI**       | Blue   | AI interface                 |
| 🟢 **Ollama**           | Green  | Local AI runtime             |

This color language is also used when documenting and learning the infrastructure.

---

## 🖥️ Managed Infrastructure

The project is designed to manage a complete Fedora Server environment.

### 🟦 Operating System

* Fedora Server
* System packages
* Users and permissions
* SSH
* firewalld
* systemd services
* Filesystem configuration

### 🟣 Containers

Containers are managed using Podman and Quadlet.

Current application stack includes:

* Gitea
* PostgreSQL
* nginx
* Open WebUI
* Additional services as the infrastructure evolves

### 🟢 Local AI

Ollama runs directly on the Fedora host rather than inside a container.

This allows the system to provide local AI workloads while keeping GPU access available to the host runtime.

### 📊 Monitoring

Monitoring is being developed around:

* Prometheus
* node-exporter
* Grafana

Monitoring configuration will eventually become part of the same Infrastructure as Code lifecycle.

---

## 📁 Repository Structure

The repository is intentionally structured so that infrastructure definitions remain understandable as the project grows.

```text
.
├── site.yml
├── inventory.ini
├── group_vars/
├── host_vars/
├── roles/
│   ├── base/
│   ├── security/
│   ├── podman/
│   ├── services/
│   └── monitoring/
├── files/
├── templates/
├── handlers/
├── .gitignore
├── LICENSE
└── README.md
```

The structure will evolve as infrastructure requirements become more complex.

---

## 🚀 Quick Start

### 1. Clone the repository

```bash
git clone <repository-url>
cd <repository-directory>
```

### 2. Verify Ansible

```bash
ansible --version
```

### 3. Test connectivity

```bash
ansible all -i inventory.ini -m ping
```

Expected result:

```text
z10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 4. Review the playbook

```bash
ansible-playbook site.yml --syntax-check
```

### 5. Perform a dry run

```bash
ansible-playbook site.yml --check
```

### 6. Apply the configuration

```bash
ansible-playbook site.yml
```

---

## 🔄 Configuration Lifecycle

The intended workflow is:

```text
Edit
  │
  ▼
🔴 Ansible
  │
  ▼
Validate
  │
  ├── syntax-check
  ├── check mode
  └── lint / tests
  │
  ▼
Git commit
  │
  ▼
Git push
  │
  ▼
Deploy
  │
  ▼
🟦 Fedora Server
```

The Git repository records **what the infrastructure should be**.

Ansible makes the actual system converge toward that state.

---

## 🛡️ Safe Execution

Before changing infrastructure:

```bash
ansible-playbook site.yml --syntax-check
```

Then inspect the expected changes:

```bash
ansible-playbook site.yml --check
```

Only after reviewing the result should the configuration be applied:

```bash
ansible-playbook site.yml
```

> [!IMPORTANT]
> Never commit passwords, private keys, API tokens, or other secrets to the repository.

---

## ♻️ Idempotency

Ansible configuration should be **idempotent**.

Running the same playbook repeatedly should converge the system toward the desired state without repeatedly changing resources that are already correct.

Example:

```bash
ansible-playbook site.yml
ansible-playbook site.yml
```

The second execution should normally report few or no changes.

This is one of the fundamental properties of the infrastructure.

---

## 🟣 Podman & ⚫ Quadlet

Containers are treated as infrastructure rather than manually managed processes.

The desired relationship is:

```text
🔴 Ansible
     │
     ▼
⚫ Quadlet definition
     │
     ▼
systemd
     │
     ▼
🟣 Podman
     │
     ├── Container
     ├── Network
     ├── Volume
     └── Image
```

Quadlet provides the declarative bridge between systemd and Podman.

This allows container workloads to participate in the normal systemd service lifecycle.

---

## 🔐 Security

Security is part of the infrastructure definition rather than a manual post-installation step.

The project aims to manage:

* SSH configuration
* Authentication
* firewalld
* Service exposure
* File permissions
* Container isolation
* Least-privilege access
* Secrets handling

> [!WARNING]
> Manual changes made directly on managed systems can create configuration drift.

---

## 🔎 Validation

Infrastructure changes should be validated before deployment.

Current validation workflow:

```bash
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --check
ansible-playbook site.yml
```

Future validation may include:

* ansible-lint
* Molecule
* Automated integration tests
* CI validation
* Configuration compliance checks

---

## 🌱 Git Workflow

Infrastructure changes are treated as code changes.

```text
Working Tree
     │
     ▼
Review
     │
     ▼
git diff
     │
     ▼
Commit
     │
     ▼
Push
     │
     ▼
Repository
```

Useful commands:

```bash
git status
git diff
git add .
git commit -m "Configure ..."
git log --oneline
git push
```

Each commit should represent a meaningful infrastructure change.

---

## 🧭 Rebuild Strategy

One of the primary goals of this project is **reproducibility**.

A clean Fedora installation should eventually be capable of becoming the complete server environment by applying the Ansible configuration.

```text
Clean Fedora
     │
     ▼
🔴 Ansible
     │
     ├── Base configuration
     ├── Security
     ├── Packages
     ├── Services
     ├── 🟣 Podman
     ├── ⚫ Quadlet
     └── Monitoring
     │
     ▼
Configured Server
```

The objective is not merely to automate an existing server.

The objective is to make the server **rebuildable**.

---

## 📈 Drift Management

Configuration drift occurs when the actual server state differs from the state defined in Git.

Examples include:

* Manually installed packages
* Changed firewall rules
* Modified configuration files
* Manually created containers
* Changed systemd configuration

The desired model is:

```text
Git
 │
 │ desired state
 ▼
🔴 Ansible
 │
 │ convergence
 ▼
🟦 Fedora Server
 │
 │ actual state
 └───────────────┐
                 │
                 ▼
             Validation
```

Over time, automated compliance checks can detect and report differences.

---

## 🛣️ Roadmap

* [x] Fedora Server baseline
* [x] Ansible connectivity
* [x] Basic system configuration
* [x] Podman
* [x] Quadlet
* [x] Containerized services
* [ ] Structured Ansible roles
* [ ] Automated validation
* [ ] ansible-lint
* [ ] CI pipeline
* [ ] Prometheus monitoring
* [ ] Grafana dashboards
* [ ] Automated compliance checks
* [ ] Multi-server deployment
* [ ] Full clean-install rebuild

---

## 🧠 Operational Philosophy

This project follows a few simple principles:

**Infrastructure is code.**

If it matters, define it in Git.

**Automation over repetition.**

If a task needs to be performed more than once, automate it.

**Declarative over manual.**

Describe the desired state instead of documenting a sequence of manual actions.

**Reproducibility over tribal knowledge.**

A new server should not depend on remembering everything that was done to the previous one.

**Small, understandable systems.**

Complexity should be introduced only when it provides a clear benefit.

---

## 📜 License

This project is licensed under the **GNU Affero General Public License v3.0**.

See [`LICENSE`](LICENSE) for the complete license text.

---

<p align="center">
  <sub>Infrastructure as Code · Fedora · Ansible · Podman · Quadlet</sub>
</p>
