# ACT Scripts

## ACT Inventory for act and containerlabs

The script build both an act or clab inventory file separate from your main prod inventory

inventory.yaml
```yaml
---
all:
  children:
    CVP:
      hosts:
        cvp:
    PROD: 
      children:
        SITE1_FABRIC:
          children:
            SITE1_SPINES:
              hosts:
                s1-spine1:
                  ansible_host: 10.0.1.10
                  clab_mgmt_ip: 172.20.20.110
                  act_mgmt_ip: 192.168.1.10
                s1-spine2:
                  ansible_host: 10.0.1.11
                  clab_mgmt_ip: 172.20.20.111
                  act_mgmt_ip: 192.168.1.11
            SITE1_LEAFS:
              hosts:
                s1-leaf1:
                  ansible_host: 10.0.1.20
                  clab_mgmt_ip: 172.20.20.120
                  act_mgmt_ip: 192.168.1.20
                s1-leaf2:
                  ansible_host: 10.0.1.21
                  clab_mgmt_ip: 172.20.20.121
                  act_mgmt_ip: 192.168.1.21
                s1-leaf3:
                  ansible_host: 10.0.1.22
                  clab_mgmt_ip: 172.20.20.122
                  act_mgmt_ip: 192.168.1.22
                s1-leaf4:
                  ansible_host: 10.0.1.23
                  clab_mgmt_ip: 172.20.20.123
                  act_mgmt_ip: 192.168.1.23
    SITE1_FABRIC_SERVICES:
      children:
        SITE1_SPINES:
        SITE1_LEAFS:
    SITE1_FABRIC_PORTS:
      children:
        SITE1_SPINES:
        SITE1_LEAFS:

```

```bash
python3 scripts/update_clab_act_inventory --inv_file "inventory.yml" \
--fabric_name "SITE1_FABRIC" \
--site "site1" \
--env_name "PROD" \
--act "temp/act-api-inventory.yml" # if no --act is supplied it will update the containerlab inventory

```

If you have ce_act_cli installed you can pull directly from act.
https://github.com/wdion-arista/ce_act_cli

```bash
ce_act labs read -n "act_site_lab_name" -o yaml > "act-api-inventory.yml"
```

