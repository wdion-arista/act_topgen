# ACT & ContainerLab Topology Generation

This Ansible role generates topology files for both **Arista Cloud Test (ACT)** and **ContainerLab (clab)** from AVD structured config files.

It reads the `ethernet_interfaces` from each device's structured config, extracts peer connection data, and builds the topology link definitions automatically. Supports both AVD 5.x (top-level peer fields) and AVD 6.x (`metadata` sub-key) structured config formats.

## Prerequisites

You must run the AVD `eos_designs` role first to generate structured configs before this role can be used.

## Example Playbook

Use the same top-level group you have for your fabric as `hosts`. Override role variables to customize the topology output.

```yaml
---
- name: Build ACT / ContainerLab Topology
  hosts: FABRIC
  connection: local
  gather_facts: false

  tasks:
    - name: Generate Topology Files
      import_role:
        name: act-topgen
```

## What Gets Generated

| Output | Location | Description |
|--------|----------|-------------|
| ACT topology | `act/topology.yml` | Nodes, links, and node_defaults for CE ACT |
| ContainerLab topology | `clab/<name>.clab.yml` | Nodes, links, and kinds for containerlab |
| Interface mapping | `clab/interface_mapping.json` | Maps cEOS `eth0` to `Management1` and all Ethernet interfaces used in the topology |
| Base configs | `clab/configs/<hostname>.cfg` | Vanilla startup configs per device |

## Role Variables

### Input / Output

| Variable | Default | Description |
|----------|---------|-------------|
| `structured_folder` | `intended/structured_configs/` | Path to AVD structured config YAML files |
| `avd_structured_config_file_format` | `yml` | File extension of structured configs |
| `output_folder` | `act/` | ACT topology output directory |
| `output_filename` | `topology.yml` | ACT topology output filename |

### ACT Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `act_eos_version` | `4.30.4M` | EOS version for ACT vEOS nodes |
| `act_generic_os_version` | `Rocky-8.5` | OS version for ACT generic nodes |
| `act_tools_os_version` | `ubuntu-2204-lts` | OS version for ACT tools-server |
| `act_cvp_version` | `2023.1.3` | CVP version for ACT |
| `act_device_user` | `cvpadmin` | Default username for EOS devices |
| `act_device_password` | `""` | Default password for EOS devices |
| `act_default_ports` | `[]` | Extra ports to add to all EOS nodes |
| `act_use_device_models` | `true` | Include device models from structured config metadata |
| `act_cvp_user` | `root` | CVP username |
| `act_cvp_password` | `""` | CVP password |
| `act_cvp_instance_type` | `singlenode` | CVP instance type |
| `act_cvp_ip` | `192.168.0.5` | CVP management IP |
| `act_cvp_auto_configuration` | `true` | Enable CVP auto-configuration |
| `act_cvp_size` | `medium` | CVP instance size (`medium`, `large`, `xlarge`) |
| `act_tools_server_ip` | `192.168.0.6` | Tools-server management IP |
| `act_tools_server_size` | `medium` | Tools-server instance size |
| `act_veos_size` | `medium` | Default vEOS instance size |
| `act_add_cvp` | `true` | Add CVP node to ACT topology |
| `act_add_tools_server` | `true` | Add tools-server node to ACT topology |
| `act_add_connected_nodes` | `false` | Add non-fabric nodes (servers, endpoints) to topology |
| `act_connected_nodes_map` | `{other: veos}` | Map AVD `peer_type` to ACT `node_type` |
| `act_connected_nodes_range` | `192.168.0.128/25` | IP range for auto-assigned connected node IPs |
| `act_use_old_connections` | `false` | Use legacy `nodes[].neighbors` format instead of `links[]` |
| `act_veos_internet_access` | `false` | Enable internet access on vEOS devices |

### ContainerLab Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `clab_name` | `demo` | ContainerLab topology name |
| `clab_prefix` | `""` | ContainerLab name prefix (empty = use clab default) |
| `clab_output_folder` | `{{ inventory_dir }}/clab/` | ContainerLab output directory |
| `clab_output_filename` | `topology.clab.yml` | ContainerLab topology filename |
| `clab_eos_version` | `arista/ceos:4.34.1F` | cEOS container image |
| `clab_linux_image` | `ghcr.io/aristanetworks/aclabs/host-ubuntu:rev1.0` | Linux container image for servers |
| `clab_add_tools_server` | `false` | Add a tools-server linux node |
| `clab_tools_server_ip` | `null` | Tools-server management IP |
| `clab_config_dir` | `configs` | Directory for startup config files |
| `clab_config_default` | `default.cfg` | Default startup config filename |
| `clab_avd_configs` | `{{ inventory_dir }}/intended/configs` | Path to AVD-generated EOS configs |
| `clab_structured_folder` | `{{ inventory_dir }}/clab/intended/structured_configs` | Path to clab structured configs |
| `clab_device_default` | `true` | Use vanilla configs (`true`) or AVD configs (`false`) as startup |
| `clab_static_mgmt_ip` | `true` | Assign static management IPs from inventory |
| `clab_interface_mapping_file` | `interface_mapping.json` | Interface mapping filename for cEOS |
| `clab_mgmt_interface_name` | `Management1` | cEOS management interface name (AVD 6.x default) |
| `clab_connected_nodes_map` | `{server: linux, workstation: linux}` | Map AVD `peer_type` to containerlab node `kind` |

### Interface Mapping (AVD 6.x)

AVD 6.x in digital twin mode defaults the management interface to `Management1` instead of `Management0`. The role automatically:

1. Generates an `interface_mapping.json` that maps `eth0` to `Management1` plus all Ethernet interfaces used in the topology
2. Binds the mapping file into each cEOS container at `/mnt/flash/EosIntfMapping.json`
3. Sets `enforce-startup-config: true` on the `arista_ceos` kind

To change the management interface name, override `clab_mgmt_interface_name`.

## AVD Compatibility

The role supports both AVD 5.x and 6.x structured config formats:

- **AVD 5.x**: `peer`, `peer_interface`, `peer_type` are top-level keys on each `ethernet_interfaces` entry
- **AVD 6.x**: These fields moved under a `metadata` sub-key

The templates automatically check `metadata` first and fall back to top-level keys.

## Inventory Update Script

The `update_clab_act_inventory` script (located at `common/scripts/update_clab_act_inventory` in [ceos-act-projects](https://github.com/wdion-arista/ceos-act-projects)) generates environment-specific inventory files from your production `inventory.yml`. It reads IP addresses from either a ContainerLab or ACT deployment and produces a new inventory with updated `ansible_host` values and the appropriate environment group name.

### ContainerLab Inventory

After deploying a containerlab topology, generate `inventory-containerlab.yml` from the clab-generated ansible inventory:

```bash
python common/scripts/update_clab_act_inventory \
  --inv_file sites/eastcoast/inventory.yml \
  --fabric_name EASTCOAST_FABRIC \
  --env_name PROD \
  --clab sites/eastcoast/clab/clab-MandE-eastcoast/ansible-inventory.yml \
  --site eastcoast
```

This:
- Copies the source inventory
- Replaces `ansible_host` IPs with the containerlab management IPs
- Renames the `PROD` group to `CLAB`
- Removes `clab_mgmt_ip` and `act_mgmt_ip` host vars
- Saves to `inventory-containerlab.yml`

### ACT Inventory

After deploying an ACT lab, generate `inventory-act.yml` from the ACT lab resource file:

```bash
python common/scripts/update_clab_act_inventory \
  --inv_file sites/eastcoast/inventory.yml \
  --fabric_name EASTCOAST_FABRIC \
  --env_name PROD \
  --act sites/eastcoast/act/lab-resource.yml
```

This:
- Copies the source inventory
- Replaces `ansible_host` IPs with ACT `internal_ip` values
- Adds `cloud_ip` for each device
- Renames the `PROD` group to `ACT`
- Updates tools-server and server IPs if present
- Saves to `inventory-act.yml`

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--inv_file` | Yes | Path to the source `inventory.yml` |
| `--fabric_name` | Yes | Top-level fabric group name (e.g., `EASTCOAST_FABRIC`) |
| `--env_name` | No | Parent environment group to rename (e.g., `PROD`) |
| `--clab` | No* | Path to containerlab's `ansible-inventory.yml` |
| `--act` | No* | Path to ACT lab resource YAML |
| `--site` | No | Site name, appended to `PROJECT_NAME` for clab hostname stripping |
| `--inventory-type` | No | Inventory structure: `default` (group -> hosts) or `sub_groups` (group -> children -> sub_group -> hosts). Defaults to `default` or `INVENTORY_TYPE` env var |
| `--verbose` | No | Enable verbose output |

\* One of `--clab` or `--act` is required.

### Environment Variables

| Variable | Description |
|----------|-------------|
| `PROJECT_NAME` | Project name used in containerlab hostnames (set in `.env`, default: `test`) |
| `INVENTORY_TYPE` | Default inventory structure type (set in `.env`, default: `default`) |
