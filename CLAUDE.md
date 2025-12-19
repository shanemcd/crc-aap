# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains Ansible playbooks and roles for deploying Ansible Automation Platform (AAP) on OpenShift Local (CRC). It uses the `kubernetes.core` collection to manage Kubernetes/OpenShift resources.

## Commands

### Deploy AAP
```bash
ansible-playbook deploy-aap.yml
```

### Deploy with specific version
```bash
ansible-playbook deploy-aap.yml -e aap_version=2.5
```

### Deploy with Lightspeed enabled
```bash
# First create the chatbot secret
ansible-playbook create-chatbot-secret.yml -e @chatbot-vars.yml

# Then deploy with Lightspeed
ansible-playbook deploy-aap.yml -e aap_lightspeed_disabled=false
```

### Test chatbot credentials
```bash
ansible-playbook test-chatbot-secret.yml -e @chatbot-vars.yml
```

## Architecture

The deployment uses Ansible roles executed in sequence:

1. **install_aap_operator** - Installs the AAP operator via OLM (Operator Lifecycle Manager)
   - Creates namespace (e.g., `aap26`)
   - Creates OperatorGroup and Subscription
   - Waits for ClusterServiceVersion to reach `Succeeded` phase

2. **override_operator_images** - Conditionally overrides operator images
   - Runs when any `aap_*_operator_image` variable is set
   - Pushes images to internal OpenShift registry
   - Detaches deployments from OLM and patches with custom image

3. **deploy_aap** - Deploys the AnsibleAutomationPlatform custom resource
   - Optionally verifies chatbot secret exists (when Lightspeed enabled)
   - Handles chatbot image override when `chatbot_image` is set
   - Creates the AAP CR with Controller, EDA, Hub, and Lightspeed components
   - Resource requirements default to empty `{}` (no limits) for development use

4. **reset_operator_images** - Resets operator images to stock (standalone playbook)
   - Restores original images from saved ConfigMap
   - Re-attaches deployments to OLM

## Key Variables

All configurable via `-e` flag:

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_version` | `2.6` | AAP version (derives namespace and operator channel) |
| `aap_namespace` | `aap26` | Target namespace (auto-derived from version) |
| `aap_controller_disabled` | `false` | Disable Controller component |
| `aap_eda_disabled` | `false` | Disable EDA component |
| `aap_hub_disabled` | `false` | Disable Hub component |
| `aap_lightspeed_disabled` | `true` | Disable Lightspeed (requires chatbot secret) |
| `kubeconfig` | omit | Path to kubeconfig file |
| `aap_*_operator_image` | - | Override operator image (gateway, controller, hub, eda, lightspeed, resource) |
| `chatbot_image` | - | Override Lightspeed chatbot image |

## Lightspeed Chatbot Providers

Supported `chatbot_llm_provider_type` values:
- `openai` - OpenAI API
- `azure_openai` - Azure OpenAI (uses `api-key` header)
- `rhoai_vllm` - Red Hat OpenShift AI vLLM
