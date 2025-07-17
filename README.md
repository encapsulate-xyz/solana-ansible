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

| Name                                         | Default Value                                                       | Description                                                  |
|----------------------------------------------|----------------------------------------------------------------------|--------------------------------------------------------------|
| `org`                                        | `encapsulate`                                                       | Organization name used for internal config                   |
| `domain_regex`                               | `^([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}$`                                   | Regex pattern to validate domain names                       |
| `monitor_server_dns`                         | `monitor.encapsulate.xyz`                                           | DNS address of the monitoring server                         |
| `ansible_port`                               | `8192`                                                               | SSH port used by Ansible to connect to nodes                 |
| `solana.validator.source_url`                | `https://github.com/jito-foundation/jito-solana/releases/download` | Source URL to download the Jito Solana validator binary      |
| `solana.validator.ports.rpc_port`            | `8899`                                                               | Port used for JSON RPC interface                             |
| `solana.validator.ports.dynamic_port_range`  | `8000–8020`                                                          | Range of dynamic ports used for validator communication      |

- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains network specific variables.

| Name                                         | Default Value                                                        | Description                                                                                  |
|----------------------------------------------|-----------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| `solana.validator.version`                   | `v2.2.19-jito`                                                        | Version of the Jito Solana validator binary                                                  |
| `solana.validator.known_validators`          | `7Np41oeYqPefeNQEHSv1UDhYrehxin3NStELsSKCT4K2`,<br> `GdnSyH3YtwcxFvQrVVJMm1JhTS4QVX7MFsX56uJLUfiZ`,<br> `DE1bawNcRJB9rVm3buyMVfr8mBEoyyu73NBovf2oXJsJ`, <br> `CakcnaRDHka2gXyfbEd2d3xsvkJkqsLw2akB3zsN1D2S` | List of known validator identity pubkeys                                                    |
| `solana.validator.entrypoints`               | `entrypoint.mainnet-beta.solana.com:8001`,<br> `entrypoint2.mainnet-beta.solana.com:8001`,<br> `entrypoint3.mainnet-beta.solana.com:8001`,<br> `entrypoint4.mainnet-beta.solana.com:8001`,<br> `entrypoint5.mainnet-beta.solana.com:8001` | List of validator network entrypoints                                                        |
| `solana.validator.expected_genesis_hash`     | `5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d`                        | Expected genesis hash of the cluster                                                         |
| `solana.validator.limit_ledger_size`         | *(empty string)*                                                     | Enables ledger pruning if set; currently disabled                                            |
| `solana.validator.tip_payment_program_pubkey`| `T1pyyaTNZsKv2WcRAB8oVnk93mLJw2XzjtVYqCsaHqt`                         | Public key for Jito tip payment program                                                      |
| `solana.validator.tip_distribution_program_pubkey` | `4R3gSG8BpU4t19KYj8CfnbtRpnT8gtk4dvTHxVRwc2r7`                    | Public key for Jito tip distribution program                                                 |
| `solana.validator.merkle_root_upload_authority` | `GZctHpWXmsZC1YHACTGGcHhYxjdRqQvTpYkb9LMvxDib`                    | Authority to upload merkle roots                                                             |
| `solana.validator.commission_bps`            | `800`                                                                | Commission rate in basis points (800 = 8%)                                                   |
| `solana.validator.relayer_url`               | `http://amsterdam.mainnet.relayer.jito.wtf:8100`                     | URL of the Jito relayer                                                                      |
| `solana.validator.block_engine_url`          | `https://mainnet.block-engine.jito.wtf`                              | URL of the block engine                                                                      |
| `solana.validator.shred_receiver_address`    | `74.118.140.240:1002`                                                | IP address and port to receive shreds                                                        |

- **`group_vars/vault.yml`**: Store secret variables, such as HCP Vault url, passwords in this file.

| Name                                      | Default Value                | Description                                                |
|-------------------------------------------|-------------------------------|------------------------------------------------------------|
| `vault.default.hcp.vault.url`             | `"your_hcp_vault_url"`        | URL of the HCP Vault used for secure secret storage        |
| `vault.mainnet.external.metrics.password` | `solana_mainnet_metrics_password` | Password used to authenticate metrics access on mainnet   |
| `vault.testnet.external.metrics.password` | `solana_testnet_metrics_password` | Password used to authenticate metrics access on testnet   |

### Usage

1. First, install the dependencies:

  ```bash
   ansible-galaxy install -r collections/requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Then run the playbook:

  ```bash
  ansible-playbook setup_node.yml -l validator.solana.mainnet.encapsulate.xyz -e "fetch_validator_keys=true fetch_snapshot=true"
  ```

**Note**: The variable `fetch_validator_keys` enables fetching secrets from HCP Vault, while `fetch_snapshot` allows syncing from a snapshot when setting up the validator for the first time.

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
