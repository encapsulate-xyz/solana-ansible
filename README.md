# Ansible Playbook to deploy Solana Node

This Ansible playbook automates the deployment and configuration of Solana Node. For this setup we are using [Jito Foundation MEV Solana Client](https://github.com/jito-foundation/jito-solana). It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as identity and vote account keys. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves secrets from HashiCorp Vault. The keys are expected to follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`

i.e.:
- `mainnet/solana/encapsulate/validator/validator-keypair.json`
- `mainnet/solana/encapsulate/validator/vote-account-keypair.json`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/solana-ansible.git
cd solana-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for testnet.yml

```yaml
---
# maintains mainnet inventory grouped by project names
all:
  vars:
    env: mainnet
  children:
    solana:
      hosts:
        validator.solana.mainnet.encapsulate.xyz:
          type: validator
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the source urls, ports and configurations.

```
---
# maintains ports, bianry or repo urls

org: your_org
domain_regex: ^([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}$
monitor_server_dns: update_the_dns_of_our_monitor_server
ansible_port: update_your_server_ssh_port

architecture_map:
  x86_64: amd64
  aarch64: arm64

default_node_types:
  - validator
  - default

tmp_dir: /tmp

solana:
  validator:
    source_url: https://github.com/jito-foundation/jito-solana/releases/download
    ports:
      rpc_port: 8899    # You can define the rpc port of your wish
      dynamic_port_range: 8000-8020     # You can configure a dynamic port range for Solana to use. It’s recommended to have at least 20 free ports available for for solana to utilize
```

- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains network specific variables.
```
---
# maintains versions and can overwrite all.yml
solana:
  validator:
    version: the_latest_jito_client_version
    known_validators:
      - add
      - known
      - validators
      - [refererence](https://docs.anza.xyz/clusters/available)
    entrypoints:
      - add
      - entrypoints
      - [refererence](https://docs.anza.xyz/clusters/available)
    expected_genesis_hash: [refererence](https://docs.anza.xyz/clusters/available)
    limit_ledger_size:  [refererence](https://github.com/agjell/sol-tutorials/blob/master/solana-validator-faq.md#6b-how-big-is-the-ledger-how-much-storage-space-do-i-need-for-my-validator)
    tip_payment_program_pubkey: [refererence](https://jito-foundation.gitbook.io/mev/jito-solana/command-line-arguments)
    tip_distribution_program_pubkey: [refererence](https://jito-foundation.gitbook.io/mev/jito-solana/command-line-arguments)
    merkle_root_upload_authority: [refererence](https://jito-foundation.gitbook.io/mev/jito-solana/command-line-arguments)
    commission_bps: set_your_desired_commision_rate
    relayer_url: [refererence](https://jito-foundation.gitbook.io/mev/jito-solana/command-line-arguments)
    block_engine_url: [refererence](https://jito-foundation.gitbook.io/mev/jito-solana/command-line-arguments)
    shred_receiver_address: [refererence](https://jito-foundation.gitbook.io/mev/jito-solana/command-line-arguments)
```

- **`group_vars/vault.yml`**: Store secret variables, such as HCP Vault url, passwords in this file.

```
# maintains anything sensitive like api keys
vault:
  default:
    hcp:
      vault:
        url: "your_hcp_vault_url"
  mainnet:
    external:
      metrics:
        password: solana_mainnet_metrics_password
  testnet:
    external:
      metrics:
        password: solana_testnet_metrics_password
```

### Usage

1. First, install the dependencies:

  ```bash
   ansible-galaxy install -r collections/requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Then run the playbook:

  ```bash
  ansible-playbook setup_node.yml -l validator.solana.mainnet.encapsulate.xyz -e "fetch_validator_keys=true snapshot_fetch=true"
  ```

**Note**: The variable `fetch_validator_keys` enables fetching secrets from HCP Vault, while `snapshot_fetch` allows syncing from a snapshot when setting up the validator for the first time.

After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [validator.solana.mainnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.solana.mainnet.encapsulate.xyz",
        "Groups: ['solana']",
        "Project: solana",
        "Environment: testnet",
        "Type: validator",
        "Version: v2.2.19-jito",
        "Username: solana",
        "Service Name: solana",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] ********************************************************************************************************************
Pausing for 40 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [validator.solana.mainnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [validator.solana.mainnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.solana.mainnet.encapsulate.xyz",
        "Project: solana",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [validator.solana.mainnet.encapsulate.xyz]
```
