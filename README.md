# ansible-netbox-integration

An Ansible playbook that queries a [NetBox](https://netbox.dev/) instance for network device data (device details, interfaces, and IP addresses) and renders a configuration file for each device using a Jinja2 template.

---

## How it works

1. Fetches device information from the NetBox DCIM API.
2. Fetches interface information for the device.
3. Fetches IP addresses assigned to the device from the NetBox IPAM API.
4. Renders a configuration file from the Jinja2 template at `templates/device_config.j2`.
5. Saves the rendered config to `/tmp/<device_name>_config.txt` on the target host.

---

## Prerequisites

- Ansible 2.9+
- Network access to your NetBox instance
- A valid NetBox API token with at least **read** permissions on DCIM and IPAM
- An inventory file listing your target hosts

---

## Required variables

These variables must be supplied at runtime — they are intentionally **not** stored in the repository.

| Variable | Description |
|---|---|
| `netbox_url` | Base URL of your NetBox instance, e.g. `https://netbox.example.com` |
| `netbox_token` | NetBox API authentication token |
| `device_name` | Name of the device to look up in NetBox (must match exactly) |

---

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/kevinmrr/ansible-netbox-integration.git
cd ansible-netbox-integration
```

### 2. Create an inventory file

Create a file named `inventory.ini` (not committed to the repo) listing your target hosts:

```ini
[all]
mydevice.example.com
```

### 3. Run the playbook

Pass required variables with `-e` on the command line:

```bash
ansible-playbook -i inventory.ini netbox-integration.yml \
  -e "netbox_url=https://netbox.example.com" \
  -e "netbox_token=YOUR_API_TOKEN" \
  -e "device_name=mydevice"
```

Or use a variables file that you keep locally and outside version control:

```bash
ansible-playbook -i inventory.ini netbox-integration.yml \
  -e "@my_vars.yml"
```

Example `my_vars.yml` (never commit this file):

```yaml
netbox_url: "https://netbox.example.com"
netbox_token: "YOUR_API_TOKEN"
device_name: "mydevice"
```

### 4. Check the output

The configuration file is written to `/tmp/<device_name>_config.txt` on the target host. You can retrieve it with:

```bash
ansible all -i inventory.ini -m fetch \
  -a "src=/tmp/mydevice_config.txt dest=./output/ flat=yes"
```

---

## Project structure

```
ansible-netbox-integration/
├── netbox-integration.yml   # Main playbook
└── templates/
    └── device_config.j2     # Jinja2 configuration template
```

---
