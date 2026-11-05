# RustDesk Ubuntu Terminal Setup

A practical guide to installing and configuring RustDesk on an Ubuntu virtual machine using SSH for administration.

## Project Overview

This project documents the installation and configuration of RustDesk on an Ubuntu VM connected to a television.

The objective was to configure RustDesk to connect to an organisation's self-hosted RustDesk server, allowing remote desktop access without needing to administer the machine directly through its graphical interface.

The project covers Linux package management, systemd service management, RustDesk configuration, unattended access, and troubleshooting.

## Project Objectives

- Install RustDesk on Ubuntu.
- Configure RustDesk to use a self-hosted server.
- Configure unattended remote access.
- Understand how RustDesk operates as a Linux system service.
- Investigate Linux processes and configuration file locations.
- Test service recovery after restarting RustDesk.

## Environment

| Component | Details |
|---|---|
| Operating system | Ubuntu |
| Architecture | x86_64 |
| RustDesk version | 1.5.1 |
| Installation method | Debian package (.deb) |
| Administration | SSH |
| Service manager | systemd |
| Display manager | LightDM |
| Remote access | Self-hosted RustDesk server |

**Note:** The VM had a graphical desktop environment managed by LightDM. Administration and configuration were performed through SSH.

---

## 1. Download and Install RustDesk

First, check the system architecture:

```bash
uname -m
```

The system used for this project returned:

```text
x86_64
```

Download the RustDesk 1.5.1 Debian package:

```bash
wget https://github.com/rustdesk/rustdesk/releases/download/1.5.1/rustdesk-1.5.1-x86_64.deb
```

Update the package repositories:

```bash
sudo apt update
```

Install RustDesk:

```bash
sudo apt install -y ./rustdesk-1.5.1-x86_64.deb
```

Verify the installation:

```bash
rustdesk --version
```

Expected output:

```text
1.5.1
```

**Troubleshooting note:** When downloading a specific release, ensure that the package filename matches the release version in the download URL. A mismatch can result in a 404 error.

---

## 2. Verify the RustDesk Service

RustDesk runs as a system service managed by systemd.

Check the service status:

```bash
sudo systemctl status rustdesk
```

The expected status is:

```text
Active: active (running)
```

Check whether the service is enabled at startup:

```bash
sudo systemctl is-enabled rustdesk
```

Expected output:

```text
enabled
```

Useful service management commands:

```bash
# Start RustDesk
sudo systemctl start rustdesk

# Stop RustDesk
sudo systemctl stop rustdesk

# Restart RustDesk
sudo systemctl restart rustdesk

# Check service status
sudo systemctl status rustdesk
```

### Service Status Screenshot

The following screenshot demonstrates that the RustDesk systemd service is enabled and running.

<!-- Add service-status.png here after uploading it to the screenshots directory. -->

---

## 3. Investigate RustDesk Processes

During troubleshooting, the running processes were inspected:

```bash
ps aux | grep -i rustdesk
```

The installation showed a system service running as `root`, while graphical RustDesk processes were running under the `lightdm` account.

The observed process layout was:

- `root` — RustDesk system service.
- `lightdm` — RustDesk graphical server and tray processes.

This distinction was important when investigating why the custom server configuration was not being applied as expected.

**Important:** Process ownership and configuration locations can differ between Linux installations. Inspect the actual running processes rather than assuming that every installation uses the same accounts.

---

## 4. Configure the Self-Hosted RustDesk Server

RustDesk can be configured to use a self-hosted server by providing the appropriate server address and public key.

The configuration files identified during this project included:

```text
/root/.config/rustdesk/RustDesk2.toml
/var/lib/lightdm/.config/rustdesk/RustDesk2.toml
```

In this particular environment, both configuration files were updated.

Example configuration structure:

```toml
rendezvous_server = 'YOUR_SERVER_ADDRESS:21116'

[options]
custom-rendezvous-server = 'YOUR_SERVER_ADDRESS'
key = 'YOUR_PUBLIC_SERVER_KEY'
```

Replace the placeholders with the appropriate server details.

### Configuration Troubleshooting

Initially, the Ubuntu VM appeared in the RustDesk client's Discovery section but showed as offline.

After investigating the running processes and configuration files, updating the `RustDesk2.toml` files in both the root and LightDM configuration directories resolved the issue.

This was an observed result in this environment, not a universal requirement for all RustDesk installations.

### Security Considerations

- Do not publish actual company server addresses or keys if they are considered confidential.
- Never publish passwords, device IDs, or authentication secrets.
- Avoid uploading the complete `RustDesk.toml` file, as it may contain sensitive values such as encrypted IDs, salts, passwords, and pairing information.
- Preserve existing configuration options unless you understand their purpose.

---

## 5. Configure Unattended Access

To configure a password for unattended access, the following command was used:

```bash
sudo rustdesk --password 'YOUR_STRONG_PASSWORD'
```

RustDesk confirmed that the password was successfully configured.

**Security note:** Never commit actual passwords to GitHub. Use a strong, unique password for the real deployment.

---

## 6. Test Remote Connectivity

After configuring the server details and unattended-access password:

1. Open the RustDesk client on the remote computer.
2. Ensure the client is configured to use the same self-hosted server.
3. Locate the Ubuntu VM.
4. Connect using the configured unattended-access password.

The VM became accessible through the RustDesk client.

---

## 7. Test Service Restart Persistence

A final test was performed to confirm that remote access remained available after restarting the RustDesk service.

Restart the service:

```bash
sudo systemctl restart

8. Lessons Learned

This project provided practical experience with:

Installing Linux applications using Debian packages.

Managing services with systemd.

Investigating Linux processes and user ownership.

Understanding application configuration files.

Configuring self-hosted remote-access infrastructure.

Troubleshooting connectivity and configuration persistence.

Testing service recovery after a restart.

Administering a graphical Ubuntu VM remotely through SSH.

One of the key lessons was that Linux applications can use different execution contexts for their background services and graphical components. Understanding which user runs each process can make configuration troubleshooting much easier.

9. Useful Commands Reference

Task

Command

Check architecture

uname -m

Check RustDesk version

rustdesk --version

Check service status

sudo systemctl status rustdesk

Start service

sudo systemctl start rustdesk

Stop service

sudo systemctl stop rustdesk

Restart service

sudo systemctl restart rustdesk

Check startup status

sudo systemctl is-enabled rustdesk

Inspect processes

ps aux | grep -i rustdesk

10. References

RustDesk Official Website

RustDesk GitHub Repository

RustDesk Documentation

Project Status

Completed: RustDesk 1.5.1 was installed and configured on the Ubuntu VM, connected to the organisation's self-hosted server, and tested successfully after a system service restart.

This repository documents the practical implementation, configuration, and troubleshooting process
