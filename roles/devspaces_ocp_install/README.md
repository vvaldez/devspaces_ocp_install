# devspaces_ocp_install

A role to install OpenShift Dev Spaces on OpenShift using the operator.

## Requirements

This role requires the `kubernetes` (version 12.0.0 or later) Python module.
In addition the `kubernetes.core` and `redhat.openshift` Ansible collections are required.

## Role Variables

A description of the settable variables for this role.

| Variable Name                                      | Required | Default Value                 | Description                                                                                  |
|----------------------------------------------------|:--------:|-------------------------------|----------------------------------------------------------------------------------------------|
| devspaces_ocp_install_namespace                    |   Yes    |                               | Namespace to create operator and instance in                                                 |
| devspaces_ocp_install_create_namespace             |    No    | true                          | Create the Namespace for the operator and instance. Valid values are: `true`, `false`        |
| devspaces_ocp_install_namespace_manifest_overrides |    No    |                               | YAML Manifest to override the generated `Namespace` resource                                 |
| devspaces_ocp_install_connection                   |   Yes    |                               | Dictionary containing keys defined in the `connection variables table`                       |
| devspaces_ocp_install_operator                     |   Yes*   |                               | Dictionary containing keys defined in the `operator variables table`                         |
| devspaces_ocp_install_instance                     |   Yes*   |                               | Dictionary containing keys defined in the `instance variables table`                         |

\* Variable and required keys must be defined when the type of tag is specified (e.g. `--tags instance` requires the devspaces_ocp_install_instance variable be defined).
If the variable is omitted the corresponding component will not be installed (e.g. if only devspaces_ocp_install_operator variable is defined then the instance installation will be skipped)

### devspaces_ocp_install_connection keys

| Key Name       | Required | Default Value | Description                                                  |
|----------------|:--------:|---------------|--------------------------------------------------------------|
| host           |   Yes    |               | OCP cluster to create the Dev Spaces objects in              |
| username       |   Yes*   |               | Username to use for authenticating with OCP                  |
| password       |   Yes*   |               | Password to use for authenticating with OCP                  |
| api_key        |   Yes*   |               | OCP API Token                                                |
| validate_certs |          |               | Validate SSL certificates. Valid values are: `true`, `false` |

\* Either `api_key` or `username` and `password` can be specified.

### devspaces_ocp_install_operator keys

| Key Name                         | Required | Default Value | Description                                                       |
|----------------------------------|:--------:|---------------|-------------------------------------------------------------------|
| channel                          |   Yes    |               | Channel to subscribe (e.g. stable or fast)                        |
| approval                         |          | Automatic     | Update approval method. Valid values are Automatic or Manual.     |
| starting_csv                     |          |               | Set the starting ClusterServiceVersion                            |
| operatorgroup_create             |          | true          | Create the `OperatorGroup` for the Operator                       |
| operatorgroup_manifest_overrides |          |               | YAML Manifest to override the generated `OperatorGroup` resource  |
| subscription_manifest_overrides  |          |               | YAML Manifest to override the generated `Subscription` resource   |

### devspaces_ocp_install_instance keys

| Key Name                       | Required | Default Value                        | Description                                                              |
|--------------------------------|:--------:|--------------------------------------|--------------------------------------------------------------------------|
| instance_name                  |   Yes    |                                      | Name of the CheCluster instance to create                                |
| debug                          |          | false                                | Enable debug mode for Dev Spaces server                                  |
| storage_pvc_strategy           |          | per-user                             | Storage strategy for workspaces (per-user, common, or per-workspace)     |
| create_link                    |          | true                                 | Create an OCP console application link (i.e. apply ConsoleLink CR)       |
| link_text                      |          | Dev Spaces (<INSTANCE_NAME>)         | Text used when creating the OCP console application link                 |
| instance_manifest_overrides    |          |                                      | YAML Manifest to override the generated `CheCluster` resource            |
| consolelink_manifest_overrides |          |                                      | YAML Manifest to override the generated `ConsoleLink` resource           |

## Dependencies

This role depends on the `redhat.openshift` and `kubernetes.core` collections.

## Example Playbook

The following playbook will install OpenShift Dev Spaces:

```yml
---
- name: Install Dev Spaces on OCP playbook
  hosts: localhost
  gather_facts: false

  vars:
    devspaces_ocp_install_connection:
      host: "https://api.example.com:6443"
      username: kubeadmin
      password:
      validate_certs: false
    devspaces_ocp_install_namespace: openshift-devspaces
    devspaces_ocp_install_operator:
      channel: "stable"
    devspaces_ocp_install_instance:
      instance_name: devspaces
      storage_pvc_strategy: per-user

  roles:
    - devspaces_ocp_install
...
```

### Using API Token for Authentication

```yml
---
- name: Install Dev Spaces on OCP playbook with API Token
  hosts: localhost
  gather_facts: false

  vars:
    devspaces_ocp_install_connection:
      host: "https://api.example.com:6443"
      api_key: "{{ lookup('env', 'K8S_AUTH_API_KEY') }}"
      validate_certs: true
    devspaces_ocp_install_namespace: openshift-devspaces
    devspaces_ocp_install_operator:
      channel: "stable"
    devspaces_ocp_install_instance:
      instance_name: devspaces

  roles:
    - devspaces_ocp_install
...
```

### Installing Only the Operator

```yml
---
- name: Install only Dev Spaces operator
  hosts: localhost
  gather_facts: false

  vars:
    devspaces_ocp_install_connection:
      host: "https://api.example.com:6443"
      username: kubeadmin
      password:
      validate_certs: false
    devspaces_ocp_install_namespace: openshift-devspaces
    devspaces_ocp_install_operator:
      channel: "stable"

  roles:
    - devspaces_ocp_install
...
```

### Using Manifest Overrides

```yml
---
- name: Install Dev Spaces with custom configuration
  hosts: localhost
  gather_facts: false

  vars:
    devspaces_ocp_install_connection:
      host: "https://api.crc.testing:6443"
      username: kubeadmin
      password:
      validate_certs: false
    devspaces_ocp_install_namespace: openshift-devspaces
    devspaces_ocp_install_operator:
      channel: "stable"
    devspaces_ocp_install_instance:
      instance_name: devspaces
      instance_manifest_overrides:
        spec:
          components:
            cheServer:
              debug: true
              logLevel: DEBUG
            pluginRegistry:
              disableInternalRegistry: false

  roles:
    - devspaces_ocp_install
...
```

## Tags

This role supports the following tags for selective execution:

- `operator` - Install only the Dev Spaces operator
- `instance` - Install only the CheCluster instance (requires operator to be already installed)

Example usage:

```bash
# Install only the operator
ansible-playbook playbook.yml --tags operator

# Install only the instance
ansible-playbook playbook.yml --tags instance

# Install everything (default)
ansible-playbook playbook.yml
```

## License

[GPLv3+](LICENSE)

## Author Information

* Vinny Valdez

Based on the structure and patterns from the [aap_ocp_install](https://github.com/redhat-cop/aap_utilities/tree/devel/roles/aap_ocp_install) role by:
* Brant Evans
* Derek Waters
* Andrew Block
