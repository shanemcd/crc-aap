# Advanced Configuration

Additional configuration options for operator installation and component settings.

## Operator Installation

Configure how the AAP operator is installed via OLM.

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_operator_channel` | `stable-{{ aap_version }}` | Operator channel (e.g., `stable-2.6`) |
| `aap_operator_source` | `redhat-operators` | Catalog source name |
| `aap_operator_source_namespace` | `openshift-marketplace` | Catalog source namespace |
| `aap_operator_package` | `ansible-automation-platform-operator` | Operator package name |
| `aap_operator_install_plan_approval` | `Manual` | Install plan approval mode |

### Example: Use a Custom Catalog

```bash
ansible-playbook deploy-aap.yml \
  -e aap_operator_source=my-catalog \
  -e aap_operator_source_namespace=my-namespace
```

## AAP Custom Resource

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_cr_name` | `myaap` | Name of the AnsibleAutomationPlatform CR |
| `aap_no_log` | `false` | Suppress sensitive output in operator logs |

## Hub Storage

Configure persistent storage for Automation Hub.

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_hub_file_storage_access_mode` | `ReadWriteOnce` | PVC access mode |
| `aap_hub_file_storage_size` | `20Gi` | PVC size |
| `aap_hub_file_storage_class` | (empty) | Storage class (uses cluster default if empty) |
| `aap_hub_storage_type` | `File` | Storage type |
| `aap_hub_content_replicas` | `1` | Number of content server replicas |

### Example: Use a Specific Storage Class

```bash
ansible-playbook deploy-aap.yml \
  -e aap_hub_file_storage_class=gp3-csi \
  -e aap_hub_file_storage_size=50Gi
```

## Lightspeed CA Bundle

For Lightspeed connections to LLM endpoints with self-signed certificates (e.g., internal vLLM):

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_lightspeed_ca_bundle_secret_name` | (empty) | Secret name for CA bundle |

When set, the playbook creates a secret containing the OpenShift service CA certificate. This is useful when connecting to internal services that use cluster-issued certificates.

```bash
ansible-playbook deploy-aap.yml \
  -e aap_lightspeed_disabled=false \
  -e aap_lightspeed_ca_bundle_secret_name=lightspeed-ca-bundle \
  -e @chatbot-vars.yml
```
