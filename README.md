# 🚀 Ansible Testing Lab (Docker)

A fast, portable Ansible testing environment using Docker. Starts in **seconds** instead of minutes!

## 📋 Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Network                        │
│                   192.168.100.0/24                       │
│                                                          │
│  ┌──────────────┐                                        │
│  │ controlnode  │ ─────────────────────────────────────┐ │
│  │ .10          │                                      │ │
│  │ (Ansible)    │                                      │ │
│  └──────────────┘                                      │ │
│         │                                              │ │
│         ▼                                              ▼ │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │    db01      │  │    web01     │  │    web02     │   │
│  │    .20       │  │    .21       │  │    .22       │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                          │
│  ┌──────────────┐                                        │
│  │ loadbalancer │                                        │
│  │    .30       │                                        │
│  └──────────────┘                                        │
└─────────────────────────────────────────────────────────┘
```

## 🚀 Quick Start

### 1. Start the Lab

```bash
docker-compose up -d
```

### 2. Access the Control Node

```bash
docker exec -it controlnode bash
```

**Vagrant vs Docker Command Comparison:**

| Task | Vagrant | Docker |
|------|---------|--------|
| Start environment | `vagrant up` | `docker-compose up -d` |
| SSH into control node | `vagrant ssh controlnode` | `docker exec -it controlnode bash` |
| SSH into db01 | `vagrant ssh db01` | `docker exec -it db01 bash` |
| SSH into web01 | `vagrant ssh web01` | `docker exec -it web01 bash` |
| Stop environment | `vagrant halt` | `docker-compose stop` |
| Destroy environment | `vagrant destroy` | `docker-compose down` |

### 3. Test Ansible Connectivity

```bash
# Inside the controlnode container
ansible all -m ping
```

### 4. Stop the Lab

```bash
docker-compose down
```

## 📁 Project Structure

```
.
├── docker-compose.yml    # Docker services configuration
├── Dockerfile.ansible    # Control node image with Ansible
├── hosts                 # Ansible inventory file
├── ansible-scenario2/    # Ansible playbooks (mounted to /ansible-scenario2)
├── ansible-scenario3/    # Ansible playbooks (mounted to /ansible-scenario3)
└── README.md
```

## 🖥️ Hosts & Credentials

| Host | IP Address | User | Password |
|------|------------|------|----------|
| controlnode | 192.168.100.10 | root | - |
| db01 | 192.168.100.20 | root | vagrant |
| web01 | 192.168.100.21 | root | vagrant |
| web02 | 192.168.100.22 | root | vagrant |
| loadbalancer | 192.168.100.30 | root | vagrant |

## 📝 Usage Examples

### Run a Playbook

```bash
# From your terminal
docker exec controlnode ansible-playbook /ansible/your-playbook.yml

# Or from inside the container
docker exec -it controlnode bash
cd /ansible
ansible-playbook your-playbook.yml
```

### Check All Hosts

```bash
docker exec controlnode ansible all -m shell -a "hostname"
```

### Copy Files to Hosts

```bash
docker exec controlnode ansible all -m copy -a "src=/tmp/file.txt dest=/tmp/file.txt"
```

## 🔧 Customization

### Add More Hosts

Edit `docker-compose.yml` to add new services following the existing pattern.

### Mounted Playbook Directories

The following directories are mounted in the control node:

| Host Directory | Container Path |
|----------------|----------------|
| `./ansible-scenario2/` | `/ansible-scenario2` |
| `./ansible-scenario3/` | `/ansible-scenario3` |

Add your playbooks to these directories — changes appear **instantly** without restarting!

## 🔄 When to Rebuild vs Restart

Not all changes require the same action:

| What Changed | Command to Run |
|--------------|-----------------|
| **Dockerfile** (e.g., `Dockerfile.ansible`) | `docker-compose up -d --build` |
| **docker-compose.yml** | `docker-compose up -d` |
| **Files in mounted volumes** (playbooks, configs) | **Nothing!** Changes appear instantly |

**Pro tip:** You don't need `docker-compose down` first — `docker-compose up -d --build` handles everything:
- Rebuilds images if Dockerfiles changed
- Recreates only the containers that need updating
- Leaves unchanged containers running

## ⏱️ Performance

| Method | Startup Time |
|--------|-------------|
| Docker (this) | ~30 seconds |
| Vagrant/VirtualBox | 10-20 minutes |

## 🐛 Troubleshooting

### Containers not starting?

```bash
docker-compose down
docker-compose up -d
docker-compose logs
```

### SSH connection issues?

Wait 30 seconds after starting for SSH servers to initialize, then:

```bash
docker exec controlnode ansible all -m ping
```

## 📄 License

MIT License - Feel free to use and modify!
