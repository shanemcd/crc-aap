# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ansible playbooks and roles for deploying Ansible Automation Platform (AAP) on OpenShift Local (CRC). Uses `kubernetes.core` collection.

## Quick Reference

```bash
# Deploy AAP
ansible-playbook deploy-aap.yml

# Deploy with Lightspeed
ansible-playbook deploy-aap.yml -e @chatbot-vars.yml -e aap_lightspeed_disabled=false

# Deploy with custom images
ansible-playbook deploy-aap.yml -e @images.yml

# Reset operator images to stock
ansible-playbook reset-operator-images.yml
```

## Architecture

Roles executed in sequence by `deploy-aap.yml`:

1. **install_aap_operator** - Installs AAP operator via OLM
2. **override_operator_images** - Conditionally overrides operator images when `aap_*_operator_image` vars are set
3. **deploy_aap** - Creates AAP CR, handles chatbot secret creation and app image overrides

## Key Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_version` | `2.6` | AAP version (derives namespace) |
| `aap_namespace` | `aap26` | Target namespace |
| `aap_*_disabled` | varies | Disable components (controller, eda, hub, lightspeed) |
| `aap_*_operator_image` | - | Override operator images |
| `gateway_image`, `controller_image`, etc. | - | Override application images |
| `chatbot_llm_provider_*` | - | LLM provider config (auto-creates secret) |

## Documentation

For detailed information, see:

- @docs/lightspeed.md - Lightspeed chatbot setup and LLM provider configuration
- @docs/image-customization.md - Operator and application image customization
- @docs/resource-requirements.md - Per-component resource requirements
- @images.yml.example - Template for image override variables
