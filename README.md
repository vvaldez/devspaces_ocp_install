# OpenShift Dev Spaces Automation

This repository contains an Ansible playbook to automate the installation of **Red Hat OpenShift Dev Spaces**. 

The playbook handles the full deployment lifecycle:
- **Cleanup**: Removes stale CRDs and Subscriptions to prevent webhook conversion/validation errors from previous attempts.
- **Operator Installation**: Sets up the Namespace, OperatorGroup (Cluster-wide), and Subscription.
- **Readiness Checks**: Waits for the Operator pod and Webhook service to be fully healthy.
- **Cluster Deployment**: Configures a default `CheCluster` instance.
- **URL Extraction**: Retrieves the final Dev Spaces Dashboard URL once the instance is Active.

## Prerequisites

* **OpenShift Cluster**: Version 4.20+ recommended.
* **Tools**: `oc` CLI tool and `ansible` installed locally.
* **Ansible Collection**:
  ```bash
  ansible-galaxy collection install -r collections/requirements.yml
  ```

##  Configuration

Modify the `vars` section in `install_devspaces.yml` to suit your environment:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `namespace` | `openshift-devspaces` | The project where the operator and CheCluster will be created. |
| `operator_channel` | `stable` | The OLM subscription channel (use `stable` or a specific version like `v3.16`). |

## Usage

1. Login to your cluster:
```bash
oc login --token=<your-token> --server=[https://api.your-cluster.com:6443](https://api.your-cluster.com:6443)
```

2. Run the playbook:
```bash
ansible-playbook install_devspaces.yml
```