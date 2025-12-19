# Resource Requirements

Configure resource limits and requests for AAP components.

## Default Behavior

All components deploy with no resource limits (`{}`) for development use. Set any variable to `null` to use operator defaults instead.

## Variables

### Controller

| Variable | Description |
|----------|-------------|
| `aap_controller_web_resource_requirements` | Web container |
| `aap_controller_task_resource_requirements` | Task container |
| `aap_controller_ee_resource_requirements` | Execution environment |
| `aap_controller_postgres_resource_requirements` | PostgreSQL database |
| `aap_controller_redis_resource_requirements` | Redis cache |
| `aap_controller_rsyslog_resource_requirements` | Rsyslog |
| `aap_controller_init_container_resource_requirements` | Init containers |

### EDA

| Variable | Description |
|----------|-------------|
| `aap_eda_api_resource_requirements` | API server |
| `aap_eda_worker_resource_requirements` | Default worker |
| `aap_eda_activation_worker_resource_requirements` | Activation worker |
| `aap_eda_scheduler_resource_requirements` | Scheduler |
| `aap_eda_event_stream_resource_requirements` | Event stream |
| `aap_eda_ui_resource_requirements` | UI |
| `aap_eda_redis_resource_requirements` | Redis cache |
| `aap_eda_postgres_resource_requirements` | PostgreSQL database |

### Hub

| Variable | Description |
|----------|-------------|
| `aap_hub_api_resource_requirements` | API server |
| `aap_hub_content_resource_requirements` | Content server |
| `aap_hub_worker_resource_requirements` | Worker |
| `aap_hub_web_resource_requirements` | Web UI |
| `aap_hub_postgres_resource_requirements` | PostgreSQL database |
| `aap_hub_redis_resource_requirements` | Redis cache |

## Example

```yaml
aap_controller_web_resource_requirements:
  requests:
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

## Usage

```bash
ansible-playbook deploy-aap.yml -e @resource-requirements.yml
```
