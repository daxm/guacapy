# Guacapy

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)

A comprehensive Python client library for the Apache Guacamole REST API, providing an intuitive interface for managing Guacamole users, connections, groups, and permissions programmatically.

## Features

- 🏗️ **Manager Pattern**: Organized resource management through specialized manager classes
- 🔐 **Authentication**: Token-based authentication with optional TOTP 2FA support
- 📝 **Template Validation**: Pre-defined templates for RDP, SSH, and VNC connections
- 🔍 **Search & Discovery**: Name-based and regex search capabilities for connections and groups
- 📊 **Active Connections**: Monitor and terminate active user sessions
- 👥 **User & Group Management**: Complete CRUD operations for users and groups
- 🔗 **Connection Management**: Create and manage RDP, SSH, and VNC connections
- 🛡️ **Permission System**: Granular permission assignment for users and connections
- 📈 **Session Monitoring**: Real-time tracking of active connections and usage

## Installation

### From PyPI
```bash
pip install guacapy
```

### From Source
```bash
git clone https://github.com/pschmitt/guacapy.git
cd guacapy
pip install .
```

### Development Installation
```bash
git clone https://github.com/pschmitt/guacapy.git
cd guacapy
pip install -e .[dev]
```

## Quick Start

```python
from guacapy import Guacamole

# Initialize the client
client = Guacamole(
    hostname="guac.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443
)

# List all users
users = client.users.list()
print(f"Found {len(users)} users")

# Create a new SSH connection
from copy import deepcopy
ssh_payload = deepcopy(client.connections.SSH_TEMPLATE)
ssh_payload.update({
    "name": "My Linux Server",
    "parameters": {
        "hostname": "server.example.com",
        "port": "22",
        "username": "admin"
    }
})

connection = client.connections.create(ssh_payload)
print(f"Created connection: {connection['identifier']}")

# Assign user to connection
client.users.assign_connection("john_doe", connection["identifier"], "READ")
print("User assigned to connection")

# Logout when done
client.logout()
```

## Documentation

### 📚 User Guide (`docs/user_guide.md`)
**For:** Developers using guacapy in their applications

Complete guide with:
- Detailed usage examples for all manager classes
- Step-by-step workflows for common tasks
- Error handling patterns and best practices
- Real-world scenarios and troubleshooting

[Read the User Guide →](docs/user_guide.md)

### 🤖 AI Codebase Guide (`docs/ai_codebase_guide.md`)
**For:** AI assistants, code analysis tools, and deep codebase understanding

Comprehensive technical reference including:
- Architecture and design patterns
- Complete API method documentation
- Data structure specifications and templates
- HTTP endpoint mappings and response codes
- Parameter type mappings and return values
- AI-specific metadata and quick references

[Read the AI Codebase Guide →](docs/ai_codebase_guide.md)

### 🌐 Official API Documentation
**For:** Understanding the underlying Guacamole REST API

The guacapy library is based on the official Apache Guacamole REST API:
- [Guacamole REST API Documentation](https://github.com/ridvanaltun/guacamole-rest-api-documentation)

## Usage Examples

### User Management
```python
# Create a new user
user_payload = deepcopy(client.users.CREATE_USER_TEMPLATE)
user_payload.update({
    "username": "new_user",
    "password": "secure_password",
    "attributes": {
        "guac-full-name": "New User",
        "guac-email-address": "user@example.com"
    }
})
client.users.create(user_payload)

# Get user details and permissions
user_details = client.users.user_details("new_user")
permissions = client.users.user_permissions("new_user")
```

### Connection Management
```python
# Create RDP connection
rdp_payload = deepcopy(client.connections.RDP_TEMPLATE)
rdp_payload.update({
    "name": "Windows Server",
    "parameters": {
        "hostname": "windows.example.com",
        "port": "3389",
        "username": "Administrator"
    }
})
client.connections.create(rdp_payload)

# Find connections by name
servers = client.connections.get_by_name("server", regex=True)
```

### Group Management
```python
# Create user group
group_payload = deepcopy(client.user_groups.GROUP_TEMPLATE)
group_payload.update({"identifier": "developers"})
client.user_groups.create(group_payload)

# Add users to group
client.users.assign_usergroups("john_doe", "developers")
```

### Permission Management
```python
# Assign system permissions
client.permissions.assign_system_permission("admin", "CREATE_USER")

# Assign connection access
client.users.assign_connection("user", "connection_id", "READ")
```

### Active Connection Monitoring
```python
# List active connections
active = client.active_connections.list()
for conn_id, details in active.items():
    print(f"User {details['username']} connected to {details['remoteHost']}")

# Terminate connection
client.active_connections.kill("connection_id")
```

## Architecture

Guacapy follows a clean, modular architecture:

```
guacapy/
├── __init__.py          # Package exports
├── client.py            # Main Guacamole class
├── utilities.py         # Helper functions and utilities
└── managers/            # Resource management
    ├── base.py          # Abstract base manager
    ├── users.py         # User management
    ├── connections.py   # Connection management
    ├── connection_groups.py  # Group management
    ├── active_connections.py # Session monitoring
    ├── user_groups.py   # User group management
    ├── sharing_profiles.py   # Sharing profiles
    ├── schema.py        # Schema information
    └── permissions.py   # Permission management
```

### Key Components

- **Guacamole Class**: Main entry point providing access to all managers
- **Manager Pattern**: Each resource type has a dedicated manager class
- **Template System**: Pre-defined templates for connection creation
- **Utility Functions**: Centralized HTTP handling and validation

## Requirements

- **Python**: 3.9 or higher
- **Dependencies**: 
  - requests (2.32.5)
- **Guacamole Server**: Version 1.1.0+ (tested with 1.6.0)

## Development

### Setup
```bash
# Install development dependencies
pip install -e .[dev]

# Run tests
pytest

# Type checking
mypy guacapy/

# Linting
flake8 guacapy/

# Code formatting
black guacapy/
```

### Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests and linting
5. Submit a pull request

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## Authors

- **Philipp Schmitt** - *Initial work* - [philipp@schmitt.co](mailto:philipp@schmitt.co)
- **Dax Mickelson** - *Contributor* - [github@daxm.net](mailto:github@daxm.net)

## Support

- 📖 **Documentation**: Check the [User Guide](docs/user_guide.md) and [AI Codebase Guide](docs/ai_codebase_guide.md)
- 🐛 **Issues**: [GitHub Issues](https://github.com/yourusername/guacapy/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/yourusername/guacapy/discussions)

## Changelog

### v1.20251014.1
- Initial release with comprehensive API coverage
- Manager pattern implementation
- Template-based connection creation
- TOTP 2FA support
- Complete documentation suite
