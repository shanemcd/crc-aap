# Lightspeed Chatbot Setup

Deploy AAP with Lightspeed chatbot enabled.

## Prerequisites

You need API credentials for one of the supported LLM providers:
- OpenAI
- Azure OpenAI
- Red Hat OpenShift AI vLLM

## Setup

### 1. Configure credentials

```bash
cp chatbot-vars.yml.example chatbot-vars.yml
```

Edit `chatbot-vars.yml` with your provider settings:

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

### 2. Test credentials (optional)

```bash
ansible-playbook test-chatbot-secret.yml -e @chatbot-vars.yml
```

This makes a test API call to verify your credentials work.

### 3. Create the chatbot secret

```bash
ansible-playbook create-chatbot-secret.yml -e @chatbot-vars.yml
```

### 4. Deploy AAP with Lightspeed

```bash
ansible-playbook deploy-aap.yml -e aap_lightspeed_disabled=false
```

## Known Issues

### 403 Forbidden Error (CSRF)

The Lightspeed chatbot may return a 403 Forbidden error due to CSRF origin validation in AAP 2.6.

**Workaround:** Set `CSRF_TRUSTED_ORIGINS`:

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

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `aap_lightspeed_disabled` | `true` | Set to `false` to enable |
| `aap_lightspeed_extra_settings` | `[]` | Additional Django settings |
| `chatbot_llm_provider_type` | - | `openai`, `azure_openai`, or `rhoai_vllm` |
| `chatbot_llm_provider_url` | - | LLM API endpoint |
| `chatbot_llm_provider_credentials` | - | API key or token |
| `chatbot_llm_provider_model` | - | Model name |
