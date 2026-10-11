# Percona Server for MongoDB Installation: Single-Node Replica Set

[English](README.md) | [Українська](README.uk.md)

## Requirements

- **Ansible Core 2.15 or later** — installed on your local machine.
- **The Ansible `community.mongodb` collection** — installed on your local machine.
- **PyMongo 4.8.0** — installed on the remote servers automatically during playbook execution.

## Initial Environment Setup (Ubuntu)

### 1. Install Ansible and the community.mongodb Collection

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible-core
ansible-galaxy collection install community.mongodb
```

### 2. Create a Working Directory

```bash
mkdir -p ~/LustyKrusty_github
```

### 3. Clone the Repository

To clone the repository over SSH, you need an SSH key added to your GitHub account.

```bash
cd ~/LustyKrusty_github/
git clone git@github.com:LustyKrusty/percona_mongodb_install_one_node_replica.git
```

### 4. Encrypt secrets.yaml

Fill in `secrets.yaml` with your own values, then encrypt the file using Ansible Vault:

```bash
cd ~/LustyKrusty_github/percona_mongodb_install_one_node_replica/
ansible-vault encrypt secrets.yaml
```

You will need the password you set during encryption when running the playbook.

### 5. Configure ansible.cfg

```bash
cd ~/LustyKrusty_github/percona_mongodb_install_one_node_replica/
cp ansible.cfg_example ansible.cfg
nano ansible.cfg
```

Set your own values for the following parameters:

```ini
remote_user = YOUR_SSH_USER
private_key_file = /path/to/your/private_key
```

- `remote_user` — the user used to connect to the remote servers over SSH.
- `private_key_file` — the path to your private SSH key.

## hosts Inventory Guidelines

When adding a new server to the `hosts` inventory file, use the following format:

```ini
[prod_hetzner]
hetzner_mongodb.local rs_prio=10 rs_role=primary env=prod data_center=amazon location=eu-central-a
```

You can use an IP address, such as `172.1.1.1`, instead of a DNS name. Replace the example group and variable values to match your environment.

| Field | Description |
| --- | --- |
| `[prod_hetzner]` | The server group name. Use the server prefix according to your naming convention. |
| `hetzner_mongodb.local` | The server's DNS name or IP address. |
| `rs_role` | The server role: `primary`, `secondary`, or `mongos`. The `primary` and `secondary` roles apply to replica set members; `mongos` is a sharded cluster router. |
| `env` | The environment, such as `prod`. |
| `data_center` | The cloud provider, such as `amazon`, `google`, or `hetzner`. |
| `location` | The server location according to your naming convention, such as `eu-central-a`. |
| `rs_prio` | The member's priority in primary elections, such as `10`, `5`, or `0`. Give your preferred primary a higher priority than each other member. A member with priority `0` cannot become primary. |

## Running the Playbook

```bash
cd ~/LustyKrusty_github/percona_mongodb_install_one_node_replica/
ansible-playbook install_mongodb.yaml --ask-vault-pass
```

When prompted, enter the Ansible Vault password you set when encrypting `secrets.yaml`.

