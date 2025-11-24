# Volume Mounts: How to Make Them Writable

## Overview

This document explains how `volume_mounts` is used in the codebase and the changes made to support writable volumes.

## How Volume Mounts Work

### Data Flow

1. **ComputerConfiguration** (`code_execution_interface.py`)
   - Defines `volume_mounts: list[VolumeMount]` 
   - Each `VolumeMount` has: `host_path`, `container_path`, and `read_only` flag

2. **Conversion Layer** (`task_to_alcatraz_config.py`)
   - Converts `volume_mounts` → Alcatraz `volumes_config` format
   - Maps to: `{volume_name: {bind_source, bind_dest, mode}}`

3. **Alcatraz Runtime** (`local.py`)
   - Reads `volumes_config` from the cluster config
   - Creates Docker volume bindings with the specified mode
   - Mode can be "ro" (read-only) or "rw" (read-write)

### Volume Mount Definition

```python
class VolumeMount(BaseModel):
    """Mounts files from the host onto the container."""
    host_path: str
    container_path: str
    read_only: bool = True  # Defaults to read-only for security
```

## Changes Made

### 1. Added `read_only` field to `VolumeMount`
**File:** `/project/common/nanoeval/nanoeval/solvers/computer_tasks/code_execution_interface.py`

- Added `read_only: bool = True` field to the `VolumeMount` class
- Defaults to True for security reasons
- Updated field description to reflect that volumes can be writable

### 2. Added conversion logic for `volume_mounts`
**File:** `/project/common/nanoeval_alcatraz/nanoeval_alcatraz/task_to_alcatraz_config.py`

- Added code to convert `task.volume_mounts` to Alcatraz `volumes_config` format
- Converts `read_only=True` → `mode="ro"` and `read_only=False` → `mode="rw"`
- Merges with any existing `volumes_config` (volume_mounts takes precedence)

## How to Use Writable Volume Mounts

### Example Usage

```python
from nanoeval.solvers.computer_tasks.code_execution_interface import (
    ComputerConfiguration, 
    VolumeMount
)

config = ComputerConfiguration(
    cwd="/root",
    docker_image="your-image:tag",
    volume_mounts=[
        # Read-only mount (default)
        VolumeMount(
            host_path="/path/on/host/data",
            container_path="/data",
            # read_only=True is the default
        ),
        # Writable mount
        VolumeMount(
            host_path="/path/on/host/output",
            container_path="/output",
            read_only=False  # This makes it writable!
        ),
    ]
)
```

### What Happens Behind the Scenes

When the above configuration is used:

1. `task_to_alcatraz_config()` converts it to:
   ```python
   {
       "volume_0": {
           "bind_source": "/path/on/host/data",
           "bind_dest": "/data",
           "mode": "ro"
       },
       "volume_1": {
           "bind_source": "/path/on/host/output",
           "bind_dest": "/output",
           "mode": "rw"  # Writable!
       }
   }
   ```

2. Alcatraz creates Docker volumes with these bindings:
   - `/path/on/host/data` → `/data` (read-only)
   - `/path/on/host/output` → `/output` (read-write)

## Key Files

- **VolumeMount definition:** `project/common/nanoeval/nanoeval/solvers/computer_tasks/code_execution_interface.py` (lines 166-173)
- **Conversion logic:** `project/common/nanoeval_alcatraz/nanoeval_alcatraz/task_to_alcatraz_config.py` (lines 42-57)
- **Alcatraz implementation:** `project/common/alcatraz/alcatraz/clusters/local.py` (lines 78-82, 609-623)

## Notes

- The `volumes_config` field in `ComputerConfiguration` is deprecated but still supported
- `volume_mounts` is the new, preferred API
- Mode defaults to read-only (`ro`) for security reasons
- Volume mount paths are validated during the `setup()` phase to ensure they exist in the container
