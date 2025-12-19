# Operator Image Customization

Replace stock AAP operator images with custom builds for development and testing.

## Quick Start

```bash
# Override with a locally built image
ansible-playbook override-operator-images.yml \
  -e aap_gateway_operator_image=localhost/my-gateway-operator:dev

# Override with a remote image
ansible-playbook override-operator-images.yml \
  -e aap_gateway_operator_image=quay.io/myuser/gateway-operator:dev

# Reset to stock
ansible-playbook reset-operator-images.yml
```

## How It Works

1. Check if image exists in local podman cache
2. If not, pull it from the remote registry
3. Push to internal OpenShift registry
4. Patch deployment to use the internal image
5. Pod rolls out with new image

All images end up at:
```
image-registry.openshift-image-registry.svc:5000/<namespace>/<operator>-operator:override
```

## Supported Operators

| Variable | Operator |
|----------|----------|
| `aap_gateway_operator_image` | Gateway |
| `aap_controller_operator_image` | Controller |
| `aap_hub_operator_image` | Hub |
| `aap_eda_operator_image` | EDA |
| `aap_lightspeed_operator_image` | Lightspeed |
| `aap_resource_operator_image` | Resource |

## Examples

### Local build

```bash
# Build your operator
cd ~/src/gateway-operator
make docker-build IMG=localhost/gateway-operator:dev

# Deploy it
ansible-playbook override-operator-images.yml \
  -e aap_gateway_operator_image=localhost/gateway-operator:dev
```

### Remote image

```bash
ansible-playbook override-operator-images.yml \
  -e aap_gateway_operator_image=quay.io/myuser/gateway-operator:dev
```

### Multiple operators

```bash
ansible-playbook override-operator-images.yml \
  -e aap_controller_operator_image=localhost/controller-operator:dev \
  -e aap_hub_operator_image=localhost/hub-operator:dev
```

## What the Playbook Does

### override-operator-images.yml

1. Saves original image and ownerReferences to ConfigMap
2. Checks if image exists locally (podman)
3. Pulls image if not local
4. Pushes to internal registry using `oc whoami -t` for auth
5. Grants `system:image-puller` role to operator ServiceAccount
6. Removes ownerReferences (detaches from OLM)
7. Removes imagePullSecrets
8. Patches deployment with internal registry image
9. Waits for rollout

### reset-operator-images.yml

1. Reads original data from ConfigMap
2. Patches deployment with original image
3. Restores ownerReferences (re-attaches to OLM)
4. Deletes ConfigMap

## Prerequisites

- `oc` CLI logged in to cluster
- `podman` available
- `containers.podman` Ansible collection installed

```bash
ansible-galaxy collection install containers.podman
```

Note: The playbook uses `validate_certs: false` for the internal registry push since CRC uses a self-signed certificate.

## Troubleshooting

### Push fails with auth error

Make sure you're logged in:

```bash
oc login ...
oc whoami -t  # Should return a token
```

### Image not found locally

The playbook will try to pull it. Make sure you can pull manually:

```bash
podman pull quay.io/myuser/my-image:tag
```

### ImagePullBackOff after override

Check the RoleBinding exists:

```bash
oc get rolebinding -n aap26 | grep image-puller
```

## Technical Details

### Internal Registry Path

Images are pushed to:
```
default-route-openshift-image-registry.apps-crc.testing/<namespace>/<operator>-operator:override
```

Deployments reference the in-cluster URL:
```
image-registry.openshift-image-registry.svc:5000/<namespace>/<operator>-operator:override
```

### OLM Detachment

Deployments are detached from OLM by removing `ownerReferences`. This prevents OLM from reverting changes.

### Container Names

| Operator | Container |
|----------|-----------|
| gateway | `manager` |
| controller | `automation-controller-manager` |
| hub | `automation-hub-operator` |
| eda | `eda-manager` |
| lightspeed | `ansible-lightspeed-manager` |
| resource | `platform-resource-manager` |
