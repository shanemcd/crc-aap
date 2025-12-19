# Image Customization

Replace stock AAP images with custom builds for development and testing.

## Overview

Image overrides are integrated into the main playbooks. Simply pass the appropriate variables and the images will be automatically pushed to the internal OpenShift registry and configured.

All images end up at a predictable internal path:
```
image-registry.openshift-image-registry.svc:5000/<namespace>/<image-name>:override
```

## Prerequisites

- `oc` CLI logged in to cluster
- `podman` available
- `containers.podman` Ansible collection

```bash
ansible-galaxy collection install containers.podman
```

## Operator Images

Override operator images by passing variables to the install or deploy playbooks.

### Usage

```bash
# Override gateway operator during deploy
ansible-playbook deploy-aap.yml \
  -e aap_gateway_operator_image=localhost/gateway-operator:dev

# Override during operator-only install
ansible-playbook install-operator.yml \
  -e aap_gateway_operator_image=quay.io/myuser/gateway-operator:dev

# Multiple operators
ansible-playbook deploy-aap.yml \
  -e aap_controller_operator_image=localhost/controller-operator:dev \
  -e aap_hub_operator_image=localhost/hub-operator:dev

# Reset to stock images
ansible-playbook reset-operator-images.yml
```

### Supported Variables

| Variable | Operator |
|----------|----------|
| `aap_gateway_operator_image` | Gateway |
| `aap_controller_operator_image` | Controller |
| `aap_hub_operator_image` | Hub |
| `aap_eda_operator_image` | EDA |
| `aap_lightspeed_operator_image` | Lightspeed |
| `aap_resource_operator_image` | Resource |

### How It Works

1. Image is checked in local podman cache
2. Pulled from remote registry if not local
3. Pushed to internal OpenShift registry
4. Original image and ownerReferences saved to ConfigMap
5. Deployment detached from OLM (ownerReferences removed)
6. Deployment patched with internal registry image

The `reset-operator-images.yml` playbook reverses this process using the saved ConfigMap data.

## Application Images

Override application images (like the Lightspeed chatbot) by passing variables to the deploy playbook.

### Chatbot

```bash
ansible-playbook deploy-aap.yml \
  -e aap_lightspeed_disabled=false \
  -e chatbot_image=quay.io/myuser/lightspeed-chatbot:dev
```

To reset, remove the override fields from the AAP CR:

```bash
kubectl patch aap myaap -n aap26 --type=json \
  -p '[{"op":"remove","path":"/spec/lightspeed/chatbot_image"},{"op":"remove","path":"/spec/lightspeed/chatbot_image_version"}]'
```

## Troubleshooting

### Push fails with auth error

```bash
oc login ...
oc whoami -t  # Should return a token
```

### Image not found locally

The playbook will try to pull it. Verify you can pull manually:

```bash
podman pull quay.io/myuser/my-image:tag
```

### ImagePullBackOff after override

Check the RoleBinding exists:

```bash
oc get rolebinding -n aap26 | grep image-puller
```

## Technical Details

### Internal Registry

Images are pushed via the external route:
```
default-route-openshift-image-registry.apps-crc.testing/<namespace>/<name>:override
```

Resources reference the in-cluster URL:
```
image-registry.openshift-image-registry.svc:5000/<namespace>/<name>:override
```

### OLM Detachment

Operator deployments are detached from OLM by removing `ownerReferences`. This prevents OLM from reverting image changes. The reset playbook restores these references.

### Operator Container Names

| Operator | Container |
|----------|-----------|
| gateway | `manager` |
| controller | `automation-controller-manager` |
| hub | `automation-hub-operator` |
| eda | `eda-manager` |
| lightspeed | `ansible-lightspeed-manager` |
| resource | `platform-resource-manager` |
