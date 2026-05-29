# SSH Combined-Username Login for JupyterLab Instances

**English** | **[中文](ssh-combined-name-login-cn.md)**

## Background

When multiple users share the same JupyterLab instance, the system defaults to logging in as `root`. If everyone operates as `root`, their environments may interfere with one another. This guide explains how to configure separate system users for a single instance and log in via SSH with distinct usernames.

## Prerequisites

- JupyterLab image version **v0.0.6** or later (supports the startup callback script `start-hook.sh`)
- Storage mounted with read/write access to `/workspace`

## Configuration

### Step 1: Set up the multi-user initialization script

When a JupyterLab instance starts, it runs the callback script `start-hook.sh`. You can use this script to create system users and bind each user to the platform-assigned SSH public key.

| Item | Details |
|------|---------|
| Callback script | `start-hook.sh` (if it does not exist, log in as `root`, create it, and make it executable with `chmod +x`) |
| Reference implementation | [start-hook-init-user.sh](start-hook-init-user.sh) (the steps below are based on this script) |
| Storage location | Place `start-hook-init-user.sh` anywhere under `/workspace` on your Storage volume; invoke it from `start-hook.sh` |

### Step 2: Create user directories

Under `/workspace` on Storage, set up the following directory structure:

1. Create the `.users` directory (full path: `/workspace/.users`)
2. Under `.users`, create an **empty directory for each user**, using the desired username as the directory name (for example, `myuser` → `/workspace/.users/myuser`)

The script scans subdirectory names under `.users` and creates matching system users. Directories may be empty; no files need to be placed in them beforehand.

After these two steps, configuration is complete.

## Usage

### Automatic execution

After the instance starts, `start-hook-init-user.sh` runs automatically and creates the users defined under `.users`.

### Manual execution

If the instance was already running when you finished configuration, log in as `root` and run `start-hook-init-user.sh` manually. The result is the same as running it at startup.

## SSH login

### Obtaining the login command

The SSH command shown in the console defaults to the `root` user. Example format:

```text
ssh jp-***********@ssh-us-west-2.pyromind.ai -p 2222
```

### Logging in with a custom username

Append `+username` between the instance identifier and the hostname to specify the target user (the system user must already exist from the steps above):

```text
ssh jp-***********+myuser@ssh-us-west-2.pyromind.ai -p 2222
```

| Login type | Example command |
|------------|-----------------|
| `root` | `ssh jp-***********@ssh-us-west-2.pyromind.ai -p 2222` |
| Custom user `myuser` | `ssh jp-***********+myuser@ssh-us-west-2.pyromind.ai -p 2222` |

## Further customization

Once configured, multiple users can log in independently to the same JupyterLab instance. For additional requirements (such as permissions or environment variables), adjust the logic in `start-hook-init-user.sh` as needed.
