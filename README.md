# crc-aap

Deploy Ansible Automation Platform (AAP) on OpenShift Local (formerly CRC).

## Prerequisites

- OpenShift Local (`crc`) installed
- `ansible-core` with `kubernetes.core` collection

## Bootstrap CRC

```bash
crc start --cpus=12 --memory=40000 -d 100
```

## Quick Start

```bash
# Deploy AAP (installs operator automatically)
ansible-playbook deploy-aap.yml

# With Lightspeed chatbot enabled:
ansible-playbook deploy-aap.yml -e aap_lightspeed_disabled=false
```

### Optional: Configure Lightspeed Chatbot

```bash
cp chatbot-vars.yml.example chatbot-vars.yml
# Edit chatbot-vars.yml with your OpenAI API key
ansible-playbook create-chatbot-secret.yml -e @chatbot-vars.yml
ansible-playbook deploy-aap.yml -e aap_lightspeed_disabled=false
```

## Supported Versions

- AAP 2.6 (default)
- AAP 2.5

To deploy a specific version:
```bash
ansible-playbook install-operator.yml -e aap_version=2.5
ansible-playbook deploy-aap.yml -e aap_version=2.5
```

## Access

After deployment, access AAP at:
```
https://myaap-aap<version>.apps-crc.testing
```

Default admin credentials:
```bash
kubectl get secret myaap-admin-password -n aap<version> -o jsonpath='{.data.password}' | base64 -d
```

## Components

| Component | Default |
|-----------|---------|
| Controller | Enabled |
| EDA | Enabled |
| Hub | Enabled |
| Lightspeed | Disabled |

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

## Customization

Override defaults via extra vars:

```bash
ansible-playbook deploy-aap.yml \
  -e aap_hub_disabled=true \
  -e aap_eda_disabled=true \
  -e aap_lightspeed_disabled=false
```

See `roles/*/defaults/main.yml` for all options.

## Known Issues

### Lightspeed Chatbot 403 Error (AAP 2.6)

The Lightspeed chatbot returns a 403 Forbidden error due to CSRF origin validation. This affects OCP deployments in AAP 2.6.

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
