# VirtualHere Server Ansible Role

This Ansible role automates the installation and configuration of the VirtualHere USB server on Linux systems.

## Overview

VirtualHere is a software that allows you to use USB devices remotely over a network. This role handles:

- Downloading the appropriate binary for your system architecture
- Installing the binary to `/usr/local/sbin`
- Creating the configuration directory
- Setting up a systemd service for automatic startup
- Managing the VirtualHere service lifecycle

## Requirements

- Linux system with systemd
- Root or sudo privileges
- Internet connectivity to download the VirtualHere binary
- Supported architectures:
  - x86_64
  - i386
  - aarch64 (ARM64)
  - armv7l (32-bit ARM)
  - armv6l (32-bit ARM)
  - mips
  - mipsel

## Role Variables

### Default Variables

The following variables are defined in `defaults/main.yaml`:

```yaml
# Architecture to binary name mapping
virtualhere_binaries:
  x86_64: vhusbdx86_64
  i386: vhusbdi386
  armv7l: vhusbdarm
  aarch64: vhusbdarm64
  mips: vhusbdmips
  mipsel: vhusbdmipsel

# Service configuration
virtualhere_service_enabled: true
virtualhere_service_state: started

# Configuration file path
virtualhere_config_path: /usr/local/etc/virtualhere/config.ini

# Binary installation path
virtualhere_binary_path: /usr/local/sbin
```

### Variable Descriptions

- `virtualhere_binaries`: Maps system architectures to their corresponding VirtualHere binary filenames
- `virtualhere_service_enabled`: Whether to enable the service at boot (default: true)
- `virtualhere_service_state`: Desired state of the service (default: started)
- `virtualhere_config_path`: Path to the VirtualHere configuration file
- `virtualhere_binary_path`: Installation directory for the VirtualHere binary

## Role Files

```
virtualhere-server/
├── defaults/
│   └── main.yaml          # Default variables
├── handlers/
│   └── main.yaml          # Service handlers
├── tasks/
│   └── main.yaml          # Installation tasks
├── templates/
│   └── virtualhere.service.j2  # Systemd service file template
└── README.md              # This file
```

## Task Flow

1. **Gather system facts** - Collects hardware information including architecture
2. **Determine binary filename** - Maps the detected architecture to the correct binary
3. **Download VirtualHere binary** - Fetches the binary from virtualhere.com
4. **Install binary** - Copies the binary to `/usr/local/sbin` with execute permissions
5. **Create configuration directory** - Ensures `/usr/local/etc/virtualhere` exists
6. **Deploy systemd service** - Creates the service file from template
7. **Enable and start service** - Enables autostart and starts the service
8. **Clean up** - Removes the temporary downloaded binary

## Usage

### Basic Usage

Include this role in your playbook:

```yaml
---
- hosts: all
  roles:
    - common.virtualhere_server
```

### With Custom Configuration Path

```yaml
---
- hosts: all
  roles:
    - role: common.virtualhere_server
      vars:
        virtualhere_config_path: /etc/virtualhere/config.ini
```

### Disable Autostart

```yaml
---
- hosts: all
  roles:
    - role: common.virtualhere_server
      vars:
        virtualhere_service_enabled: false
```

## Handlers

The role provides the following handlers that can be triggered by other roles or tasks:

- `reload systemd` - Reloads the systemd daemon (useful after service file changes)
- `restart virtualhere` - Restarts the VirtualHere service

## Service Details

The role creates a systemd service file at `/etc/systemd/system/virtualhere.service` with the following characteristics:

- **Type**: forking (service runs in background)
- **After**: network.target (starts after network is ready)
- **Restart**: on-failure (automatically restarts if it crashes)
- **RestartSec**: 10 seconds (wait before restart)
- **Enabled**: true (starts on system boot)

### Service Control

Once the role is applied, you can manage the service manually:

```bash
# Check status
systemctl status virtualhere

# Start the service
systemctl start virtualhere

# Stop the service
systemctl stop virtualhere

# Restart the service
systemctl restart virtualhere

# View logs
journalctl -u virtualhere -f
```

## Configuration

The role creates the configuration directory but does not create a default configuration file. You can:

1. Create a configuration file manually at `/usr/local/etc/virtualhere/config.ini`
2. Use another Ansible task/role to deploy a configuration file
3. Use VirtualHere's web interface to configure the server

For VirtualHere configuration details, visit: https://www.virtualhere.com/

## Architecture Detection

The role automatically detects your system architecture and downloads the appropriate binary:

| Architecture | Binary |
|---|---|
| x86_64 | vhusbdx86_64 |
| i386 | vhusbdi386 |
| aarch64 | vhusbdarm64 |
| armv7l | vhusbdarm |
| armv6l | vhusbdarm |
| mips | vhusbdmips |
| mipsel | vhusbdmipsel |

Unsupported architectures will fall back to the i386 binary.

## Idempotency

This role is idempotent. Running it multiple times will not cause issues:

- The binary will be re-downloaded and reinstalled
- The service file will be updated if the template has changed
- The service will be enabled and started (if not already)

## Error Handling

If the role fails:

1. Check internet connectivity to download the binary
2. Verify systemd is installed and running
3. Ensure sufficient disk space in `/tmp` and `/usr/local/sbin`
4. Check for permission issues (role requires sudo)
5. Review the Ansible output for specific error messages

## Troubleshooting

### Service fails to start
```bash
journalctl -u virtualhere -n 50
```

### Check binary was installed correctly
```bash
ls -la /usr/local/sbin/vhusbdx86_64
```

### Verify configuration directory exists
```bash
ls -la /usr/local/etc/virtualhere
```

### Check systemd service file
```bash
cat /etc/systemd/system/virtualhere.service
systemctl show virtualhere
```

## Dependencies

This role has no dependencies on other Ansible roles.

## License

This role is created to work with VirtualHere. Please refer to VirtualHere's license terms at https://www.virtualhere.com/

## Author

Created as an Ansible conversion of the official VirtualHere installation script.

## Links

- VirtualHere Official Site: https://www.virtualhere.com/
- Original Installation Script: https://github.com/virtualhere/script/blob/main/install_server