# netpenguins.ludus_k3s

Installs and configures a lightweight k3s Kubernetes cluster on Linux hosts. Supports `server` (control plane) and `agent` (worker) installations.

## 📝 Quick notes

- Targets Linux hosts (Ubuntu/Debian)
- [🚧 WIP] Dashboard: the role can deploy a dashboard. `ludus_k3s_dashboard_type` supports `"kubernetes"`, or empty (none).
- I usually spin up a Windows machine in my config installing Headlamps desktop app via choco for inspecting/managing clusters. Headlamp is lightweight and easy to run — https://headlamp.dev

## 🛠️ Installation in Ludus

#### Install via Ansible Galaxy:

```sh
ludus ansible role add netpenguins.ludus_k3s
```

#### Or clone directly:

```sh
git clone https://github.com/netpenguins/ludus_k3s.git /opt/netpenguins.ludus_k3s
ludus ansible role add /opt/netpenguins.ludus_k3s
```

## ⚡️ Role Variables

Available variables (see `defaults/main.yml`):

| Variable | Description | Default/Example |
|---|---|---|
| `ludus_k3s_role` | Install mode: `server` (control plane) or `agent` (worker) | `server` |
| `ludus_k3s_version` | k3s version to install (empty = latest) | `""` |
| `ludus_k3s_server_args` | Extra arguments passed to k3s server | `""` |
| `ludus_k3s_agent_args` | Extra arguments passed to k3s agent | `""` |
| `ludus_k3s_server_url` | k3s server URL (required when role is `agent`) | `""` |
| `ludus_k3s_token` | Cluster join token (servers auto-generate if blank) | `""` |
| `ludus_k3s_disable_components` | List of components to disable (e.g. `traefik`, `servicelb`) | `[]` |
| `ludus_k3s_kubeconfig_mode` | File mode for written kubeconfig | `"644"` |
| `ludus_k3s_install_dir` | Directory to install k3s binary | `/usr/local/bin` |
| `ludus_k3s_data_dir` | k3s data directory | `/var/lib/rancher/k3s` |
| `ludus_k3s_dashboard` | Whether to install a dashboard | `false` |
| `ludus_k3s_dashboard_type` | Dashboard type: `"kubernetes"`, `"headlamp"`, or empty | `""` |

## Usage
### Example Ludus Configuration
```
# yaml-language-server: $schema=https://docs.ludus.cloud/schemas/range-config.json
ludus:
  # k3s Server (Control Plane)
  - vm_name: "{{ range_id }}-k3s-1"
    hostname: "{{ range_id }}-k3s-server"
    template: debian-12-x64-server-template
    vlan: 10
    ip_last_octet: 10
    ram_gb: 4
    cpus: 2
    linux: true
    roles:
      - ludus_k3s
    role_vars:
      ludus_k3s_role: server
      ludus_k3s_disable_components:
        - traefik
  # k3s Agent 1 (Worker Node)
  - vm_name: "{{ range_id }}-k3s-agent-1"
    hostname: "{{ range_id }}-k3s-agent-1"
    template: debian-12-x64-server-template
    vlan: 10
    ip_last_octet: 11
    ram_gb: 2
    cpus: 2
    linux: true
    roles:
      - ludus_k3s
    role_vars:
      ludus_k3s_role: agent
      ludus_k3s_server_url: "https://10.{{ range_second_octet }}.10.10:6443"
  # k3s Agent 2 (Worker Node)
  - vm_name: "{{ range_id }}-k3s-agent-2"
    hostname: "{{ range_id }}-k3s-agent-2"
    template: debian-12-x64-server-template
    vlan: 10
    ip_last_octet: 12
    ram_gb: 2
    cpus: 2
    linux: true
    roles:
      - ludus_k3s
    role_vars:
      ludus_k3s_role: agent
      ludus_k3s_server_url: "https://10.{{ range_second_octet }}.10.10:6443"
  #########################################################
  # For a nicer UI experience, add a Windows VM with 
  #   headlamp and add your k3s cluster kubeconfig to it
  #########################################################
  - vm_name: "{{ range_id }}-win11-1"
    hostname: "{{ range_id }}-WIN11-22H2-1"
    template: win11-22h2-x64-enterprise-template
    vlan: 10
    ip_last_octet: 21
    ram_gb: 8
    cpus: 4
    windows:
      install_additional_tools: true
      chocolatey_ignore_checksums: true
      chocolatey_packages:
        - vscodium
        - headlamp
        - kubernetes-helm # <-- Needed if you wish to use helm charts in headlamp
```

## 🔗 Links

- k3s: https://k3s.io/
- Headlamp: https://headlamp.dev

## For Ludus, by NetPenguins

Happy Hacking🐧

