# Lightspeed Chatbot Setup

Deploy AAP with Lightspeed chatbot enabled.

## Prerequisites

API credentials for one of the supported LLM providers:
- OpenAI
- Azure OpenAI
- Red Hat OpenShift AI vLLM

## Quick Start

Create `chatbot-vars.yml` with your provider settings and deploy:

```bash
ansible-playbook deploy-aap.yml -e @chatbot-vars.yml -e aap_lightspeed_disabled=false
```

The chatbot configuration secret is created automatically.

## Provider Configuration

Example `chatbot-vars.yml`:

```yaml
# OpenAI
chatbot_llm_provider_type: openai
chatbot_llm_provider_url: https://api.openai.com/v1
chatbot_llm_provider_credentials: sk-...
chatbot_llm_provider_model: gpt-4

# Azure OpenAI
chatbot_llm_provider_type: azure_openai
chatbot_llm_provider_url: https://your-resource.openai.azure.com
chatbot_llm_provider_credentials: your-api-key
chatbot_llm_provider_model: gpt-4

# RHOAI vLLM
chatbot_llm_provider_type: rhoai_vllm
chatbot_llm_provider_url: https://your-vllm-endpoint
chatbot_llm_provider_credentials: your-token
chatbot_llm_provider_model: your-model
```

## Test Credentials (Optional)

```bash
ansible-playbook test-chatbot-secret.yml -e @chatbot-vars.yml
```

## Known Issues

### 403 Forbidden Error (CSRF)

The chatbot may return 403 due to CSRF origin validation.

**Workaround:** Add to your vars file:

```yaml
aap_lightspeed_extra_settings:
  - setting: CSRF_TRUSTED_ORIGINS
    value: "https://myaap-aap26.apps-crc.testing"
```

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_lightspeed_disabled` | `true` | Set to `false` to enable |
| `aap_lightspeed_extra_settings` | `[]` | Additional Django settings |
| `aap_lightspeed_ca_bundle_secret_name` | (empty) | Secret for CA bundle (self-signed certs) |
| `chatbot_llm_provider_type` | - | `openai`, `azure_openai`, or `rhoai_vllm` |
| `chatbot_llm_provider_url` | - | LLM API endpoint |
| `chatbot_llm_provider_credentials` | - | API key or token |
| `chatbot_llm_provider_model` | - | Model name |
