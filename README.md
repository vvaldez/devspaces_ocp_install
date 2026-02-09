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

Provide values for the `vars` section in `install_devspaces.yml` to suit your environment. Best practice is to store any credentials in a separate file that is NOT committed to this repository. Something like `secrets.yml`. For details on all the variables see the role readme.

Here is a sample that can be used to install on a fresh cluster:

### `secrets.yml`
```yaml
---
devspaces_ocp_install_connection:
  host: "https://api.example.com:6443/"
  username: "kubeadmin"
  password:
  validate_certs: false
devspaces_ocp_install_namespace: openshift-devspaces
devspaces_ocp_install_operator:
  channel: stable
devspaces_ocp_install_instance:
  instance_name: devspaces
  storage_pvc_strategy: per-user
...
```

## Usage

. Run the playbook:

```bash
ansible-playbook install_devspaces.yml -e @secrets.yml
```

## Access Dev Spaces

After successful installation, you'll see a message like:

```
TASK [devspaces_ocp_install : Display Dev Spaces URL]
ok: [localhost] => {
    "msg": "Dev Spaces is ready! Access the dashboard at: https://devspaces-openshift-devspaces.apps.your-cluster.example.com"
}
```

You can also access it from the OpenShift Console:
- Navigate to the Applications menu (9-square grid icon)
- Look for "Dev Spaces (devspaces)" under "Red Hat Applications"

## Troubleshooting

### Check Operator Installation Status

```bash
oc get csv -n openshift-devspaces
oc get subscription -n openshift-devspaces
```

### Check CheCluster Status

```bash
oc get checluster -n openshift-devspaces
oc describe checluster devspaces -n openshift-devspaces
```

### View Operator Logs

```bash
oc logs -n openshift-devspaces deployment/devspaces-operator -f
```

## Next Steps

- Read the full [README](roles/devspaces_ocp_install/README.md) for detailed documentation
- Customize the role variables for your specific needs
- Integrate into your existing Ansible automation

## Getting Help

- OpenShift Dev Spaces Documentation: https://access.redhat.com/documentation/en-us/red_hat_openshift_dev_spaces
- Original playbook: https://github.com/vvaldez/devspaces_ocp_install
- Role structure based on: https://github.com/redhat-cop/aap_utilities

## License

GPL-3.0-or-later
