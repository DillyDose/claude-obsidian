---
type: synthesis
title: "DevOps Knowledge Map for Shared Notebook Server Setup"
created: 2026-05-10
updated: 2026-05-10
question: "What knowledge do I need to set up a cluster where others can create notebooks and run scripts?"
answer_quality: solid
tags:
  - devops
  - infrastructure
  - jupyter
  - kubernetes
  - linux
status: developing
related:
  - "[[SSH Fundamentals]]"
---

# DevOps Knowledge Map for Shared Notebook Server Setup

Setting up a shared notebook environment (e.g. JupyterHub on Kubernetes) requires knowledge across nine layers. Each layer is a prerequisite for the one above it.

---

## The Stack

```
User's browser
     |
     v
JupyterHub (pod in Kubernetes)
     |  -- authenticates user
     |  -- spawns a pod per user
     v
User's pod (container with Jupyter + Python packages)
     |
     v
Persistent Volume (NFS) -- files survive pod restarts
```

Kubernetes manages scheduling. Docker images define what's inside each pod. The firewall controls which ports are reachable. SSH is how the admin manages everything.

---

## Layer 1: Linux fundamentals

Every server runs headless Linux (no GUI). The CLI is the only interface.

| Term | Meaning |
|------|---------|
| Shell | Program that reads typed commands. Usually `bash` |
| Terminal | Window where you type. On a remote server, accessed via SSH |
| Root | Superuser with permission to do anything. Like Windows Administrator |
| `sudo` | Run one command as root without being root |
| Path | Address of a file, e.g. `/home/username/notebooks/` |
| Process | A running program. Every notebook kernel is a process |
| Daemon | A process running silently in the background |
| `systemctl` | Tool that starts, stops, and restarts daemons |
| Package manager | `apt install python3` — like an app store for the terminal |

Mental model: Linux is a tree of files starting at `/`. Everything, including hardware devices, is a file somewhere in that tree.

---

## Layer 2: Networking

How computers find and talk to each other.

| Term | Meaning |
|------|---------|
| IP address | A computer's phone number on a network. e.g. `192.168.1.10` |
| Port | A numbered door on a computer. Port 22 = SSH, 80 = HTTP, 443 = HTTPS, 8888 = Jupyter |
| SSH | Secure Shell. The tool for logging into a remote Linux server |
| Firewall | Rules about which ports are open or blocked |
| DNS | Translates names to IPs. `myserver.uni.edu` -> `10.0.1.5` |
| Localhost / `127.0.0.1` | "This machine itself." Only reachable from the same machine |
| Subnet | A group of IPs that can communicate directly |

Mental model: IP = room number, port = which door in the room, firewall = security guard checking who can enter.

---

## Layer 3: Containers (Docker)

A way to package a program with everything it needs so it behaves identically on any machine. JupyterHub gives each user their own container, so one user's broken environment cannot affect another's.

| Term | Meaning |
|------|---------|
| Container | An isolated, running instance of an application |
| Image | The template a container is created from. Like a class; a container is an instance |
| `Dockerfile` | Instructions for building an image |
| Docker Hub | Public library of pre-built images. `jupyter/scipy-notebook` is one to use directly |
| Volume | A folder shared between container and host. Data survives when the container stops |
| Registry | Any server that stores and serves images |

Mental model: An image is a recipe. A container is the dish cooked from it. Many dishes from one recipe, each in its own bowl.

---

## Layer 4: Kubernetes (the cluster)

Manages many containers across many machines. You describe what you want and Kubernetes makes it happen: scheduling pods, restarting failures, balancing load.

| Term | Meaning |
|------|---------|
| Cluster | A group of machines (nodes) managed together by Kubernetes |
| Node | One machine in the cluster. One is the control plane; others are workers |
| Pod | The smallest unit Kubernetes manages. Usually one container |
| Deployment | A description of how many pod copies to run and how to update them |
| Service | A stable network address for a group of pods. The address stays fixed even as pods come and go |
| Namespace | A virtual partition inside a cluster. Different projects or users in different namespaces |
| `kubectl` | CLI tool for talking to Kubernetes. `kubectl get pods` shows running pods |
| Helm | Package manager for Kubernetes. Installs pre-packaged "charts" instead of writing raw config |
| Helm chart | A pre-packaged Kubernetes application. JupyterHub has an official one |
| Persistent Volume (PV) | Storage that outlives a pod. Required so user notebooks survive pod restarts |

Mental model: Kubernetes is a hotel manager. You say "30 rooms ready for guests." The manager assigns rooms, handles broken ones, and keeps a waiting list. You don't care which specific room number each guest gets.

---

## Layer 5: JupyterHub

The actual notebook platform. Gives each user their own Jupyter environment.

| Term | Meaning |
|------|---------|
| Hub | The central server handling login and spawning environments |
| Spawner | Creates a per-user environment. On Kubernetes, creates a pod per user |
| Authenticator | Checks who is allowed to log in (passwords, Google, university SSO) |
| Single-user server | The notebook server running inside each user's pod |
| Profile | A selectable config for users. e.g. "Small (2 CPU)" vs "Large (8 CPU, GPU)" |
| `jupyterhub_config.py` | The main config file for all of the above |

Mental model: JupyterHub is the lobby and receptionist. A user checks in (authenticates), gets assigned a room (pod spawned), and works in their own private space (single-user server).

---

## Layer 6: Storage

Where files live. On a cluster this requires a shared network filesystem because pods can run on different machines.

| Term | Meaning |
|------|---------|
| NFS | Network File System. A server sharing a folder over the network so all nodes see the same files |
| Persistent Volume Claim (PVC) | A pod's request for storage. Kubernetes connects it to a matching PV |
| `StorageClass` | Describes what kind of storage is available (fast SSD, slow NFS, cloud disk) |

Mental model: NFS is a shared network drive. Every computer (node) can access it regardless of which desk (node) the user sits at today.

---

## Layer 7: User and access management

| Term | Meaning |
|------|---------|
| Linux users and groups | User accounts on the OS. `useradd`, `passwd`, `sudoers` |
| SSH keys | Cryptographic keys for authentication (see [[SSH Fundamentals]]) |
| LDAP / OAuth | Protocols for institutional single sign-on. Plug into JupyterHub's authenticator |

---

## Layer 8: Monitoring and operations

| Term | Meaning |
|------|---------|
| `journalctl` | Read logs from systemd daemons |
| `kubectl logs` | Read logs from a Kubernetes pod |
| Prometheus | The standard metrics collection system |
| Grafana | Dashboard for visualizing Prometheus metrics |

---

## Layer 9: Cloud or on-prem infrastructure

| Term | Meaning |
|------|---------|
| VMs | Virtual machines. Cloud providers rent these. Each node in your cluster is a VM |
| Managed Kubernetes | EKS (AWS), GKE (GCP), AKS (Azure) — Kubernetes with the control plane managed for you |
| Terraform / Ansible | Infrastructure as Code tools for reproducible cluster setup |

---

## Realistic learning order

1. Linux CLI fluency
2. Networking fundamentals (conceptual + hands-on with `ssh`, `curl`)
3. Docker (build images, run containers, write compose files)
4. Kubernetes basics via `minikube` locally
5. Deploy JupyterHub via the official Helm chart

The "Zero to JupyterHub" official docs cover steps 4-5 end to end.
