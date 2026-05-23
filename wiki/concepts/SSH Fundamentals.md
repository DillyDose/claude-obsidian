---
type: concept
title: "SSH Fundamentals"
created: 2026-05-10
updated: 2026-05-10
tags:
  - ssh
  - networking
  - linux
  - devops
status: developing
related:
  - "[[DevOps Knowledge Map for Shared Notebook Server Setup]]"
---

# SSH Fundamentals

SSH (Secure Shell) opens a terminal on a remote machine. Every command typed runs on that remote machine, not the local laptop. Connecting looks simple but has six layers that can each fail independently, which is why getting it working the first time takes effort.

---

## Why connecting is harder than it looks

### 1. The server has to be reachable at all

Before SSH starts, the machine must be on the network and its IP address must be known.

- The machine may be behind a VPN. Connect to VPN first, then SSH.
- The machine may only be reachable from inside a specific network (firewall blocks outside access).
- A private IP (`10.x.x.x`, `192.168.x.x`) is not reachable from the internet at all.

### 2. The SSH daemon has to be running

SSH is a daemon called `sshd`. If it is not running, the connection is refused. On a fresh server it is usually on, but not always. Check with `systemctl status sshd`.

### 3. Port 22 has to be open

SSH uses port 22 by default. A firewall blocking port 22 produces "connection timed out" — which looks identical to "the server is off." Some admins move SSH to a non-standard port (e.g. 2222) to reduce bot attacks. The port must be known to connect.

### 4. Authentication

Two methods to prove identity:

**Password login.** Straightforward but many servers disable it for security. A disabled password login returns "permission denied" with no explanation.

**SSH keys.** More common and secure. A pair of files:
- Private key: lives on the local machine, never shared.
- Public key: placed on the server before connecting.

The server checks if the private key matches a known public key. No match means no entry.

| Term | Meaning |
|------|---------|
| `ssh-keygen` | Generates a key pair on the local machine |
| `~/.ssh/id_ed25519` | Where the private key lives (ed25519 is the modern default) |
| `~/.ssh/authorized_keys` | File on the server listing public keys allowed to connect |
| `ssh-copy-id` | Copies a public key to the server automatically |

The catch: the public key must get onto the server before the first login. If password login is disabled and no key is there, the account is locked out completely.

### 5. The correct username

Connecting as `john@10.0.1.5` vs `admin@10.0.1.5` are different accounts with different permissions. The wrong username gives "permission denied" that looks the same as a wrong key.

### 6. The first connection warning

The first SSH to any server shows:

```
The authenticity of host '10.0.1.5' can't be established.
Are you sure you want to continue connecting? (yes/no)?
```

Type `yes` and SSH remembers the server in `~/.ssh/known_hosts`. If the server's identity ever changes (reinstalled, different machine at same IP), SSH refuses to connect with a warning. The old record must be removed from `~/.ssh/known_hosts` manually.

---

## Why your teammate said getting it working removes the headache

Once SSH works, managing the server is routine. But reaching that point the first time requires:

- Network access (VPN or on-site)
- Knowing the IP and port
- The right SSH key set up and public key on the server
- The correct username

Each is a separate thing to debug. Most SSH errors only say "permission denied" or "connection timed out" without indicating which layer failed. After the first successful connection, future sessions are a single command.

---

## Pre-setup checklist

- [ ] Know the server's IP address and SSH port (default 22, may differ)
- [ ] Know if a VPN is required
- [ ] Know the username for login
- [ ] Generate an SSH key pair on the local machine with `ssh-keygen` if one does not exist
- [ ] Get the public key onto the server before the setup day
- [ ] Test the connection once before the actual setup day

---

## Basic commands

```bash
# Generate a key pair (one time, on your laptop)
ssh-keygen -t ed25519

# Copy your public key to the server (requires password login to be enabled)
ssh-copy-id username@server-ip

# Connect
ssh username@server-ip

# Connect on a non-standard port
ssh -p 2222 username@server-ip
```
