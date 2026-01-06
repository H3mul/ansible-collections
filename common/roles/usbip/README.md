# USBIPD Ansible Role

This Ansible role automates the installation and configuration of USBIPD (USB/IP Device Server) on Linux systems with systemd.

## Overview

USBIPD is a server application that allows you to share USB devices over a network using the USB/IP protocol. This role handles:

- Installing linux-tools for the current kernel
- Installing hwdata package for hardware database
- Creating a systemd service for automatic startup and management
- Enabling and starting the USBIPD service

## Features

- Automatic kernel version detection
- Systemd service management
- Configurable listen address and port
- Auto-restart on failure
- Full service lifecycle management

## Requirements

- Linux system with systemd
- Debian/Ubuntu-based distribution (uses apt)
- `sudo` or root access
- Network connectivity (for package installation)

## Supported Distributions

- Ubuntu 16.04 and later
- Debian 8 and later
- Any Debian-based distribution with systemd

## Role Variables

All variables are optional and have sensible defaults:

### `usbipd_service_enabled` (boolean)
Whether to enable the USBIPD service at boot.
Default: `true`

### `usbipd_service_state` (string)
Desired state of the USBIPD service (`started`, `stopped`, `restarted`).
Default: `started`

### `usbipd_binary_path` (string)
Path to the usbipd binary.
Default: `/usr/sbin/usbipd`

### `usbipd_listen_address` (string)
IP address to listen on (0.0.0.0 = all interfaces).
Default: `0.0.0.0`

### `usbipd_port` (integer)
TCP port for USB/IP protocol.
Default: `3240`

## Example Playbook

### Basic Installation

```yaml
---
- hosts: all
  become: true
  roles:
    - common.usbipd
```

### Custom Configuration

```yaml
---
- hosts: servers
  become: true
  roles:
    - role: common.usbipd
      vars:
        usbipd_listen_address: "192.168.1.100"
        usbipd_port: 3240
```

### Disable Autostart

```yaml
---
- hosts: servers
  become: true
  roles:
    - role: common.usbipd
      vars:
        usbipd_service_enabled: false
        usbipd_service_state: stopped
```

## What the Role Does

1. **Updates APT Cache**: Refreshes the package manager cache
2. **Detects Kernel Version**: Runs `uname -r` to get the current kernel release
3. **Installs Packages**: Installs `linux-tools` for the specific kernel and `hwdata`
4. **Deploys Service File**: Creates a systemd service unit file from template
5. **Enables Service**: Marks the service to start on boot
6. **Starts Service**: Immediately starts the USBIPD service

## Service Management

Once installed, you can manage the USBIPD service using standard systemd commands:

```bash
# Check service status
systemctl status usbipd

# Start the service
systemctl start usbipd

# Stop the service
systemctl stop usbipd

# Restart the service
systemctl restart usbipd

# View service logs
journalctl -u usbipd -f
```

## Systemd Service Details

The role creates a systemd service file at `/etc/systemd/system/usbipd.service` with:

- **Type**: simple (service runs in foreground)
- **After**: network.target (starts after network is ready)
- **Restart**: on-failure (automatically restarts if it crashes)
- **RestartSec**: 10 seconds (wait before restart)
- **WantedBy**: multi-user.target (starts during normal boot)

## Usage Examples

### Share USB Devices

Once USBIPD is running, you can use the `usbip` client to bind and share devices:

```bash
# List available USB devices
usbip list -l

# Bind a device (replace BUSID with actual bus ID)
sudo usbip bind -b BUSID

# Unbind a device
sudo usbip unbind -b BUSID

# List bound devices
usbip list -r localhost
```

### Monitor Service

View real-time logs:
```bash
journalctl -u usbipd -f
```

Check service status:
```bash
systemctl status usbipd
```

## Handlers

The role includes the following handlers:

- **reload systemd**: Reloads the systemd daemon configuration
- **restart usbipd**: Restarts the USBIPD service

## Notes

- The role requires root/sudo privileges to install packages and create system services
- Packages are installed from the default APT repositories
- The kernel-specific linux-tools package is automatically detected and installed
- The systemd service is configured with automatic restart on failure
- USBIPD requires appropriate hardware and kernel support

## Security Considerations

- USBIPD listens on port 3240 by default
- Ensure firewall rules appropriately restrict access
- Consider using a specific listen address instead of 0.0.0.0 for security
- USB device access should be properly controlled

## Firewall Configuration

To allow USBIPD traffic (example with UFW):

```bash
sudo ufw allow 3240/tcp
```

Or with iptables:

```bash
sudo iptables -A INPUT -p tcp --dport 3240 -j ACCEPT
```

## Troubleshooting

### Service fails to start

Check the logs:
```bash
journalctl -u usbipd -n 50
```

### Package installation fails

Ensure APT cache is updated:
```bash
sudo apt update
sudo apt install linux-tools-$(uname -r) hwdata
```

### Cannot find linux-tools package

This typically happens if the kernel version is not found. Install all linux-tools:
```bash
sudo apt install linux-tools-generic hwdata
```

### Verify service is running

```bash
systemctl is-active usbipd
systemctl is-enabled usbipd
```

### Check USBIPD is listening

```bash
sudo netstat -tlnp | grep usbipd
# or
sudo ss -tlnp | grep usbipd
```

## Dependencies

This role has no dependencies on other Ansible roles.

## License

This role is provided as-is for use with USBIPD.

USBIPD is part of the Linux kernel tools. For more information, visit the Linux kernel documentation.

## Links

- Linux USB/IP Protocol: https://www.kernel.org/doc/html/latest/usb/usbip_protocol.html
- USBIPD Documentation: https://github.com/torvalds/linux/tree/master/tools/usb/usbip
