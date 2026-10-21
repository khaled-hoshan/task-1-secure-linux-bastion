# Secure Linux Bastion Host & Process Management

## Overview

This project was completed as part of the Elevvo Cloud & DevOps Engineering Internship.

The goal was to configure a Linux server with hardened SSH access, deploy a Python web service, manage it with systemd, and verify its operation and security settings.

## Environment

- Operating system: Ubuntu 24.04 LTS (virtual machine)
- Host system: Arch Linux
- Virtualization: QEMU/KVM with libvirt
- Remote access: OpenSSH using an Ed25519 key pair
- Web service: Python 3 HTTP server
- Service manager: systemd

## 1. SSH Hardening

SSH was configured to:

- Allow public-key authentication.
- Disable password authentication.
- Disable keyboard-interactive authentication.
- Prohibit direct root login.

Configuration example: `configs/ssh-hardening.conf`

Validation:

```bash
sudo sshd -t
sudo sshd -T | grep -E '^(pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|permitrootlogin) '
```

The effective configuration was checked, and key-based SSH access was tested.

**Security note:** The private SSH key is not included in this repository.

## 2. Python Web Service

A small HTTP service was deployed at `/opt/user-web/server.py`.

- `GET /health` returns HTTP 200 and a JSON health status.
- Other paths return HTTP 404.
- The service listens on port 8080.

Source code: `src/server.py`

Example health check:

```bash
curl -i http://127.0.0.1:8080/health
```

The service was also tested from the Arch host using the VM's private network address.

## 3. Dedicated Service Account

The application runs under a dedicated system account named `user-web`, rather than root.

The application directory and source file have restricted permissions.

## 4. systemd Service Management

The service is defined in `configs/user-web.service`.

It uses systemd to:

- Start the application.
- Run it as `user-web`.
- Restart it after an unexpected failure.
- Start automatically at boot.

Useful commands:

```bash
sudo systemctl status user-web
sudo systemctl start user-web
sudo systemctl stop user-web
sudo systemctl restart user-web
sudo systemctl enable user-web
sudo journalctl -u user-web -n 30 --no-pager
```

An unexpected process failure was tested, and systemd restarted the service.

## 5. systemd Security Hardening

Additional restrictions are documented in `configs/hardening.conf`, including:

- Private devices and temporary storage.
- Restricted kernel and process information access.
- Restricted address families.
- An empty capability bounding set.
- Restrictions on privilege escalation and system calls.

The effective settings were inspected after applying the override.

Security audit:

```bash
sudo systemd-analyze security user-web.service --no-pager
```

The observed audit score was 3.9/10 (reported by systemd as `OK`). This score is a heuristic, not a guarantee of security.

## 6. Verification Results

The following checks were performed:

- SSH hardening configuration validated.
- Ed25519 key-based SSH login succeeded.
- Password and keyboard-interactive authentication were disabled.
- Direct root login was disabled.
- The health endpoint returned HTTP 200.
- The host could reach the service over the VM's private network.
- systemd service startup, shutdown, and unexpected-failure recovery were tested.
- Service logs and the systemd security audit were inspected.

## 7. Evidence

Screenshots are available in the `evidence/` directory:

1. `01-ssh-hardening.png` — effective SSH settings.
2. `02-service-status.png` — systemd service status.
3. `03-local-health-check.png` — local HTTP health check.
4. `04-host-connectivity.png` — host-to-VM connectivity.
5. `05-security-audit.png` — systemd security audit.
6. `06-restart-logs.png` — service recovery and logs.

## 8. Repository Structure

```text
task-1-secure-linux-bastion/
├── README.md
├── .gitignore
├── configs/
│   ├── hardening.conf
│   ├── ssh-hardening.conf
│   └── user-web.service
├── evidence/
│   ├── 01-ssh-hardening.png
│   ├── 02-service-status.png
│   ├── 03-local-health-check.png
│   ├── 04-host-connectivity.png
│   ├── 05-security-audit.png
│   └── 06-restart-logs.png
└── src/
    └── server.py
```

## Disclaimer

The configuration files are provided as project documentation and examples. Paths, usernames, IP addresses, and service settings may need to be adapted to another environment.

This project was tested in a local virtual-machine environment. It does not claim that the server is exposed to or hardened for the public internet.
