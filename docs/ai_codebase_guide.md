# AI-Friendly Guacapy Codebase Documentation

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture & Design Patterns](#architecture--design-patterns)
3. [Core Classes & Components](#core-classes--components)
4. [Data Structures & Templates](#data-structures--templates)
5. [API Endpoints Mapping](#api-endpoints-mapping)
6. [Common Usage Patterns](#common-usage-patterns)
7. [Error Handling Patterns](#error-handling-patterns)
8. [Code Navigation Guide](#code-navigation-guide)
9. [Example Workflows](#example-workflows)
10. [AI-Specific Metadata](#ai-specific-metadata)

---

## Project Overview

**Package:** guacapy  
**Version:** 1.20251014.1 (date-based versioning: major.YYYYMMDD.patch)  
**Purpose:** Python client for Apache Guacamole REST API  
**Python Version:** 3.9+  
**Dependencies:** requests 2.32.5  
**Target API:** Guacamole REST API (tested with Guacamole 1.6.0)  
**API Documentation:** Based on unofficial documentation for Guacamole v1.1.0  
**License:** GPL-3.0  
**Authors:** Philipp Schmitt, Dax Mickelson  

### Development Dependencies
- **mypy** ^1.18.2 - Static type checking
- **black** ^25.9.0 - Code formatter  
- **flake8** ^7.3.0 - Linter  

### Key Features
- Complete REST API coverage for Guacamole
- Manager pattern for organized resource handling
- Template-based payload validation
- TOTP 2FA support
- Comprehensive error handling
- Type hints throughout

---

## Architecture & Design Patterns

### Core Components

1. **Guacamole Client** (`guacapy/client.py`)
   - Main entry point for API interactions
   - Handles authentication and session management
   - Provides property-based access to managers

2. **Manager Pattern** (`guacapy/managers/`)
   - 8 specialized managers for different resource types
   - All inherit from `BaseManager`
   - Consistent CRUD interface across all managers

3. **BaseManager** (`guacapy/managers/base.py`)
   - Abstract base class for all managers
   - Handles common initialization and datasource management

4. **Utility Functions** (`guacapy/utilities.py`)
   - Centralized HTTP request handling
   - Payload validation
   - Authentication helpers
   - Search utilities

### Key Design Patterns

1. **Property-Based Manager Access**
   ```python
   client = Guacamole(...)
   users = client.users        # UserManager instance
   connections = client.connections  # ConnectionManager instance
   ```

2. **Template-Based Validation**
   - Each manager defines templates for payload validation
   - Ensures API compatibility and reduces errors

3. **Centralized HTTP Handling**
   - All API requests go through `requester()` utility
   - Consistent error handling and logging
   - Automatic token management

4. **Token-Based Authentication**
   - Session tokens for API access
   - Optional TOTP 2FA support
   - Automatic token refresh handling

---

## Core Classes & Components

### Main Client Class

#### `Guacamole` (guacapy/client.py)

**Initialization Parameters:**
```python
Guacamole(
    hostname: str,                    # Guacamole server hostname
    username: str,                    # Authentication username
    password: str,                    # Authentication password
    secret: Optional[str] = None,     # TOTP secret for 2FA
    connection_protocol: str = "https", # http or https
    connection_port: int = 443,       # Server port
    base_url_path: str = "/",         # Base URL path
    default_datasource: Optional[str] = None, # Override datasource
    use_cookies: bool = False,        # Use cookies for auth
    ssl_verify: bool = False,         # SSL certificate verification
    logging_level: Optional[str] = None # Logging configuration
)
```

**Manager Properties:**
- `client.users` → `UserManager`
- `client.connections` → `ConnectionManager`
- `client.connection_groups` → `ConnectionGroupManager`
- `client.active_connections` → `ActiveConnectionManager`
- `client.user_groups` → `UserGroupManager`
- `client.sharing_profiles` → `SharingProfileManager`
- `client.schema` → `SchemaManager`
- `client.permissions` → `PermissionsManager`

**Session Management Methods:**
- `logout()` - Revoke authentication token
- `get_json_token(payload)` - JSON-based authentication

### Manager Classes (8 total)

#### 1. UserManager (guacapy/managers/users.py)

**Purpose:** User CRUD operations, permissions, groups, history

**Key Methods:**
- `list()` → `Dict[str, Any]` - List all users
- `user_details(username)` → `Dict[str, Any]` - Get user details
- `user_permissions(username)` → `Dict[str, Any]` - Get user permissions
- `user_effective_permissions(username)` → `Dict[str, Any]` - Get effective permissions
- `user_usergroups(username)` → `Dict[str, Any]` - Get user's groups
- `user_history(username)` → `Dict[str, Any]` - Get connection history
- `assign_usergroups(username, usergroup)` → `requests.Response` - Add user to group
- `revoke_usergroups(username, usergroup)` → `requests.Response` - Remove user from group
- `assign_connection(username, connection_id, permission, is_connection_group)` → `requests.Response`
- `revoke_connection(username, connection_id, permission, is_connection_group)` → `requests.Response`
- `update_password(username, old_password, new_password)` → `requests.Response`
- `self_details()` → `Dict[str, Any]` - Get authenticated user details
- `create(payload)` → `Dict[str, Any]` - Create new user
- `update(username, payload)` → `requests.Response` - Update user
- `delete(username)` → `requests.Response` - Delete user

#### 2. ConnectionManager (guacapy/managers/connections.py)

**Purpose:** Connection CRUD operations with protocol-specific templates

**Key Methods:**
- `list()` → `Dict[str, Any]` - List all connections
- `details(identifier)` → `Dict[str, Any]` - Get connection details
- `parameters(identifier)` → `Dict[str, Any]` - Get connection parameters
- `get_by_name(name, regex=False)` → `Optional[Dict[str, Any]]` - Find by name
- `create(payload)` → `Optional[Dict[str, Any]]` - Create connection
- `update(identifier, payload)` → `requests.Response` - Update connection
- `delete(identifier)` → `requests.Response` - Delete connection

**Protocol Templates:**
- `RDP_TEMPLATE` - Remote Desktop Protocol
- `SSH_TEMPLATE` - Secure Shell
- `VNC_TEMPLATE` - Virtual Network Computing

#### 3. ConnectionGroupManager (guacapy/managers/connection_groups.py)

**Purpose:** Organizational connection groups

**Key Methods:**
- `list()` → `Dict[str, Any]` - List all connection groups
- `details(identifier)` → `Dict[str, Any]` - Get group details
- `get_by_name(name, regex=False)` → `Optional[Dict[str, Any]]` - Find by name
- `create(payload)` → `Optional[Dict[str, Any]]` - Create group
- `update(identifier, payload)` → `requests.Response` - Update group
- `delete(identifier)` → `requests.Response` - Delete group

#### 4. ActiveConnectionManager (guacapy/managers/active_connections.py)

**Purpose:** Monitor and manage active connections

**Key Methods:**
- `list()` → `Dict[str, Any]` - List active connections
- `details(identifier)` → `Optional[Dict[str, Any]]` - Get connection details
- `kill(identifier)` → `Optional[requests.Response]` - Terminate connection

#### 5. UserGroupManager (guacapy/managers/user_groups.py)

**Purpose:** User group management

**Key Methods:**
- `list()` → `Dict[str, Any]` - List all user groups
- `details(identifier)` → `Dict[str, Any]` - Get group details
- `members(identifier)` → `Dict[str, Any]` - Get group members
- `edit_members(identifier, payload)` → `requests.Response` - Modify members
- `create(payload)` → `Optional[Dict[str, Any]]` - Create group
- `update(identifier, payload)` → `requests.Response` - Update group
- `delete(identifier)` → `Optional[requests.Response]` - Delete group

#### 6. SharingProfileManager (guacapy/managers/sharing_profiles.py)

**Purpose:** Connection sharing profiles

**Key Methods:**
- `list()` → `Dict[str, Any]` - List sharing profiles
- `details(identifier)` → `Dict[str, Any]` - Get profile details
- `parameters(identifier)` → `Dict[str, Any]` - Get profile parameters
- `create(payload)` → `Optional[Dict[str, Any]]` - Create profile
- `update(identifier, payload)` → `requests.Response` - Update profile
- `delete(identifier)` → `requests.Response` - Delete profile

#### 7. SchemaManager (guacapy/managers/schema.py)

**Purpose:** Protocol and attribute schemas

**Key Methods:**
- `protocols()` → `Dict[str, Any]` - Get supported protocols
- `user_attributes()` → `Dict[str, Any]` - Get user attributes

#### 8. PermissionsManager (guacapy/managers/permissions.py)

**Purpose:** System permissions management

**Key Methods:**
- `assign_system_permission(username, permission)` → `requests.Response`
- `revoke_system_permission(username, permission)` → `requests.Response`

### Utility Functions (guacapy/utilities.py)

**Core Functions:**
- `requester(guac_client, url, method, params, payload, token, verify, allow_redirects, cookies, json_response)` → `Union[requests.Response, Dict[str, Any]]`
- `validate_payload(payload, template, allow_partial)` → `None`
- `get_totp_token(secret)` → `str`
- `get_hotp_token(secret, intervals_no)` → `int`
- `configure_logging(level, logger_name, log_file)` → `None`

**Search Functions:**
- `_find_by_name(guac_client, data, name, key, child_key, regex)` → `Optional[Dict[str, Any]]`
- `_find_connection_by_name(guac_client, connection_data, name, regex)` → `Optional[Dict[str, Any]]`
- `_find_connection_group_by_name(guac_client, group_data, name, regex)` → `Optional[Dict[str, Any]]`
- `get_connection_group_by_name(guac_client, name, regex, datasource)` → `Optional[Dict[str, Any]]`

---

## Data Structures & Templates

### Connection Templates

#### RDP Template
```python
ConnectionManager.RDP_TEMPLATE = {
    "name": "",
    "parentIdentifier": "ROOT",
    "protocol": "rdp",
    "attributes": {
        "max-connections": "",
        "max-connections-per-user": "",
        "weight": "",
        "failover-only": "",
        "guacd-port": "",
        "guacd-encryption": "",
        "guacd-hostname": "",
    },
    "parameters": {
        "hostname": "",
        "port": "3389",
        "username": "",
        "password": "",
        "security": "rdp",
        "disable-audio": "",
        "server-layout": "",
        "domain": "",
        "enable-font-smoothing": "",
        "ignore-cert": "",
        "console": "",
        "width": "",
        "height": "",
        "dpi": "",
        "color-depth": "",
        "console-audio": "",
        "enable-printing": "",
        "enable-drive": "",
        "create-drive-path": "",
        "enable-wallpaper": "",
        "enable-theming": "",
        "enable-full-window-drag": "",
        "enable-desktop-composition": "",
        "enable-menu-animations": "",
        "preconnection-id": "",
        "enable-sftp": "",
        "sftp-port": "",
    },
}
```

#### SSH Template
```python
ConnectionManager.SSH_TEMPLATE = {
    "name": "",
    "parentIdentifier": "ROOT",
    "protocol": "ssh",
    "attributes": {
        "max-connections": "",
        "max-connections-per-user": "",
    },
    "parameters": {
        "hostname": "",
        "port": "22",
        "username": "",
        "password": "",
    },
}
```

#### VNC Template
```python
ConnectionManager.VNC_TEMPLATE = {
    "name": "",
    "parentIdentifier": "ROOT",
    "protocol": "vnc",
    "attributes": {
        "max-connections": "",
        "max-connections-per-user": "",
        "weight": "",
        "failover-only": "",
        "guacd-port": "",
        "guacd-encryption": "",
        "guacd-hostname": "",
    },
    "parameters": {
        "hostname": "",
        "port": "5900",
        "password": "",
        "read-only": "",
        "swap-red-blue": "",
        "cursor": "",
        "color-depth": "",
        "clipboard-encoding": "",
        "dest-port": "",
        "create-recording-path": "",
        "enable-sftp": "",
        "sftp-port": "",
        "sftp-server-alive-interval": "",
        "enable-audio": "",
    },
}
```

### User Templates

#### Create User Template
```python
UserManager.CREATE_USER_TEMPLATE = {
    "username": "",
    "password": "",
    "attributes": {
        "disabled": "",
        "expired": "",
        "access-window-start": "",
        "access-window-end": "",
        "valid-from": "",
        "valid-until": "",
        "timezone": None,
        "guac-full-name": "",
        "guac-organization": "",
        "guac-organizational-role": "",
        "guac-email-address": "",
    },
}
```

#### Update User Template
```python
UserManager.UPDATE_USER_TEMPLATE = {
    "username": "",
    "attributes": {
        "disabled": "",
        "expired": "",
        "access-window-start": "",
        "access-window-end": "",
        "valid-from": "",
        "valid-until": "",
        "timezone": None,
        "guac-full-name": "",
        "guac-organization": "",
        "guac-organizational-role": "",
        "guac-email-address": "",
    },
}
```

### Other Templates

#### Connection Group Template
```python
ConnectionGroupManager.ORG_TEMPLATE = {
    "name": "",
    "parentIdentifier": "ROOT",
    "type": "ORGANIZATIONAL",
    "attributes": {
        "max-connections": "",
        "max-connections-per-user": "",
    },
}
```

#### User Group Template
```python
UserGroupManager.GROUP_TEMPLATE = {
    "identifier": "",
    "attributes": {"disabled": ""},
}
```

#### Sharing Profile Template
```python
SharingProfileManager.PROFILE_TEMPLATE = {
    "name": "",
    "primaryConnectionIdentifier": "",
    "parameters": {"read-only": ""},
    "attributes": {},
}
```

---

## API Endpoints Mapping

### Base URL Structure
```
{protocol}://{hostname}:{port}{base_path}api/session/data/{datasource}
```

### Manager Endpoints

#### UserManager
- `GET /users` → `list()`
- `GET /users/{username}` → `user_details(username)`
- `GET /users/{username}/permissions` → `user_permissions(username)`
- `GET /users/{username}/effectivePermissions` → `user_effective_permissions(username)`
- `GET /users/{username}/userGroups` → `user_usergroups(username)`
- `GET /users/{username}/history` → `user_history(username)`
- `PATCH /users/{username}/userGroups` → `assign_usergroups()`, `revoke_usergroups()`
- `PATCH /users/{username}/permissions` → `assign_connection()`, `revoke_connection()`
- `PUT /users/{username}/password` → `update_password()`
- `GET /self` → `self_details()`
- `POST /users` → `create()`
- `PUT /users/{username}` → `update()`
- `DELETE /users/{username}` → `delete()`

#### ConnectionManager
- `GET /connections` → `list()`
- `GET /connections/{identifier}` → `details(identifier)`
- `GET /connections/{identifier}/parameters` → `parameters(identifier)`
- `POST /connections` → `create()`
- `PUT /connections/{identifier}` → `update()`
- `DELETE /connections/{identifier}` → `delete()`

#### ConnectionGroupManager
- `GET /connectionGroups` → `list()`
- `GET /connectionGroups/{identifier}` → `details(identifier)`
- `POST /connectionGroups` → `create()`
- `PUT /connectionGroups/{identifier}` → `update()`
- `DELETE /connectionGroups/{identifier}` → `delete()`

#### ActiveConnectionManager
- `GET /activeConnections` → `list()`
- `GET /activeConnections/{identifier}` → `details(identifier)`
- `DELETE /activeConnections/{identifier}` → `kill(identifier)`

#### UserGroupManager
- `GET /userGroups` → `list()`
- `GET /userGroups/{identifier}` → `details(identifier)`
- `GET /userGroups/{identifier}/memberUsers` → `members(identifier)`
- `PATCH /userGroups/{identifier}/memberUsers` → `edit_members()`
- `POST /userGroups` → `create()`
- `PUT /userGroups/{identifier}` → `update()`
- `DELETE /userGroups/{identifier}` → `delete()`

#### SharingProfileManager
- `GET /sharingProfiles` → `list()`
- `GET /sharingProfiles/{identifier}` → `details(identifier)`
- `GET /sharingProfiles/{identifier}/parameters` → `parameters(identifier)`
- `POST /sharingProfiles` → `create()`
- `PUT /sharingProfiles/{identifier}` → `update()`
- `DELETE /sharingProfiles/{identifier}` → `delete()`

#### SchemaManager
- `GET /schema/protocols` → `protocols()`
- `GET /schema/userAttributes` → `user_attributes()`

#### PermissionsManager
- `PATCH /users/{username}/permissions` → `assign_system_permission()`, `revoke_system_permission()`

### Authentication Endpoints
- `POST /tokens` → Authentication
- `DELETE /tokens/{token}` → Logout

### HTTP Response Codes
- `200 OK` - Successful GET requests
- `204 No Content` - Successful PUT/PATCH/DELETE requests
- `400 Bad Request` - Invalid payload or duplicate resource
- `401 Unauthorized` - Authentication failure
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error (known issue with UserGroupManager.delete())

---

## Common Usage Patterns

### Initialization
```python
from guacapy import Guacamole

# Basic initialization
client = Guacamole(
    hostname="guac.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443
)

# With 2FA
client = Guacamole(
    hostname="guacamole.example.com",
    username="admin",
    password="secret",
    secret="TOTP_SECRET_KEY",
)

# With custom settings
client = Guacamole(
    hostname="guacamole.example.com",
    username="admin",
    password="secret",
    connection_protocol="https",
    connection_port=443,
    ssl_verify=True,
    logging_level="DEBUG"
)
```

### CRUD Operations Pattern

#### List Resources
```python
# List all users
users = client.users.list()

# List all connections
connections = client.connections.list()

# List connection groups
groups = client.connection_groups.list()
```

#### Get Details
```python
# Get user details
user = client.users.user_details("username")

# Get connection details
connection = client.connections.details("connection_id")

# Get group details
group = client.connection_groups.details("group_id")
```

#### Create Resources
```python
from copy import deepcopy

# Create user
user_payload = deepcopy(UserManager.CREATE_USER_TEMPLATE)
user_payload.update({
    "username": "newuser",
    "password": "password123",
    "attributes": {
        "guac-full-name": "New User",
        "guac-email-address": "user@example.com"
    }
})
new_user = client.users.create(user_payload)

# Create SSH connection
ssh_payload = deepcopy(ConnectionManager.SSH_TEMPLATE)
ssh_payload.update({
    "name": "My SSH Server",
    "parameters": {
        "hostname": "test_server",
        "port": "22",
        "username": "admin"
    }
})
new_connection = client.connections.create(ssh_payload)
```

#### Update Resources
```python
# Update user
update_payload = deepcopy(UserManager.UPDATE_USER_TEMPLATE)
update_payload.update({
    "username": "existinguser",
    "attributes": {
        "guac-email-address": "newemail@example.com"
    }
})
client.users.update("existinguser", update_payload)

# Update connection
update_payload = deepcopy(ConnectionManager.SSH_TEMPLATE)
update_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server"
    }
})
client.connections.update("connection_id", update_payload)
```

#### Delete Resources
```python
# Delete user
client.users.delete("username")

# Delete connection
client.connections.delete("connection_id")

# Delete group
client.connection_groups.delete("group_id")
```

### Permission Management

#### User Permissions
```python
# Assign user to connection
client.users.assign_connection("username", "connection_id", "READ")

# Assign user to connection group
client.users.assign_connection("username", "group_id", "READ", is_connection_group=True)

# Revoke connection access
client.users.revoke_connection("username", "connection_id", "READ")
```

#### System Permissions
```python
# Assign system permission
client.permissions.assign_system_permission("username", "CREATE_USER")

# Revoke system permission
client.permissions.revoke_system_permission("username", "CREATE_USER")
```

#### User Groups
```python
# Add user to group
client.users.assign_usergroups("username", "group_name")

# Remove user from group
client.users.revoke_usergroups("username", "group_name")
```

### Search Operations
```python
# Find connection by name
connection = client.connections.get_by_name("My Server")

# Find connection group by name
group = client.connection_groups.get_by_name("Production")

# Find with regex
connections = client.connections.get_by_name("server.*", regex=True)
```

### Active Connection Management
```python
# List active connections
active = client.active_connections.list()

# Get active connection details
details = client.active_connections.details("connection_id")

# Terminate active connection
client.active_connections.kill("connection_id")
```

### Session Management
```python
# Get authenticated user details
self_info = client.users.self_details()

# Logout
client.logout()
```

---

## Error Handling Patterns

### HTTP Error Codes

#### 401 Unauthorized
- **Cause:** Invalid credentials or expired token
- **Handling:** Re-authenticate or check credentials
- **Example:**
```python
try:
    users = client.users.list()
except requests.HTTPError as e:
    if e.response.status_code == 401:
        print("Authentication failed - check credentials")
```

#### 403 Forbidden
- **Cause:** Insufficient permissions
- **Handling:** Check user permissions or use admin account
- **Example:**
```python
try:
    client.users.create(payload)
except requests.HTTPError as e:
    if e.response.status_code == 403:
        print("Insufficient permissions for this operation")
```

#### 404 Not Found
- **Cause:** Resource doesn't exist
- **Handling:** Check resource identifier or existence
- **Example:**
```python
try:
    user = client.users.user_details("nonexistent")
except requests.HTTPError as e:
    if e.response.status_code == 404:
        print("User not found")
```

#### 400 Bad Request
- **Cause:** Invalid payload or duplicate resource
- **Handling:** Validate payload or check for duplicates
- **Example:**
```python
try:
    user = client.users.create(payload)
except requests.HTTPError as e:
    if e.response.status_code == 400:
        print("Invalid payload or user already exists")
```

#### 500 Internal Server Error
- **Cause:** Server error (known issue with UserGroupManager.delete())
- **Handling:** Check for known issues or retry
- **Example:**
```python
try:
    client.user_groups.delete("group_name")
except requests.HTTPError as e:
    if e.response.status_code == 500:
        print("Known issue with user group deletion - manual SQL required")
```

### Optional Return Values
Many methods return `Optional` types for non-critical failures:

```python
# Active connection might not exist
connection = client.active_connections.details("id")
if connection is None:
    print("Connection not active")

# Connection creation might fail due to duplicate
new_conn = client.connections.create(payload)
if new_conn is None:
    print("Connection already exists")
```

### Known API Quirks

1. **UserGroupManager.delete() 500 Error**
   - **Issue:** SQL syntax error in Guacamole's MySQL JDBC module
   - **Workaround:** Manual SQL deletion
   - **SQL:** `DELETE FROM guacamole_entity WHERE type = 'USER_GROUP' AND name = 'groupname'`
   - **Reference:** GUACAMOLE-2088

2. **Connection Creation 400 Error**
   - **Issue:** Duplicate connection names
   - **Handling:** Check existing connections before creation

---

## Code Navigation Guide

### File Structure
```
guacapy/
├── __init__.py              # Package exports (Guacamole class)
├── client.py                # Main Guacamole client class
├── utilities.py             # Helper functions and utilities
└── managers/
    ├── __init__.py          # Manager class exports
    ├── base.py              # BaseManager abstract class
    ├── users.py             # UserManager
    ├── connections.py       # ConnectionManager
    ├── connection_groups.py # ConnectionGroupManager
    ├── active_connections.py # ActiveConnectionManager
    ├── user_groups.py       # UserGroupManager
    ├── sharing_profiles.py  # SharingProfileManager
    ├── schema.py            # SchemaManager
    └── permissions.py       # PermissionsManager
```

### Key Entry Points

1. **Package Import**
   ```python
   from guacapy import Guacamole
   ```

2. **Client Class**
   - File: `guacapy/client.py`
   - Class: `Guacamole`
   - Lines: 58-341

3. **Manager Implementations**
   - Base: `guacapy/managers/base.py:BaseManager`
   - Users: `guacapy/managers/users.py:UserManager`
   - Connections: `guacapy/managers/connections.py:ConnectionManager`
   - Connection Groups: `guacapy/managers/connection_groups.py:ConnectionGroupManager`
   - Active Connections: `guacapy/managers/active_connections.py:ActiveConnectionManager`
   - User Groups: `guacapy/managers/user_groups.py:UserGroupManager`
   - Sharing Profiles: `guacapy/managers/sharing_profiles.py:SharingProfileManager`
   - Schema: `guacapy/managers/schema.py:SchemaManager`
   - Permissions: `guacapy/managers/permissions.py:PermissionsManager`

4. **Utility Functions**
   - File: `guacapy/utilities.py`
   - Key functions: `requester()`, `validate_payload()`, `get_totp_token()`

### Import Patterns
```python
# Main client
from guacapy import Guacamole

# Individual managers (if needed)
from guacapy.managers import UserManager, ConnectionManager

# Utilities (if needed)
from guacapy.utilities import requester, validate_payload
```

### Class Hierarchy
```
BaseManager (abstract)
├── UserManager
├── ConnectionManager
├── ConnectionGroupManager
├── ActiveConnectionManager
├── UserGroupManager
├── SharingProfileManager
├── SchemaManager
└── PermissionsManager
```

---

## Example Workflows

### 1. User Creation and Permission Assignment

```python
from guacapy import Guacamole
from copy import deepcopy

# Initialize client
client = Guacamole(
    hostname="guacamole.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443,
)

# Create new user
user_payload = deepcopy(UserManager.CREATE_USER_TEMPLATE)
user_payload.update({
    "username": "john_doe",
    "password": "secure_password",
    "attributes": {
        "guac-full-name": "John Doe",
        "guac-email-address": "john.doe@company.com",
        "guac-organization": "IT Department"
    }
})

new_user = client.users.create(user_payload)
print(f"Created user: {new_user}")

# Create SSH connection
ssh_payload = deepcopy(ConnectionManager.SSH_TEMPLATE)
ssh_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "22",
        "username": "admin"
    }
})

new_connection = client.connections.create(ssh_payload)
print(f"Created connection: {new_connection}")

# Assign user to connection
client.users.assign_connection("john_doe", new_connection["identifier"], "READ")
print("Assigned user to connection")

# Assign system permission
client.permissions.assign_system_permission("john_doe", "CREATE_CONNECTION")
print("Assigned system permission")
```

### 2. Connection Setup (RDP/SSH/VNC)

```python
from guacapy import Guacamole
from copy import deepcopy

client = Guacamole(
    hostname="guac.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443
)

# Create RDP connection
rdp_payload = deepcopy(ConnectionManager.RDP_TEMPLATE)
rdp_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "3389",
        "username": "Administrator",
        "password": "admin_password",
        "security": "rdp",
        "enable-drive": "true",
        "enable-printing": "true"
    }
})

rdp_connection = client.connections.create(rdp_payload)
print(f"Created RDP connection: {rdp_connection}")

# Create SSH connection
ssh_payload = deepcopy(ConnectionManager.SSH_TEMPLATE)
ssh_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "22",
        "username": "root"
    }
})

ssh_connection = client.connections.create(ssh_payload)
print(f"Created SSH connection: {ssh_connection}")

# Create VNC connection
vnc_payload = deepcopy(ConnectionManager.VNC_TEMPLATE)
vnc_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "5900",
        "password": "vnc_password",
        "read-only": "false"
    }
})

vnc_connection = client.connections.create(vnc_payload)
print(f"Created VNC connection: {vnc_connection}")
```

### 3. Group Management

```python
from guacapy import Guacamole
from copy import deepcopy

client = Guacamole(
    hostname="guac.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443
)

# Create connection group
group_payload = deepcopy(ConnectionGroupManager.ORG_TEMPLATE)
group_payload.update({
    "name": "Example CG",
    "parentIdentifier": "ROOT"
})

new_group = client.connection_groups.create(group_payload)
print(f"Created connection group: {new_group}")

# Create user group
user_group_payload = deepcopy(UserGroupManager.GROUP_TEMPLATE)
user_group_payload.update({
    "identifier": "example_group"
})

new_user_group = client.user_groups.create(user_group_payload)
print(f"Created user group: {new_user_group}")

# Add users to group
client.users.assign_usergroups("john_doe", "example_group")
client.users.assign_usergroups("jane_smith", "example_group")
print("Added users to group")

# Get group members
members = client.user_groups.members("example_group")
print(f"Group members: {members}")
```

### 4. Active Connection Monitoring

```python
from guacapy import Guacamole
import time

client = Guacamole(
    hostname="guac.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443
)

# Monitor active connections
def monitor_connections():
    active_connections = client.active_connections.list()
    
    if not active_connections:
        print("No active connections")
        return
    
    print(f"Active connections: {len(active_connections)}")
    
    for conn_id, details in active_connections.items():
        print(f"Connection {conn_id}:")
        print(f"  User: {details.get('username', 'Unknown')}")
        print(f"  Remote Host: {details.get('remoteHost', 'Unknown')}")
        print(f"  Start Date: {details.get('startDate', 'Unknown')}")
        
        # Get detailed information
        full_details = client.active_connections.details(conn_id)
        if full_details:
            print(f"  Full details: {full_details}")

# Monitor for 5 minutes
for i in range(5):
    print(f"\n--- Monitoring cycle {i+1} ---")
    monitor_connections()
    time.sleep(60)  # Wait 1 minute

# Terminate all active connections (if needed)
active_connections = client.active_connections.list()
for conn_id in active_connections.keys():
    result = client.active_connections.kill(conn_id)
    if result:
        print(f"Terminated connection {conn_id}")
    else:
        print(f"Failed to terminate connection {conn_id}")
```

### 5. Session Management

```python
from guacapy import Guacamole

# Initialize with 2FA
client = Guacamole(
    hostname="guacamole.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443
    secret="TOTP_SECRET_KEY",  # Base32 encoded secret
)

# Get current user details
self_info = client.users.self_details()
print(f"Authenticated as: {self_info['username']}")
print(f"Last active: {self_info.get('lastActive', 'Unknown')}")

# Get available data sources
print(f"Available data sources: {client.data_sources}")
print(f"Primary data source: {client.primary_datasource}")

# JSON-based authentication (for extensions)
json_payload = {
    "username": "admin",
    "password": "admin_password"
}
json_token = client.get_json_token(json_payload)
print(f"JSON token: {json_token}")

# Logout when done
client.logout()
print("Logged out successfully")
```

---

## AI-Specific Metadata

### Quick Reference Tables

#### Manager Methods Summary

| Manager | List | Details | Create | Update | Delete | Special Methods |
|---------|------|---------|--------|--------|--------|-----------------|
| UserManager | ✓ | user_details() | ✓ | ✓ | ✓ | permissions, groups, history, password |
| ConnectionManager | ✓ | details() | ✓ | ✓ | ✓ | parameters, get_by_name |
| ConnectionGroupManager | ✓ | details() | ✓ | ✓ | ✓ | get_by_name |
| ActiveConnectionManager | ✓ | details() | - | - | kill() | - |
| UserGroupManager | ✓ | details() | ✓ | ✓ | ✓ | members, edit_members |
| SharingProfileManager | ✓ | details() | ✓ | ✓ | ✓ | parameters |
| SchemaManager | - | - | - | - | - | protocols, user_attributes |
| PermissionsManager | - | - | - | - | - | assign_system_permission, revoke_system_permission |

#### Parameter Type Mappings

| Parameter Type | Python Type | Description |
|----------------|-------------|-------------|
| username | str | User identifier |
| identifier | str | Resource identifier |
| payload | Dict[str, Any] | Request payload |
| permission | str | Permission name (READ, WRITE, ADMINISTER, etc.) |
| datasource | str | Data source identifier (mysql, postgresql) |
| regex | bool | Use regex matching |
| is_connection_group | bool | Target connection group vs connection |

#### Return Type Specifications

| Method Pattern | Return Type | Description |
|----------------|-------------|-------------|
| list() | Dict[str, Any] | Dictionary mapping IDs to resource details |
| details(identifier) | Dict[str, Any] | Single resource details |
| create(payload) | Optional[Dict[str, Any]] | Created resource or None if duplicate |
| update(identifier, payload) | requests.Response | HTTP response (204 No Content) |
| delete(identifier) | requests.Response | HTTP response (204 No Content) |
| search methods | Optional[Dict[str, Any]] | Found resource or None |

#### Common Exceptions

| Exception | HTTP Code | Cause | Handling |
|-----------|-----------|-------|----------|
| GuacamoleError | - | Authentication failure | Check credentials |
| requests.HTTPError | 401 | Unauthorized | Re-authenticate |
| requests.HTTPError | 403 | Forbidden | Check permissions |
| requests.HTTPError | 404 | Not Found | Check resource exists |
| requests.HTTPError | 400 | Bad Request | Validate payload |
| requests.HTTPError | 500 | Server Error | Check known issues |
| ValueError | - | Invalid payload | Use templates |

#### Template Usage Patterns

| Template | Usage | Required Fields | Optional Fields |
|----------|-------|----------------|-----------------|
| CREATE_USER_TEMPLATE | User creation | username, password | attributes |
| UPDATE_USER_TEMPLATE | User updates | username | attributes |
| RDP_TEMPLATE | RDP connections | name, parameters.hostname | All other parameters |
| SSH_TEMPLATE | SSH connections | name, parameters.hostname | parameters.port, username |
| VNC_TEMPLATE | VNC connections | name, parameters.hostname | All other parameters |
| ORG_TEMPLATE | Connection groups | name | parentIdentifier, attributes |
| GROUP_TEMPLATE | User groups | identifier | attributes |
| PROFILE_TEMPLATE | Sharing profiles | name, primaryConnectionIdentifier | parameters, attributes |

#### API Endpoint Patterns

| HTTP Method | Pattern | Purpose |
|-------------|---------|---------|
| GET | /{resource} | List resources |
| GET | /{resource}/{id} | Get resource details |
| GET | /{resource}/{id}/{subresource} | Get subresource |
| POST | /{resource} | Create resource |
| PUT | /{resource}/{id} | Update resource |
| PATCH | /{resource}/{id}/{subresource} | Modify subresource |
| DELETE | /{resource}/{id} | Delete resource |

#### Authentication Flow

1. **Initial Authentication**
   - POST /api/tokens with username/password
   - Optional TOTP token for 2FA
   - Returns authToken and availableDataSources

2. **API Requests**
   - Include token as query parameter
   - All requests go through requester() utility
   - Automatic error handling and logging

3. **Session Management**
   - Token stored in client.token
   - DELETE /api/tokens/{token} for logout
   - Token cleared on logout

#### Data Source Handling

- **Primary Data Source**: Set during authentication or via default_datasource parameter
- **Available Data Sources**: Retrieved during authentication
- **Manager Data Source**: Each manager uses client.primary_datasource by default
- **Override**: Managers can accept custom datasource parameter

#### Search Functionality

- **Name-based Search**: Available for connections and connection groups
- **Regex Support**: Optional regex matching for flexible searches
- **Recursive Search**: Handles nested connection groups
- **Utility Functions**: _find_by_name, _find_connection_by_name, _find_connection_group_by_name

This documentation provides comprehensive coverage of the guacapy codebase, enabling AI assistants to quickly understand the architecture, patterns, and usage of this Guacamole REST API client library.
