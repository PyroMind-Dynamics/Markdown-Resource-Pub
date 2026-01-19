# PyroMind API Documentation

Welcome to the PyroMind API documentation. This documentation provides comprehensive guides and instructions for using the PyroMind platform.

## Getting Started

To get started with PyroMind, you'll need:

- A PyroMind account with valid credentials
- Access to the PyroMind platform at [pyromind.ai](https://pyromind.ai)

## Documentation

### JupyterLab

Learn how to create and manage JupyterLab instances on the PyroMind platform:

- **[Create a JupyterLab and SSH to it](jupyterlab/how-to-use.md)** - Complete guide on creating a JupyterLab instance, adding SSH keys, and connecting to your instance via web interface or SSH.

## Quick Links

- [PyroMind Login](https://pyromind.ai/login.html)
- [JupyterLab Instance Management](https://pyromind.ai/jupyterlab-manage.html)

## Support

If you encounter any issues or have questions, please refer to the specific documentation sections above or contact PyroMind support.

# PyroMind AI API Offline Documentation
- Version: 1.0.0
- Specification: OAS 3.1
- Core Positioning: Reinforcement Learning as a Service Platform
- Document Scope: SandBoxes, Instance, Inference, Training Modules

## I. SandBoxes APIs
### Overview
SandBoxes APIs provide capabilities to manage isolated execution environments for development, testing, and simulation. Supports creating, querying, deleting sandboxes, and executing actions (including batch operations) and VNC connection management.

| Request Method | Endpoint Path | API Name | Description |
| :--- | :--- | :--- | :--- |
| GET | /api/v1/sandboxes | List Sandboxes Endpoint | List all sandboxes with optional filtering |
| POST | /api/v1/sandboxes | Create Sandbox Endpoint | Create a new isolated sandbox environment |
| GET | /api/v1/sandboxes/{sandbox_id} | Get Sandbox Endpoint | Retrieve details of a specified sandbox |
| DELETE | /api/v1/sandboxes/{sandbox_id} | Delete Sandbox Endpoint | Delete a specified sandbox |
| POST | /api/v1/sandboxes/{sandbox_id}/actions | Execute Action Endpoint | Execute a single action on the specified sandbox |
| GET | /api/v1/sandboxes/{sandbox_id}/vnc | Get Vnc Endpoint | Retrieve VNC connection information for remote access |
| POST | /api/v1/sandboxes/{sandbox_id}/actions/batch | Execute Batch Action Endpoint | Execute multiple actions on the specified sandbox in sequence or parallel |

### 1.1 GET /api/v1/sandboxes
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| page | Query | integer | No | Page number (default: 1) | `2` |
| page_size | Query | integer | No | Number of items per page (default: 20) | `50` |
| status | Query | string | No | Sandbox status filter (allowed values: running/stopped/error) | `running` |
| sandbox_type | Query | string | No | Sandbox type filter (e.g., gpu-sandbox/cpu-sandbox) | `gpu-sandbox` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Sandbox list retrieved successfully | `{"code": 200, "message": "sandboxes listed successfully", "data": {"total": 128, "page": 2, "page_size": 50, "sandboxes": [{"sandbox_id": "sbx_123456789", "name": "GPU-Sandbox-001", "status": "running", "sandbox_type": "gpu-sandbox", "create_time": "2024-01-01T00:00:00Z", "expire_time": "2024-01-08T00:00:00Z", "resource_config": {"cpu": 8, "memory": 32, "gpu": 2}}]}}` |
| 400 | object | Invalid parameter | `{"code": 400, "message": "invalid status value (allowed: running/stopped/error)", "data": null}` |

#### Usage Notes
- Pagination parameters (page/page_size) are recommended for large result sets to avoid performance issues.
- Status and sandbox_type filters can be used together to narrow down results.

### 1.2 POST /api/v1/sandboxes
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| name | Body | string | Yes | Sandbox name (3-64 characters, alphanumeric/hyphens/underscores) | `My-GPU-Sandbox-2024` |
| sandbox_type | Body | string | Yes | Sandbox type (must match platform-supported types) | `gpu-sandbox` |
| resource_config | Body | object | Yes | Resource configuration (cpu/memory/gpu) | `{"cpu": 16, "memory": 64, "gpu": 4}` |
| screen_resolution | Body | object | No | Screen resolution for VNC (default: {"width": 1920, "height": 1080}) | `{"width": 2560, "height": 1440}` |
| expire_time | Body | string | No | Expiration time (ISO 8601 format, default: 7 days from creation) | `2024-01-15T00:00:00Z` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 201 | object | Sandbox created successfully | `{"code": 201, "message": "sandbox created successfully", "data": {"sandbox_id": "sbx_987654321", "name": "My-GPU-Sandbox-2024", "status": "pending", "sandbox_type": "gpu-sandbox", "resource_config": {"cpu": 16, "memory": 64, "gpu": 4}, "create_time": "2024-01-01T00:00:00Z", "expire_time": "2024-01-15T00:00:00Z"}}` |
| 400 | object | Invalid parameter | `{"code": 400, "message": "resource config cpu must be positive integer", "data": null}` |
| 409 | object | Sandbox name already exists | `{"code": 409, "message": "sandbox name already exists in your namespace", "data": null}` |
| 500 | object | Failed to create sandbox | `{"code": 500, "message": "failed to create sandbox due to resource shortage", "data": null}` |

#### Usage Notes
- Sandbox names are unique within the user's namespace.
- GPU resources are limited; ensure the requested GPU count is within your quota.
- Expire_time must be a future time; expired sandboxes are automatically cleaned up.

### 1.3 GET /api/v1/sandboxes/{sandbox_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| sandbox_id | Path | string | Yes | Unique sandbox identification ID (generated on creation) | `sbx_987654321` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Sandbox details retrieved successfully | `{"code": 200, "message": "sandbox details retrieved successfully", "data": {"sandbox_id": "sbx_987654321", "name": "My-GPU-Sandbox-2024", "status": "running", "sandbox_type": "gpu-sandbox", "resource_config": {"cpu": 16, "memory": 64, "gpu": 4}, "screen_resolution": {"width": 2560, "height": 1440}, "create_time": "2024-01-01T00:00:00Z", "expire_time": "2024-01-15T00:00:00Z", "vnc_info": {"host": "vnc.pyromind.ai", "port": 5901, "password": "temp_123456"}}}` |
| 404 | object | Sandbox not found | `{"code": 404, "message": "sandbox not found (check sandbox_id or expired)", "data": null}` |

#### Usage Notes
- Use the sandbox_id returned from the create API to query details.
- VNC info is only available when the sandbox status is "running".

### 1.4 DELETE /api/v1/sandboxes/{sandbox_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| sandbox_id | Path | string | Yes | Unique sandbox identification ID | `sbx_987654321` |
| force | Query | boolean | No | Whether to force delete (ignores running tasks, default: false) | `true` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Sandbox deleted successfully | `{"code": 200, "message": "sandbox deleted successfully", "data": {"sandbox_id": "sbx_987654321", "delete_time": "2024-01-05T00:00:00Z", "force": true}}` |
| 404 | object | Sandbox not found | `{"code": 404, "message": "sandbox not found", "data": null}` |
| 409 | object | Sandbox has running tasks (non-force delete) | `{"code": 409, "message": "sandbox has running tasks, use force=true to delete", "data": null}` |

#### Usage Notes
- Non-force delete will fail if the sandbox has active tasks; use force=true for immediate deletion.
- Deleted sandboxes cannot be recovered; back up data before deletion.

### 1.5 POST /api/v1/sandboxes/{sandbox_id}/actions
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| sandbox_id | Path | string | Yes | Unique sandbox identification ID | `sbx_987654321` |
| action | Body | string | Yes | Action type (allowed: start/stop/restart/reboot/install_package/run_script) | `restart` |
| timeout | Body | integer | No | Action timeout (seconds, default: 30) | `60` |
| params | Body | object | No | Action-specific parameters (required for install_package/run_script) | `{"name": "python3-pip"}` (for install_package) |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Action executed successfully | `{"code": 200, "message": "sandbox action executed successfully", "data": {"sandbox_id": "sbx_987654321", "action": "restart", "status": "completed", "execute_time": "2024-01-05T00:00:00Z", "timeout": 60}}` |
| 400 | object | Invalid action type or parameters | `{"code": 400, "message": "invalid action type (allowed: start/stop/restart/reboot/install_package/run_script)", "data": null}` |
| 404 | object | Sandbox not found | `{"code": 404, "message": "sandbox not found", "data": null}` |
| 500 | object | Failed to execute action | `{"code": 500, "message": "failed to execute action: sandbox unresponsive", "data": null}` |

#### Usage Notes
- Different actions require different params:
  - install_package: {"name": "package_name"}
  - run_script: {"path": "/path/to/script.sh", "args": ["arg1", "arg2"]}
- Action execution status can be checked via the Get Sandbox API.

### 1.6 GET /api/v1/sandboxes/{sandbox_id}/vnc
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| sandbox_id | Path | string | Yes | Unique sandbox identification ID | `sbx_987654321` |
| refresh | Query | boolean | No | Whether to refresh VNC password (default: false) | `true` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | VNC information retrieved successfully | `{"code": 200, "message": "vnc connection info retrieved successfully", "data": {"sandbox_id": "sbx_987654321", "vnc_host": "vnc.pyromind.ai", "vnc_port": 5901, "vnc_password": "new_temp_789", "expire_time": "2024-01-06T00:00:00Z", "refresh_time": "2024-01-05T00:00:00Z"}}` |
| 404 | object | Sandbox not found or VNC not enabled | `{"code": 404, "message": "sandbox not found or vnc not enabled", "data": null}` |
| 500 | object | Failed to retrieve VNC information | `{"code": 500, "message": "failed to retrieve vnc info", "data": null}` |

#### Usage Notes
- VNC password is temporary and expires after the specified time; refresh with refresh=true if expired.
- Use a VNC client (e.g., RealVNC, TightVNC) to connect with the provided host, port, and password.

### 1.7 POST /api/v1/sandboxes/{sandbox_id}/actions/batch
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| sandbox_id | Path | string | Yes | Unique sandbox identification ID | `sbx_987654321` |
| actions | Body | array | Yes | Batch action list (each element contains action and params) | `[{"action": "install_package", "params": {"name": "python3-pip"}}, {"action": "run_script", "params": {"path": "/home/user/init.sh"}}]` |
| parallel | Body | boolean | No | Whether to execute in parallel (default: false) | `false` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Batch actions executed successfully | `{"code": 200, "message": "batch actions executed successfully", "data": {"sandbox_id": "sbx_987654321", "total_actions": 2, "success_count": 2, "failed_count": 0, "results": [{"action": "install_package", "status": "success", "message": "package installed"}, {"action": "run_script", "status": "success", "message": "script executed successfully"}]}}` |
| 400 | object | Invalid action list format | `{"code": 400, "message": "invalid actions format: each action must have 'action' field", "data": null}` |
| 404 | object | Sandbox not found | `{"code": 404, "message": "sandbox not found", "data": null}` |
| 500 | object | Partial actions failed | `{"code": 500, "message": "some batch actions failed", "data": {"sandbox_id": "sbx_987654321", "total_actions": 2, "success_count": 1, "failed_count": 1, "results": [{"action": "install_package", "status": "success", "message": "package installed"}, {"action": "run_script", "status": "failed", "message": "script file not found"}]}}` |

#### Usage Notes
- Parallel execution (parallel=true) is faster but may consume more resources; use for independent actions.
- Failed actions do not stop the entire batch; check the results array for individual action status.

## II. Instance APIs
### Overview
Instance APIs enable management of interactive computing environments for data science and machine learning workflows. Supports creating, querying, editing, pausing, resuming, and deleting instances, with configurable resources and workspace mounting.

| Request Method | Endpoint Path | API Name | Description |
| :--- | :--- | :--- | :--- |
| GET | /api/v1/instances | List Instance Endpoint | List all instances with optional filtering |
| POST | /api/v1/instances | Create Instance Endpoint | Create a new instance |
| GET | /api/v1/instances/{instance_id} | Get Instance Endpoint | Retrieve details of a specified instance |
| DELETE | /api/v1/instances/{instance_id} | Delete Instance Endpoint | Delete a specified instance |
| PUT | /api/v1/instances/{instance_id} | Edit Instance Endpoint | Update configuration of a specified instance |
| POST | /api/v1/instances/{instance_id}/pause | Pause Instance Endpoint | Pause a running instance to save resources |
| POST | /api/v1/instances/{instance_id}/resume | Resume Instance Endpoint | Resume a paused instance |
| GET | /api/v1/instances/{instance_id}/access-token | Get Access Token Endpoint | Retrieve temporary access token for instance UI |

### 2.1 GET /api/v1/instances
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| page | Query | integer | No | Page number (default: 1) | `1` |
| page_size | Query | integer | No | Number of items per page (default: 20) | `10` |
| status | Query | string | No | Instance status filter (running/paused/stopped/error) | `running` |
| resource_type | Query | string | No | Resource type filter (cpu/gpu) | `gpu` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Instance list retrieved successfully | `{"code": 200, "message": "instances listed successfully", "data": {"total": 35, "page": 1, "page_size": 10, "instances": [{"instance_id": "jp_100001", "name": "GPU-Instance-2024", "status": "running", "resource_type": "gpu", "resource_config": {"cpu": 12, "memory": 48, "gpu": 2}, "create_time": "2024-01-02T09:30:00Z", "expire_time": "2024-01-16T09:30:00Z"}, {"instance_id": "jp_100002", "name": "CPU-Instance-Basic", "status": "paused", "resource_type": "cpu", "resource_config": {"cpu": 8, "memory": 16, "gpu": 0}, "create_time": "2024-01-03T14:15:00Z", "expire_time": "2024-01-17T14:15:00Z"}]}}` |
| 400 | object | Invalid parameter | `{"code": 400, "message": "invalid status value (allowed: running/paused/stopped/error)", "data": null}` |

### 2.2 POST /api/v1/instances
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| name | Body | string | Yes | Instance name | `RL-Experiment-Instance` |
| resource_type | Body | string | Yes | Resource type (cpu/gpu) | `gpu` |
| resource_config | Body | object | Yes | Resource configuration (cpu/memory/gpu) | `{"cpu": 16, "memory": 64, "gpu": 4}` |
| image | Body | string | No | Image version (default: latest) | `pyromind-instance:2.3.1` |
| expire_time | Body | string | No | Expiration time (ISO format, default 7 days) | `2024-01-20T18:00:00Z` |
| password | Body | string | No | Access password (automatically generated by default) | `SecurePass123!` |
| workspace_mount | Body | object | No | Workspace mount configuration | `{"bucket_name": "user-123456789-data", "mount_path": "/home/jovyan/work"}` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 201 | object | Instance created successfully | `{"code": 201, "message": "instance created successfully", "data": {"instance_id": "jp_100003", "name": "RL-Experiment-Instance", "status": "pending", "resource_type": "gpu", "resource_config": {"cpu": 16, "memory": 64, "gpu": 4}, "image": "pyromind-instance:2.3.1", "create_time": "2024-01-05T10:00:00Z", "expire_time": "2024-01-20T18:00:00Z", "access_url": "https://instances.pyromind.ai/jp_100003", "password": "SecurePass123!"}}` |
| 400 | object | Invalid parameter | `{"code": 400, "message": "resource_config.gpu must be non-negative integer", "data": null}` |
| 409 | object | Instance name already exists | `{"code": 409, "message": "instance name already exists", "data": null}` |
| 500 | object | Failed to create instance | `{"code": 500, "message": "failed to create instance: insufficient GPU resources", "data": null}` |

### 2.3 GET /api/v1/instances/{instance_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| instance_id | Path | string | Yes | Unique instance identification ID | `jp_100003` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Instance details retrieved successfully | `{"code": 200, "message": "instance details retrieved successfully", "data": {"instance_id": "jp_100003", "name": "RL-Experiment-Instance", "status": "running", "resource_type": "gpu", "resource_config": {"cpu": 16, "memory": 64, "gpu": 4}, "image": "pyromind-instance:2.3.1", "create_time": "2024-01-05T10:00:00Z", "expire_time": "2024-01-20T18:00:00Z", "access_url": "https://instances.pyromind.ai/jp_100003", "workspace_mount": {"bucket_name": "user-123456789-data", "mount_path": "/home/jovyan/work", "status": "bound"}, "resource_usage": {"cpu_used": 8.2, "memory_used": 32.5, "gpu_used": 2.1}, "logs": "2024-01-05T10:05:00Z - Instance started successfully"}}` |
| 404 | object | Instance not found | `{"code": 404, "message": "instance not found", "data": null}` |

### 2.4 DELETE /api/v1/instances/{instance_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| instance_id | Path | string | Yes | Unique instance identification ID | `jp_100003` |
| backup_workspace | Query | boolean | No | Whether to back up workspace (default: false) | `true` |
| backup_bucket | Query | string | No | Backup target bucket (required if backup_workspace=true) | `user-123456789-backup` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Instance deleted successfully | `{"code": 200, "message": "instance deleted successfully", "data": {"instance_id": "jp_100003", "delete_time": "2024-01-10T15:30:00Z", "backup_workspace": true, "backup_bucket": "user-123456789-backup", "backup_path": "/instance-backups/jp_100003_20240110"}}` |
| 404 | object | Instance not found | `{"code": 404, "message": "instance not found", "data": null}` |
| 409 | object | Instance has running tasks (not force deleted) | `{"code": 409, "message": "instance has running tasks, stop first or use force=true", "data": null}` |
| 500 | object | Failed to back up workspace | `{"code": 500, "message": "failed to backup workspace: bucket access denied", "data": null}` |

### 2.5 PUT /api/v1/instances/{instance_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| instance_id | Path | string | Yes | Unique instance identification ID | `jp_100003` |
| name | Body | string | No | New instance name (leave blank if not modified) | `RL-Experiment-Instance-V2` |
| resource_config | Body | object | No | New resource configuration (leave blank if not modified) | `{"cpu": 20, "memory": 80, "gpu": 4}` |
| expire_time | Body | string | No | New expiration time (ISO format, leave blank if not modified) | `2024-01-25T18:00:00Z` |
| password | Body | string | No | New access password (leave blank if not modified) | `NewSecurePass456!` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Instance edited successfully | `{"code": 200, "message": "instance updated successfully", "data": {"instance_id": "jp_100003", "name": "RL-Experiment-Instance-V2", "updated_fields": ["name", "resource_config", "expire_time", "password"], "resource_config": {"cpu": 20, "memory": 80, "gpu": 4}, "expire_time": "2024-01-25T18:00:00Z", "update_time": "2024-01-06T11:20:00Z"}}` |
| 400 | object | Invalid parameter | `{"code": 400, "message": "resource_config.cpu must be greater than current 16", "data": null}` |
| 404 | object | Instance not found | `{"code": 404, "message": "instance not found", "data": null}` |
| 409 | object | New name already exists | `{"code": 409, "message": "new instance name already exists", "data": null}` |
| 500 | object | Failed to update instance | `{"code": 500, "message": "failed to update resource config: insufficient resources", "data": null}` |

### 2.6 POST /api/v1/instances/{instance_id}/pause
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| instance_id | Path | string | Yes | Unique instance ID | `jp_87654321` |
| save_checkpoint | Query | boolean | No | Save notebook checkpoints before pausing (default: true) | `true` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Instance paused successfully | `{"code": 200, "message": "instance paused successfully", "data": {"instance_id": "jp_87654321", "status": "paused", "pause_time": "2024-01-02T10:00:00Z", "save_checkpoint": true, "checkpoint_count": 3}}` |
| 400 | object | Invalid status (not running) | `{"code": 400, "message": "cannot pause instance in 'paused' or 'stopped' status", "data": null}` |
| 404 | object | Instance not found | `{"code": 404, "message": "instance not found", "data": null}` |
| 500 | object | Failed to pause instance | `{"code": 500, "message": "failed to save checkpoints: storage error", "data": null}` |

#### Usage Notes
- Paused instances do not consume compute resources but retain storage data.
- Checkpoints are saved to the mounted workspace by default.

### 2.7 POST /api/v1/instances/{instance_id}/resume
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| instance_id | Path | string | Yes | Unique instance ID | `jp_87654321` |
| restore_checkpoint | Query | boolean | No | Restore from latest checkpoint (default: false) | `false` |
| resource_config | Body | object | No | Override resource config (cpu/memory/gpu) | `{"cpu": 8, "memory": 32, "gpu": 1}` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Instance resumed successfully | `{"code": 200, "message": "instance resumed successfully", "data": {"instance_id": "jp_87654321", "status": "running", "resume_time": "2024-01-02T14:00:00Z", "restore_checkpoint": false, "resource_config": {"cpu": 8, "memory": 32, "gpu": 1}, "access_url": "https://instances.pyromind.ai/jp_87654321"}}` |
| 400 | object | Invalid status (not paused) | `{"code": 400, "message": "cannot resume instance in 'running' or 'stopped' status", "data": null}` |
| 404 | object | Instance not found | `{"code": 404, "message": "instance not found", "data": null}` |
| 500 | object | Failed to resume instance | `{"code": 500, "message": "failed to resume: insufficient GPU resources", "data": null}` |

#### Usage Notes
- Resource config override is limited by user quota; cannot exceed original request specs.
- Access URL is regenerated after resume; use the new URL for access.

### 2.8 GET /api/v1/instances/{instance_id}/access-token
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| instance_id | Path | string | Yes | Unique instance ID | `jp_87654321` |
| token_ttl | Query | integer | No | Token time-to-live (seconds, default: 3600) | `7200` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Access token retrieved successfully | `{"code": 200, "message": "access token generated successfully", "data": {"instance_id": "jp_87654321", "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...", "token_ttl": 7200, "expire_time": "2024-01-02T16:00:00Z", "access_url": "https://instances.pyromind.ai/jp_87654321?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}}` |
| 400 | object | Instance not running | `{"code": 400, "message": "cannot generate token for non-running instance", "data": null}` |
| 404 | object | Instance not found | `{"code": 404, "message": "instance not found", "data": null}` |
| 500 | object | Failed to generate token | `{"code": 500, "message": "token generation failed: auth service error", "data": null}` |

#### Usage Notes
- Tokens are temporary and expire after `token_ttl`; regenerate when expired.
- Do not share access tokens publicly; they grant full access to the instance.

## III. Inference APIs
### Overview
Inference APIs provide capabilities to deploy and manage model inference tasks, supporting batch inference with configurable resources, input/output data sources, and task prioritization.

| Request Method | Endpoint Path | API Name | Description |
| :--- | :--- | :--- | :--- |
| POST | /api/v1/inference | Create Inference Endpoint | Submit a new batch inference task |
| GET | /api/v1/inference | List Inference Tasks Endpoint | List all inference tasks with optional filtering |
| GET | /api/v1/inference/{job_id} | Get Inference Task Endpoint | Retrieve details of a specified inference task |
| DELETE | /api/v1/inference/{job_id} | Delete Inference Task Endpoint | Delete a specified inference task |

### 3.1 POST /api/v1/inference
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| name | Body | string | Yes | Inference task name | `RL-Traffic-Control-Inference` |
| model_type | Body | string | Yes | Model type (rl/nlp/cv) | `rl` |
| model_name | Body | string | Yes | Model name (must be uploaded to the platform) | `ppo-traffic-control-v3` |
| model_version | Body | string | No | Model version (default: latest) | `v3.2` |
| resource_config | Body | object | Yes | Resource configuration (cpu/memory/gpu) | `{"gpu": 2, "cpu": 8, "memory": 32}` |
| input_config | Body | object | Yes | Input data config (source/format/batch_size) | `{"data_source": "s3://user-data/inference-inputs/traffic-2024.csv", "data_format": "csv", "batch_size": 64}` |
| output_config | Body | object | Yes | Output data config (bucket/path/format) | `{"output_bucket": "user-data", "output_path": "/inference-outputs/traffic-result", "output_format": "parquet"}` |
| inference_params | Body | object | No | Model-specific inference params | `{"confidence_threshold": 0.8, "max_steps": 1000}` |
| priority | Body | integer | No | Task priority (1-10, default: 5) | `8` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 201 | object | Inference task created successfully | `{"code": 201, "message": "inference task created successfully", "data": {"job_id": "inf_200003", "name": "RL-Traffic-Control-Inference", "status": "pending", "model_type": "rl", "model_name": "ppo-traffic-control-v3", "create_time": "2024-01-05T13:00:00Z", "priority": 8, "task_url": "https://pyromind.ai/tasks/inf_200003"}}` |
| 400 | object | Invalid parameter | `{"code": 400, "message": "input_config.data_source is required", "data": null}` |
| 404 | object | Model not found | `{"code": 404, "message": "model ppo-traffic-control-v3 (v3.2) not found", "data": null}` |
| 409 | object | Task name already exists | `{"code": 409, "message": "inference task name already exists", "data": null}` |
| 500 | object | Task creation failed | `{"code": 500, "message": "failed to create task: invalid output bucket permission", "data": null}` |

#### Usage Notes
- Input data sources support S3, HDFS, and local platform storage; ensure read permissions.
- Higher priority tasks are scheduled first; priority 10 is the highest.

### 3.2 GET /api/v1/inference
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| page | Query | integer | No | Page number (default: 1) | `2` |
| page_size | Query | integer | No | Number of items per page (default: 20) | `25` |
| status | Query | string | No | Task status filter (pending/running/success/failed) | `running` |
| model_type | Query | string | No | Model type filter (rl/nlp/cv) | `rl` |
| start_time | Query | string | No | Start time filter (ISO format) | `2024-01-01T00:00:00Z` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Inference tasks listed successfully | `{"code": 200, "message": "inference tasks listed successfully", "data": {"total": 45, "page": 2, "page_size": 25, "jobs": [{"job_id": "inf_200003", "name": "RL-Traffic-Control-Inference", "status": "running", "progress": 65, "create_time": "2024-01-05T13:00:00Z"}]}}` |
| 400 | object | Invalid filter parameter | `{"code": 400, "message": "invalid status value (allowed: pending/running/success/failed)", "data": null}` |

#### Usage Notes
- Combine multiple filters to narrow down results (e.g., status + model_type).
- Large result sets should use pagination to avoid performance issues.

### 3.3 GET /api/v1/inference/{job_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| job_id | Path | string | Yes | Unique inference task ID | `inf_200003` |
| include_logs | Query | boolean | No | Include task logs in response (default: false) | `true` |
| log_lines | Query | integer | No | Number of log lines to return (default: 100) | `200` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Inference task details retrieved successfully | `{"code": 200, "message": "inference task details retrieved successfully", "data": {"job_id": "inf_200003", "name": "RL-Traffic-Control-Inference", "status": "running", "progress": 65, "resource_usage": {"gpu_used": 1.8, "cpu_used": 6.5, "memory_used": 28.3}, "input_config": {...}, "output_config": {...}, "logs": ["2024-01-05T13:05:00Z - Inference started", "2024-01-05T13:10:00Z - Processed 16000 rows"]}}` |
| 404 | object | Inference task not found | `{"code": 404, "message": "inference task not found", "data": null}` |

#### Usage Notes
- Logs are useful for debugging failed or slow tasks; limit `log_lines` to avoid large response sizes.
- Resource usage data is updated in real time; check periodically for resource bottlenecks.

### 3.4 DELETE /api/v1/inference/{job_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| job_id | Path | string | Yes | Unique inference task ID | `inf_200003` |
| delete_output | Query | boolean | No | Delete output results (default: false) | `false` |
| force | Query | boolean | No | Force delete running tasks (default: false) | `true` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Inference task deleted successfully | `{"code": 200, "message": "inference task deleted successfully", "data": {"job_id": "inf_200003", "delete_time": "2024-01-05T13:18:00Z", "delete_output": false, "status_before_delete": "running"}}` |
| 404 | object | Inference task not found | `{"code": 404, "message": "inference task not found", "data": null}` |
| 409 | object | Task running (non-force delete) | `{"code": 409, "message": "inference task is running, use force=true to delete", "data": null}` |
| 500 | object | Delete failed | `{"code": 500, "message": "failed to delete output: bucket access denied", "data": null}` |

#### Usage Notes
- Deleting a task does not delete output data by default; use `delete_output=true` to clean up storage.
- Force deletion will terminate running tasks immediately; data processed up to that point is retained.

## IV. Training APIs
### Overview
Training APIs enable end-to-end management of model training tasks, supporting hyperparameter configuration, resource scaling, checkpoint management, and task pause/resume.

| Request Method | Endpoint Path | API Name | Description |
| :--- | :--- | :--- | :--- |
| POST | /api/v1/training/jobs | Submit Training Job Endpoint | Submit a new model training task |
| GET | /api/v1/training/jobs | List Training Jobs Endpoint | List all training tasks with optional filtering |
| GET | /api/v1/training/jobs/{job_id} | Get Training Job Endpoint | Retrieve details of a specified training task |
| DELETE | /api/v1/training/jobs/{job_id} | Delete Training Job Endpoint | Delete a specified training task |
| POST | /api/v1/training/jobs/{job_id}/pause | Pause Training Job Endpoint | Pause a running training task |
| POST | /api/v1/training/jobs/{job_id}/resume | Resume Training Job Endpoint | Resume a paused training task |

### 4.1 POST /api/v1/training/jobs
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| name | Body | string | Yes | Training task name | `RL-PPO-LunarLander-Training` |
| training_framework | Body | string | Yes | Training framework (pytorch/tensorflow/ray) | `pytorch` |
| model_name | Body | string | Yes | Model name (custom or pre-defined) | `ppo-lunar-lander-v2` |
| resource_config | Body | object | Yes | Resource configuration (cpu/memory/gpu) | `{"gpu": 4, "cpu": 16, "memory": 64}` |
| data_config | Body | object | Yes | Data config (train/val source/format) | `{"train_data_source": "s3://user-data/train-data", "val_data_source": "s3://user-data/val-data", "data_format": "tfrecord"}` |
| hyperparameters | Body | object | Yes | Training hyperparameters | `{"learning_rate": 0.0003, "batch_size": 256, "epochs": 100, "gamma": 0.99}` |
| output_config | Body | object | Yes | Output config (model storage/save frequency) | `{"output_bucket": "user-models", "output_path": "/rl-models/ppo-lunar", "save_frequency": 10}` |
| log_config | Body | object | No | Log config (tool/path) | `{"log_tool": "tensorboard", "log_path": "s3://user-logs/ppo-training-2024"}` |
| priority | Body | integer | No | Task priority (1-10, default: 5) | `9` |
| max_run_time | Body | string | No | Max run time (ISO format, default: 72h) | `PT48H` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 201 | object | Training job submitted successfully | `{"code": 201, "message": "training job submitted successfully", "data": {"job_id": "train_300001", "name": "RL-PPO-LunarLander-Training", "status": "pending", "create_time": "2024-01-06T09:00:00Z", "task_url": "https://pyromind.ai/training/jobs/train_300001"}}` |
| 400 | object | Invalid parameter | `{"code": 400, "message": "hyperparameters.learning_rate must be positive", "data": null}` |
| 409 | object | Task name already exists | `{"code": 409, "message": "training job name already exists", "data": null}` |
| 500 | object | Submit failed | `{"code": 500, "message": "failed to submit job: insufficient GPU resources", "data": null}` |

#### Usage Notes
- Save frequency defines how often checkpoints are saved (per epoch); higher frequency uses more storage.
- Max run time prevents tasks from running indefinitely; tasks are terminated automatically after expiration.

### 4.2 GET /api/v1/training/jobs
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| page | Query | integer | No | Page number (default: 1) | `1` |
| page_size | Query | integer | No | Items per page (default: 20) | `25` |
| status | Query | string | No | Status filter (pending/running/success/failed/canceled/paused) | `running` |
| training_framework | Query | string | No | Framework filter (pytorch/tensorflow/ray) | `pytorch` |
| start_time | Query | string | No | Start time filter (ISO format) | `2024-01-01T00:00:00Z` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Training jobs listed successfully | `{"code": 200, "message": "training jobs listed successfully", "data": {"total": 89, "page": 1, "page_size": 25, "jobs": [{"job_id": "train_300001", "name": "RL-PPO-LunarLander-Training", "status": "running", "progress": 42, "create_time": "2024-01-06T09:00:00Z"}]}}` |
| 400 | object | Invalid filter | `{"code": 400, "message": "invalid status value (allowed: pending/running/success/failed/canceled/paused)", "data": null}` |

#### Usage Notes
- Filter by `training_framework` to group tasks by the ML framework used.
- Progress percentage is calculated based on completed epochs vs. total epochs.

### 4.3 GET /api/v1/training/jobs/{job_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| job_id | Path | string | Yes | Unique training job ID | `train_300001` |
| include_logs | Query | boolean | No | Include task logs (default: false) | `true` |
| log_lines | Query | integer | No | Number of log lines (default: 100) | `300` |
| include_metrics | Query | boolean | No | Include training metrics (default: false) | `true` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Training job details retrieved successfully | `{"code": 200, "message": "training job details retrieved successfully", "data": {"job_id": "train_300001", "name": "RL-PPO-LunarLander-Training", "status": "running", "progress": 58, "resource_usage": {"gpu_used": 3.8, "cpu_used": 12.4}, "hyperparameters": {...}, "metrics": {"train_loss": 0.087, "val_loss": 0.123, "reward_mean": 289.4}, "logs": [...]}}` |
| 404 | object | Training job not found | `{"code": 404, "message": "training job not found", "data": null}` |

#### Usage Notes
- Training metrics (loss, reward) are updated per epoch; use these to evaluate model performance.
- Logs include framework-specific logs and error messages; critical for debugging training failures.

### 4.4 DELETE /api/v1/training/jobs/{job_id}
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| job_id | Path | string | Yes | Unique training job ID | `train_300001` |
| delete_output | Query | boolean | No | Delete trained models/checkpoints (default: false) | `false` |
| delete_logs | Query | boolean | No | Delete training logs (default: false) | `false` |
| force | Query | boolean | No | Force delete running tasks (default: false) | `true` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Training job deleted successfully | `{"code": 200, "message": "training job deleted successfully", "data": {"job_id": "train_300001", "delete_time": "2024-01-06T20:00:00Z", "delete_output": false, "status_before_delete": "running"}}` |
| 404 | object | Training job not found | `{"code": 404, "message": "training job not found", "data": null}` |
| 409 | object | Task running (non-force delete) | `{"code": 409, "message": "training job is running, use force=true to delete", "data": null}` |
| 500 | object | Delete failed | `{"code": 500, "message": "failed to delete logs: bucket access denied", "data": null}` |

#### Usage Notes
- Deleting logs and checkpoints frees up storage but removes debugging and resume capabilities.
- Force deletion terminates running tasks immediately; partial checkpoints may be corrupted.

### 4.5 POST /api/v1/training/jobs/{job_id}/pause
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| job_id | Path | string | Yes | Unique training job ID | `train_300001` |
| save_checkpoint | Query | boolean | No | Save checkpoint before pausing (default: true) | `true` |
| checkpoint_name | Query | string | No | Custom checkpoint name (auto-generated if empty) | `pause-checkpoint-epoch-58` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Training job paused successfully | `{"code": 200, "message": "training job paused successfully", "data": {"job_id": "train_300001", "status": "paused", "pause_time": "2024-01-06T20:30:00Z", "checkpoint_name": "pause-checkpoint-epoch-58", "current_epoch": 58}}` |
| 400 | object | Invalid status | `{"code": 400, "message": "cannot pause job in 'completed' status", "data": null}` |
| 404 | object | Training job not found | `{"code": 404, "message": "training job not found", "data": null}` |
| 500 | object | Pause failed | `{"code": 500, "message": "failed to save checkpoint: storage write error", "data": null}` |

#### Usage Notes
- Custom checkpoint names make it easier to identify pause points for later resume.
- Paused jobs do not consume compute resources; resume when resources are available.

### 4.6 POST /api/v1/training/jobs/{job_id}/resume
#### Parameters
| Parameter Name | Location | Type | Required | Description | Example Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| job_id | Path | string | Yes | Unique training job ID | `train_300001` |
| resume_from_checkpoint | Query | string | No | Checkpoint to resume from (latest if empty) | `pause-checkpoint-epoch-58` |
| reset_learning_rate | Query | boolean | No | Reset learning rate (default: false) | `false` |
| new_learning_rate | Query | number | No | New learning rate (only if reset_learning_rate=true) | `0.0002` |

#### Responses
| Status Code | Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| 200 | object | Training job resumed successfully | `{"code": 200, "message": "training job resumed successfully", "data": {"job_id": "train_300001", "status": "running", "resume_time": "2024-01-07T09:00:00Z", "resumed_from_checkpoint": "pause-checkpoint-epoch-58", "current_learning_rate": 0.0003, "estimated_finish_time": "2024-01-07T15:00:00Z"}}` |
| 400 | object | Invalid status or parameter | `{"code": 400, "message": "cannot resume running job / new_learning_rate must be positive", "data": null}` |
| 404 | object | Job or checkpoint not found | `{"code": 404, "message": "training job or checkpoint not found", "data": null}` |
| 500 | object | Resume failed | `{"code": 500, "message": "failed to load checkpoint: corrupted file", "data": null}` |

#### Usage Notes
- Resetting the learning rate can help with model convergence if training stalls.
- Ensure the specified checkpoint exists in the output bucket before resume.

---

### Document Appendix
1. **Status Glossary**
   - `pending`: Task is waiting for resource allocation
   - `running`: Task is executing normally
   - `success`: Task completed without errors
   - `failed`: Task terminated due to errors
   - `canceled`: Task was manually canceled
   - `paused`: Task was paused and can be resumed

2. **Resource Quota Notes**
   - All resource requests (CPU/GPU/memory) are subject to user quota limits
   - GPU resources are shared across tasks; peak usage may affect performance

3. **Storage Compatibility**
   - Supported storage services: AWS S3, HDFS, PyroMind Platform Storage
   - Supported data formats: CSV, Parquet, TFRecord, JSON, NumPy
