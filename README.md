# crc-aap

Deploy Ansible Automation Platform (AAP) on OpenShift Local (formerly CRC).

## Prerequisites

- OpenShift Local (`crc`) installed
- `ansible-core` with `kubernetes.core` collection

## Bootstrap CRC

```bash
crc start --cpus=12 --memory=40000 -d 100
```

## Deployment

### Basic Deployment

```bash
ansible-playbook deploy-aap.yml
```

This installs the AAP operator and deploys AAP with Controller, EDA, and Hub enabled. Lightspeed is disabled by default.

### Deployment with Lightspeed

To deploy with Lightspeed chatbot, you must create the chatbot secret **before** running the deploy playbook.

**Step 1: Configure chatbot credentials**

```bash
cp chatbot-vars.yml.example chatbot-vars.yml
# Edit chatbot-vars.yml with your API credentials
```

**Step 2: Create the chatbot secret**

```bash
ansible-playbook create-chatbot-secret.yml -e @chatbot-vars.yml
```

**Step 3: Deploy AAP with Lightspeed enabled**

```bash
ansible-playbook deploy-aap.yml -e aap_lightspeed_disabled=false
```

## Access

After deployment, access AAP at:

```
https://myaap-aap<version>.apps-crc.testing
```

Retrieve the admin password:

```bash
kubectl get secret myaap-admin-password -n aap<version> -o jsonpath='{.data.password}' | base64 -d
```

## Supported Versions

| Version | Namespace | Default |
|---------|-----------|---------|
| AAP 2.6 | aap26     | Yes     |
| AAP 2.5 | aap25     | No      |

To deploy a specific version:

```bash
ansible-playbook deploy-aap.yml -e aap_version=2.5
```

## Components

| Component  | Default  |
|------------|----------|
| Controller | Enabled  |
| EDA        | Enabled  |
| Hub        | Enabled  |
| Lightspeed | Disabled |

Disable components as needed:

```bash
ansible-playbook deploy-aap.yml \
  -e aap_hub_disabled=true \
  -e aap_eda_disabled=true
```

## Customization

Use a custom kubeconfig:

```bash
ansible-playbook deploy-aap.yml -e kubeconfig=/path/to/kubeconfig
```

See `roles/*/defaults/main.yml` for all available options.

## Clean Redeployment

When redeploying, delete PVCs and secrets to avoid database password mismatches:

```bash
# Delete AAP CR
kubectl delete ansibleautomationplatform myaap -n aap26

# Delete PostgreSQL PVCs
kubectl delete pvc -n aap26 -l app.kubernetes.io/component=database

# Delete secrets (keep chatbot config)
kubectl get secrets -n aap26 -o name | \
  grep -v chatbot-configuration-secret | \
  xargs kubectl delete -n aap26

# Redeploy
ansible-playbook deploy-aap.yml
```

## Known Issues

### Lightspeed Chatbot 403 Error (AAP 2.6)

The Lightspeed chatbot returns a 403 Forbidden error due to CSRF origin validation.

**Workaround:** Set `CSRF_TRUSTED_ORIGINS` via `aap_lightspeed_extra_settings`:

```bash
ansible-playbook deploy-aap.yml \
  -e aap_lightspeed_disabled=false \
  -e '{"aap_lightspeed_extra_settings": [{"setting": "CSRF_TRUSTED_ORIGINS", "value": "https://myaap-aap26.apps-crc.testing"}]}'
```

Or in a vars file:

```yaml
aap_lightspeed_extra_settings:
  - setting: CSRF_TRUSTED_ORIGINS
    value: "https://myaap-aap26.apps-crc.testing"
```
