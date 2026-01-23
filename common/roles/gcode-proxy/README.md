# gcode-proxy Role

This Ansible role installs and manages the `gcode-proxy` application from the GitHub repository. It handles git cloning, Python virtual environment setup, package installation via pyproject.toml, configuration file deployment, and systemd service management.

## Requirements

- Ubuntu/Debian-based system
- Python 3.6+
- Root or sudo access

## Role Variables

### Installation Configuration

- `gcode_proxy_git_repo` (default: `https://github.com/H3mul/gcode-proxy.git`)
  - Git repository URL for gcode-proxy

- `gcode_proxy_git_version` (default: `main`)
  - Git branch, tag, or commit to checkout. Leave empty to use repository default.

- `gcode_proxy_install_path` (default: `/opt/gcode-proxy`)
  - Base installation directory for gcode-proxy

### User and Service Configuration

- `gcode_proxy_user` (default: `gcode-proxy`)
  - System user to run the gcode-proxy service

- `gcode_proxy_group` (default: `gcode-proxy`)
  - System group for the gcode-proxy service

- `gcode_proxy_service_enabled` (default: `true`)
  - Whether to enable the systemd service on boot

- `gcode_proxy_service_state` (default: `started`)
  - Initial state of the service (started, stopped, restarted, reloaded)

- `gcode_proxy_service_restart` (default: `always`)
  - Systemd restart policy (no, always, on-success, on-failure, on-abnormal, on-abort, on-watchdog)

- `gcode_proxy_service_restart_sec` (default: `10`)
  - Seconds to wait before restarting the service

### Configuration File

- `gcode_proxy_config_path` (default: `/etc/gcode-proxy/config.yaml`)
  - Path where the configuration file will be deployed

- `gcode_proxy_config` (default: `{}`)
  - Configuration dictionary to be converted to YAML and deployed
  - Leave empty to skip config file creation

- `gcode_proxy_environment_file` (default: `""`)
  - Optional path to an environment file to be sourced by systemd
  - Leave empty to skip EnvironmentFile directive
  - Useful for setting environment variables like API keys, tokens, etc.
  - Example: `/etc/gcode-proxy/env` or `/etc/default/gcode-proxy`

## Example Playbook

### Basic Installation

```yaml
- hosts: all
  roles:
    - gcode-proxy
```

### With Custom Configuration

```yaml
- hosts: all
  roles:
    - role: gcode-proxy
      vars:
        gcode_proxy_git_version: "v1.0.0"
        gcode_proxy_config:
          port: 5000
          host: "0.0.0.0"
          devices:
            - name: "printer"
              path: "/dev/ttyUSB0"
              baudrate: 115200
```

### With Environment File

```yaml
- hosts: all
  roles:
    - role: gcode-proxy
      vars:
        gcode_proxy_environment_file: /etc/gcode-proxy/env
        gcode_proxy_config:
          port: 5000
          host: "0.0.0.0"
```

Create the environment file on the target system:
```bash
# /etc/gcode-proxy/env
API_KEY=your-secret-key
LOG_LEVEL=debug
```

### Custom Installation Path

```yaml
- hosts: all
  roles:
    - role: gcode-proxy
      vars:
        gcode_proxy_install_path: /usr/local/gcode-proxy
        gcode_proxy_config_path: /etc/gcode-proxy/config.yaml
        gcode_proxy_config:
          port: 8080
          debug: true
```

## Directory Structure

After installation, the directory structure will look like:

```
/opt/gcode-proxy/
├── src/                    # Cloned git repository
│   ├── pyproject.toml
│   ├── gcode_proxy/
│   └── ...
└── venv/                   # Python virtual environment
    ├── bin/
    │   ├── gcode-proxy-server
    │   └── ...
    ├── lib/
    └── ...

/etc/gcode-proxy/
└── config.yaml            # Configuration file (if provided)
```

## Service Management

Once installed, you can manage the service with standard systemd commands:

```bash
# Check service status
systemctl status gcode-proxy

# View logs
journalctl -u gcode-proxy -f

# Restart service
systemctl restart gcode-proxy

# Stop service
systemctl stop gcode-proxy

# Start service
systemctl start gcode-proxy
```

## Security Features

The systemd service includes the following security hardening:

- Runs as dedicated `gcode-proxy` user (unprivileged)
- Uses `ProtectSystem=strict` and `ProtectHome=true` for filesystem isolation
- Enables `NoNewPrivileges` to prevent privilege escalation
- Uses `PrivateTmp` for isolated temporary directory
- Restricted `ReadWritePaths` to only necessary directories
- Logs to journald for audit trail

## Handlers

This role includes the following handlers:

- `restart gcode-proxy` - Restarts the gcode-proxy service
- `reload systemd and restart gcode-proxy` - Reloads systemd daemon and restarts the service

These are automatically triggered when:
- The systemd service file is modified
- The configuration file is updated

## Notes

- The role uses `become_user` to ensure proper ownership of files and venv
- If no configuration is provided via `gcode_proxy_config`, the config file will not be created
- The virtual environment is created in `{{ gcode_proxy_install_path }}/venv`
- The git repository is cloned into `{{ gcode_proxy_install_path }}/src`

## License

This role is part of the h3mul Ansible collection.