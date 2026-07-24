# act-topgen

Ansible role that generates topology files for **Arista Cloud Test (ACT)** and **ContainerLab** from AVD structured configurations.

Supports AVD 5.x and 6.x structured config formats (peer fields at top-level or under `metadata`).

## Requirements

- AVD `eos_designs` role must be run first to generate structured configs
- `arista.avd` Ansible collection installed

## Role Variables

See the top-level [README.md](../README.md) for the full variable reference.

## Dependencies

None.

## License

Apache-2.0
