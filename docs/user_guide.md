# Guacapy User Guide

A comprehensive guide to using the Guacapy Python library for managing Apache Guacamole instances.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Main Guacamole Class](#main-guacamole-class)
3. [User Management](#user-management)
4. [Connection Management](#connection-management)
5. [Connection Groups](#connection-groups)
6. [User Groups](#user-groups)
7. [Sharing Profiles](#sharing-profiles)
8. [Active Connections](#active-connections)
9. [Schema Information](#schema-information)
10. [Permissions Management](#permissions-management)
11. [Common Workflows](#common-workflows)
12. [Error Handling](#error-handling)

---

## Getting Started

### Installation

```bash
pip install guacapy
```

### Basic Setup

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
```

### Authentication Options

```python
# With 2FA (TOTP)
client = Guacamole(
    hostname="guac.example.com",
    username="guacadmin",
    password="guacpass",
    secret="YOUR_TOTP_SECRET",  # Base32 encoded secret
    connection_protocol="https",
    ssl_verify=False,
    connection_port=8443
)

# With custom settings
client = Guacamole(
    hostname="guac.example.com",
    username="guacadmin",
    password="guacpass",
    connection_protocol="https",
    connection_port=8443,
    ssl_verify=True,
    logging_level="DEBUG"
)
```

---

## Main Guacamole Class

The `Guacamole` class is your main entry point to interact with the Guacamole API. It provides access to all managers through properties.

### Properties

- `client.users` - User management
- `client.connections` - Connection management
- `client.connection_groups` - Connection group management
- `client.user_groups` - User group management
- `client.sharing_profiles` - Sharing profile management
- `client.active_connections` - Active connection monitoring
- `client.schema` - Schema information
- `client.permissions` - Permission management

### Session Management

```python
# Get current user details
current_user = client.users.self_details()
print(f"Logged in as: {current_user['username']}")

# Logout when done
client.logout()
```

---

## User Management

Access user management through `client.users`.

### List All Users

```python
users = client.users.list()
for username, details in users.items():
    print(f"User: {username}")
    print(f"  Full Name: {details['attributes'].get('guac-full-name', 'N/A')}")
    print(f"  Email: {details['attributes'].get('guac-email-address', 'N/A')}")
    print(f"  Disabled: {details.get('disabled', False)}")
```

### Get User Details

```python
user_details = client.users.user_details("john_doe")
print(f"User: {user_details['username']}")
print(f"Last Active: {user_details.get('lastActive', 'Never')}")
print(f"Attributes: {user_details['attributes']}")
```

### Create a New User

```python
from copy import deepcopy

# Create user payload
user_payload = deepcopy(client.users.CREATE_USER_TEMPLATE)
user_payload.update({
    "username": "john_doe",
    "password": "super_secret_pass",
    "attributes": {
        "guac-full-name": "John Doe",
        "guac-email-address": "john.doe@example.com",
        "guac-organization": "Example Company"
    }
})

# Create the user
new_user = client.users.create(user_payload)
print(f"Created user: {new_user}")
```

### Update User Information

```python
# Update user attributes
update_payload = deepcopy(client.users.UPDATE_USER_TEMPLATE)
update_payload.update({
    "username": "john_doe",
    "attributes": {
        "guac-email-address": "john.doe@example.com",
        "guac-full-name": "John Doe"
    }
})

client.users.update("john_doe", update_payload)
print("User updated successfully")
```

### Change User Password

```python
client.users.update_password(
    username="john_doe",
    old_password="old_password",
    new_password="super_secret_pass"
)
print("Password updated successfully")
```

### Get User Permissions

```python
# Get explicit permissions
permissions = client.users.user_permissions("john_doe")
print(f"Connection permissions: {permissions['connectionPermissions']}")
print(f"System permissions: {permissions['systemPermissions']}")

# Get effective permissions (including inherited)
effective_perms = client.users.user_effective_permissions("john_doe")
print(f"Effective permissions: {effective_perms}")
```

### Get User Groups

```python
user_groups = client.users.user_usergroups("john_doe")
print(f"User belongs to groups: {user_groups}")
```

### Get User Connection History

```python
history = client.users.user_history("john_doe")
for connection in history:
    print(f"Connection: {connection['connectionIdentifier']}")
    print(f"Start Date: {connection['startDate']}")
    print(f"Remote Host: {connection['remoteHost']}")
```

### Delete User

```python
client.users.delete("unwanted_user")
print("User deleted successfully")
```

---

## Connection Management

Access connection management through `client.connections`.

### List All Connections

```python
connections = client.connections.list()
for conn_id, details in connections.items():
    print(f"Connection {conn_id}: {details['name']}")
    print(f"  Protocol: {details['protocol']}")
    print(f"  Parent: {details.get('parentIdentifier', 'ROOT')}")
```

### Get Connection Details

```python
connection = client.connections.details("connection_id")
print(f"Name: {connection['name']}")
print(f"Protocol: {connection['protocol']}")
print(f"Parameters: {connection['parameters']}")
```

### Get Connection Parameters

```python
params = client.connections.parameters("connection_id")
print(f"Hostname: {params.get('hostname', 'N/A')}")
print(f"Port: {params.get('port', 'N/A')}")
print(f"Username: {params.get('username', 'N/A')}")
```

### Find Connection by Name

```python
# Exact match
connection = client.connections.get_by_name("My Server")
if connection:
    print(f"Found connection: {connection['identifier']}")

# Regex match
connections = client.connections.get_by_name("server.*", regex=True)
print(f"Found {len(connections)} matching connections")
```

### Create SSH Connection

```python
from copy import deepcopy

ssh_payload = deepcopy(client.connections.SSH_TEMPLATE)
ssh_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "22",
        "username": "admin"
    }
})

new_connection = client.connections.create(ssh_payload)
print(f"Created SSH connection: {new_connection}")
```

### Create RDP Connection

```python
rdp_payload = deepcopy(client.connections.RDP_TEMPLATE)
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

new_connection = client.connections.create(rdp_payload)
print(f"Created RDP connection: {new_connection}")
```

### Create VNC Connection

```python
vnc_payload = deepcopy(client.connections.VNC_TEMPLATE)
vnc_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "5900",
        "password": "vnc_password",
        "read-only": "false"
    }
})

new_connection = client.connections.create(vnc_payload)
print(f"Created VNC connection: {new_connection}")
```

### Update Connection

```python
update_payload = deepcopy(client.connections.SSH_TEMPLATE)
update_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "22",
        "username": "admin"
    }
})

client.connections.update("connection_id", update_payload)
print("Connection updated successfully")
```

### Delete Connection

```python
client.connections.delete("connection_id")
print("Connection deleted successfully")
```

---

## Connection Groups

Access connection group management through `client.connection_groups`.

### List All Connection Groups

```python
groups = client.connection_groups.list()
for group_id, details in groups.items():
    print(f"Group {group_id}: {details['name']}")
    print(f"  Type: {details['type']}")
    print(f"  Parent: {details.get('parentIdentifier', 'ROOT')}")
```

### Get Connection Group Details

```python
group = client.connection_groups.details("group_id")
print(f"Name: {group['name']}")
print(f"Type: {group['type']}")
print(f"Child Connections: {len(group.get('childConnections', []))}")
print(f"Child Groups: {len(group.get('childConnectionGroups', []))}")
```

### Find Connection Group by Name

```python
group = client.connection_groups.get_by_name("Production Servers")
if group:
    print(f"Found group: {group['identifier']}")
```

### Create Connection Group

```python
from copy import deepcopy

group_payload = deepcopy(client.connection_groups.ORG_TEMPLATE)
group_payload.update({
    "name": "Example CG",
    "parentIdentifier": "ROOT"
})

new_group = client.connection_groups.create(group_payload)
print(f"Created connection group: {new_group}")
```

### Update Connection Group

```python
update_payload = deepcopy(client.connection_groups.ORG_TEMPLATE)
update_payload.update({
    "name": "Example CG"
})

client.connection_groups.update("group_id", update_payload)
print("Connection group updated successfully")
```

### Delete Connection Group

```python
client.connection_groups.delete("group_id")
print("Connection group deleted successfully")
```

---

## User Groups

Access user group management through `client.user_groups`.

### List All User Groups

```python
groups = client.user_groups.list()
for group_id, details in groups.items():
    print(f"Group {group_id}: {details['identifier']}")
    print(f"  Disabled: {details.get('disabled', False)}")
```

### Get User Group Details

```python
group = client.user_groups.details("group_id")
print(f"Group: {group['identifier']}")
print(f"Attributes: {group['attributes']}")
```

### Get Group Members

```python
members = client.user_groups.members("example_group")
for username, details in members.items():
    print(f"Member: {username}")
    print(f"  Full Name: {details['attributes'].get('guac-full-name', 'N/A')}")
```

### Add User to Group

```python
# Add user to group
payload = [{"op": "add", "path": "/", "value": "john_doe"}]
client.user_groups.edit_members("example_group", payload)
print("User added to group")
```

### Remove User from Group

```python
# Remove user from group
payload = [{"op": "remove", "path": "/", "value": "john_doe"}]
client.user_groups.edit_members("example_group", payload)
print("User removed from group")
```

### Create User Group

```python
from copy import deepcopy

group_payload = deepcopy(client.user_groups.GROUP_TEMPLATE)
group_payload.update({
    "identifier": "example_group"
})

new_group = client.user_groups.create(group_payload)
print(f"Created user group: {new_group}")
```

### Update User Group

```python
update_payload = deepcopy(client.user_groups.GROUP_TEMPLATE)
update_payload.update({
    "identifier": "example_group",
    "attributes": {"disabled": False}
})

client.user_groups.update("example_group", update_payload)
print("User group updated successfully")
```

### Delete User Group

```python
result = client.user_groups.delete("group_name")
if result:
    print("User group deleted successfully")
else:
    print("Failed to delete user group (known issue with some Guacamole versions)")
```

---

## Sharing Profiles

Access sharing profile management through `client.sharing_profiles`.

### List All Sharing Profiles

```python
profiles = client.sharing_profiles.list()
for profile_id, details in profiles.items():
    print(f"Profile {profile_id}: {details['name']}")
    print(f"  Connection: {details['primaryConnectionIdentifier']}")
```

### Get Sharing Profile Details

```python
profile = client.sharing_profiles.details("profile_id")
print(f"Name: {profile['name']}")
print(f"Primary Connection: {profile['primaryConnectionIdentifier']}")
```

### Get Sharing Profile Parameters

```python
params = client.sharing_profiles.parameters("profile_id")
print(f"Read-only: {params.get('read-only', 'N/A')}")
```

### Create Sharing Profile

```python
from copy import deepcopy

profile_payload = deepcopy(client.sharing_profiles.PROFILE_TEMPLATE)
profile_payload.update({
    "name": "Read-Only Access",
    "primaryConnectionIdentifier": "connection_id",
    "parameters": {
        "read-only": "true"
    }
})

new_profile = client.sharing_profiles.create(profile_payload)
print(f"Created sharing profile: {new_profile}")
```

### Update Sharing Profile

```python
update_payload = deepcopy(client.sharing_profiles.PROFILE_TEMPLATE)
update_payload.update({
    "name": "Updated Profile Name",
    "primaryConnectionIdentifier": "connection_id"
})

client.sharing_profiles.update("profile_id", update_payload)
print("Sharing profile updated successfully")
```

### Delete Sharing Profile

```python
client.sharing_profiles.delete("profile_id")
print("Sharing profile deleted successfully")
```

---

## Active Connections

Access active connection monitoring through `client.active_connections`.

### List Active Connections

```python
active = client.active_connections.list()
if not active:
    print("No active connections")
else:
    for conn_id, details in active.items():
        print(f"Active Connection {conn_id}:")
        print(f"  User: {details.get('username', 'Unknown')}")
        print(f"  Remote Host: {details.get('remoteHost', 'Unknown')}")
        print(f"  Start Date: {details.get('startDate', 'Unknown')}")
```

### Get Active Connection Details

```python
details = client.active_connections.details("connection_id")
if details:
    print(f"Connection: {details['identifier']}")
    print(f"User: {details.get('username', 'Unknown')}")
    print(f"Start Date: {details.get('startDate', 'Unknown')}")
else:
    print("Connection not active")
```

### Terminate Active Connection

```python
result = client.active_connections.kill("connection_id")
if result:
    print("Connection terminated successfully")
else:
    print("Connection not found or already terminated")
```

---

## Schema Information

Access schema information through `client.schema`.

### Get Supported Protocols

```python
protocols = client.schema.protocols()
for protocol_name, details in protocols.items():
    print(f"Protocol: {protocol_name}")
    print(f"  Name: {details.get('name', 'N/A')}")
    print(f"  Attributes: {details.get('attributes', {})}")
```

### Get User Attributes

```python
attributes = client.schema.user_attributes()
for attr_name, details in attributes.items():
    print(f"Attribute: {attr_name}")
    print(f"  Type: {details.get('type', 'N/A')}")
```

---

## Permissions Management

Access permission management through `client.permissions`.

### Assign System Permission

```python
client.permissions.assign_system_permission("john_doe", "CREATE_USER")
print("System permission assigned")

# Available system permissions:
# - CREATE_USER
# - CREATE_USER_GROUP
# - CREATE_CONNECTION
# - CREATE_CONNECTION_GROUP
# - CREATE_SHARING_PROFILE
# - ADMINISTER
```

### Revoke System Permission

```python
client.permissions.revoke_system_permission("john_doe", "CREATE_USER")
print("System permission revoked")
```

### Assign Connection Permission

```python
# Give user READ access to a connection
client.users.assign_connection("john_doe", "connection_id", "READ")
print("Connection permission assigned")

# Give user READ access to a connection group
client.users.assign_connection("john_doe", "group_id", "READ", is_connection_group=True)
print("Connection group permission assigned")
```

### Revoke Connection Permission

```python
# Revoke connection access
client.users.revoke_connection("john_doe", "connection_id", "READ")
print("Connection permission revoked")
```

### Assign User to Group

```python
client.users.assign_usergroups("john_doe", "example_group")
print("User added to group")
```

### Remove User from Group

```python
client.users.revoke_usergroups("john_doe", "example_group")
print("User removed from group")
```

---

## Common Workflows

### Complete User Setup

```python
from copy import deepcopy

# 1. Create user
user_payload = deepcopy(client.users.CREATE_USER_TEMPLATE)
user_payload.update({
    "username": "john_doe",
    "password": "super_secret_pass",
    "attributes": {
        "guac-full-name": "John Doe",
        "guac-email-address": "john.doe@example.com"
    }
})
user = client.users.create(user_payload)

# 2. Create connection
ssh_payload = deepcopy(client.connections.SSH_TEMPLATE)
ssh_payload.update({
    "name": "Example Connection",
    "parameters": {
        "hostname": "test_server",
        "port": "22",
        "username": "admin"
    }
})
connection = client.connections.create(ssh_payload)

# 3. Assign connection to user
client.users.assign_connection("john_doe", connection["identifier"], "READ")

# 4. Add user to group
client.users.assign_usergroups("john_doe", "example_group")

print("User setup complete!")
```

### Bulk User Creation

```python
from copy import deepcopy

users_to_create = [
    {"username": "john_doe", "full_name": "John Doe", "email": "john.doe@example.com"},
    {"username": "jane_smith", "full_name": "Jane Smith", "email": "jane.smith@example.com"},
    {"username": "bob_wilson", "full_name": "Bob Wilson", "email": "bob.wilson@example.com"}
]

for user_data in users_to_create:
    user_payload = deepcopy(client.users.CREATE_USER_TEMPLATE)
    user_payload.update({
        "username": user_data["username"],
        "password": "super_secret_pass",
        "attributes": {
            "guac-full-name": user_data["full_name"],
            "guac-email-address": user_data["email"]
        }
    })
    
    try:
        new_user = client.users.create(user_payload)
        print(f"Created user: {user_data['username']}")
    except Exception as e:
        print(f"Failed to create user {user_data['username']}: {e}")
```

### Connection Monitoring

```python
import time

def monitor_connections():
    active = client.active_connections.list()
    
    if not active:
        print("No active connections")
        return
    
    print(f"Active connections: {len(active)}")
    for conn_id, details in active.items():
        print(f"  {details.get('username', 'Unknown')} -> {details.get('remoteHost', 'Unknown')}")
    
    return active

# Monitor for 5 minutes
for i in range(5):
    print(f"\n--- Check {i+1} ---")
    active_connections = monitor_connections()
    time.sleep(60)
```

### Cleanup Inactive Users

```python
# Get all users
users = client.users.list()

# Check for users who haven't been active recently
inactive_users = []
for username, details in users.items():
    last_active = details.get('lastActive', 0)
    # Convert to days (assuming lastActive is in milliseconds)
    days_since_active = (time.time() * 1000 - last_active) / (1000 * 60 * 60 * 24)
    
    if days_since_active > 90:  # 90 days
        inactive_users.append(username)

print(f"Found {len(inactive_users)} inactive users: {inactive_users}")

# Optionally delete inactive users
# for username in inactive_users:
#     client.users.delete(username)
#     print(f"Deleted inactive user: {username}")
```

---

## Error Handling

### Common Exceptions

```python
from guacapy import Guacamole
import requests

try:
    client = Guacamole(
        hostname="guac.example.com",
        username="guacadmin",
        password="wrong_password",
        connection_protocol="https",
        ssl_verify=False,
        connection_port=8443
    )
except Exception as e:
    print(f"Authentication failed: {e}")

try:
    user = client.users.user_details("nonexistent_user")
except requests.HTTPError as e:
    if e.response.status_code == 404:
        print("User not found")
    elif e.response.status_code == 401:
        print("Authentication required")
    elif e.response.status_code == 403:
        print("Insufficient permissions")
    else:
        print(f"HTTP error: {e.response.status_code}")
```

### Handling Optional Returns

```python
# Some methods return None for non-critical failures
new_connection = client.connections.create(payload)
if new_connection is None:
    print("Connection already exists or creation failed")

# Active connection might not exist
active_details = client.active_connections.details("connection_id")
if active_details is None:
    print("Connection is not currently active")
```

### Known Issues

```python
# UserGroupManager.delete() may fail with 500 error
result = client.user_groups.delete("group_name")
if result is None:
    print("Failed to delete user group - known issue with some Guacamole versions")
    print("Manual SQL deletion may be required")
```

---

## Best Practices

### 1. Always Use Templates

```python
# Good - use templates
from copy import deepcopy
payload = deepcopy(client.users.CREATE_USER_TEMPLATE)
payload.update({"username": "new_user", "password": "password"})

# Bad - create payload from scratch
payload = {"username": "new_user", "password": "password"}
```

### 2. Handle Errors Gracefully

```python
def safe_create_user(username, password, email):
    try:
        user_payload = deepcopy(client.users.CREATE_USER_TEMPLATE)
        user_payload.update({
            "username": username,
            "password": password,
            "attributes": {"guac-email-address": email}
        })
        return client.users.create(user_payload)
    except Exception as e:
        print(f"Failed to create user {username}: {e}")
        return None
```

### 3. Use Appropriate Permissions

```python
# Give minimal required permissions
client.users.assign_connection("user", "connection_id", "READ")  # Read-only access

# For administrative users
client.permissions.assign_system_permission("admin_user", "ADMINISTER")
```

### 4. Monitor Active Connections

```python
# Check for long-running connections
active = client.active_connections.list()
for conn_id, details in active.items():
    start_time = details.get('startDate', 0)
    duration = (time.time() * 1000 - start_time) / (1000 * 60 * 60)  # hours
    
    if duration > 8:  # 8 hours
        print(f"Long-running connection: {conn_id} ({duration:.1f} hours)")
```

### 5. Clean Up Resources

```python
# Always logout when done
try:
    # Your code here
    pass
finally:
    client.logout()
```

This guide provides comprehensive coverage of all Guacapy functionality with practical examples for each method. The focus is on the end-user experience through the main `Guacamole` class and its manager properties.
