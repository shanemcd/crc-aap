# crc-aap

Deploy Ansible Automation Platform (AAP) on OpenShift Local (CRC).

## Prerequisites

- OpenShift Local (`crc`) running
- `ansible-core` with `kubernetes.core` collection

```bash
crc start --cpus=12 --memory=40000 -d 100
```

## Quick Start

```bash
ansible-playbook deploy-aap.yml
```

This installs the AAP operator and deploys Controller, EDA, and Hub.

## Access

```bash
# URL
echo "https://myaap-aap26.apps-crc.testing"

# Admin password
kubectl get secret myaap-admin-password -n aap26 -o jsonpath='{.data.password}' | base64 -d
```

## Options

### Version

```bash
ansible-playbook deploy-aap.yml -e aap_version=2.5
```

| Version | Namespace | Default |
|---------|-----------|---------|
| 2.6     | aap26     | Yes     |
| 2.5     | aap25     | No      |

### Components

```bash
ansible-playbook deploy-aap.yml \
  -e aap_hub_disabled=true \
  -e aap_eda_disabled=true
```

| Component  | Default  |
|------------|----------|
| Controller | Enabled  |
| EDA        | Enabled  |
| Hub        | Enabled  |
| Lightspeed | Disabled |

### Kubeconfig

```bash
ansible-playbook deploy-aap.yml -e kubeconfig=/path/to/kubeconfig
```

## Documentation

- [Lightspeed Chatbot Setup](docs/lightspeed.md)
- [Operator Image Customization](docs/operator-image-customization.md)

## Clean Redeployment

```bash
# Delete AAP CR
kubectl delete ansibleautomationplatform myaap -n aap26

# Delete PVCs and secrets
kubectl delete pvc -n aap26 -l app.kubernetes.io/component=database
kubectl get secrets -n aap26 -o name | grep -v chatbot | xargs kubectl delete -n aap26

# Redeploy
ansible-playbook deploy-aap.yml
```
