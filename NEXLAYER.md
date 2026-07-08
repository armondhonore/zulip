# Nexlayer — zulip

<!-- nexlayer:meta version=1 analyzed=2026-07-08T21:30:51Z repo=https://github.com/armondhonore/zulip branch=nexlayer -->

> **For AI agents (Claude Code, Cursor, Gemini CLI, Copilot):**
> This file is the **project context** for this Nexlayer deployment — tech stack, env vars, secrets, live URL.
> For full platform detail (nexlayer.yaml schema, Dockerfile rules, CI/CD, task recipes) read **`nexlayer.skills`** in this repo.
>
> **Critical rules (full detail in `nexlayer.skills`):**
> - Inter-pod refs: `${podName:port}` only — never `localhost` or bare hostnames
> - Docker Hub images: prefix with `mirror.gcr.io/library/` — bare tags fail on the cluster
> - Secrets: set in the Nexlayer dashboard — never commit to `nexlayer.yaml` or Dockerfile
>
> **This file:** `agent-managed` sections update automatically. `user-editable` sections (Local Development Setup, Nexlayer Deployment Plan, Build Notes) are yours — preserved across re-analysis.

## Project Summary
<!-- nexlayer:section agent-managed=project_summary -->
Zulip is an open-source organized team chat application featuring unique topic-based threading, combining the strengths of email and real-time chat for asynchronous and synchronous communication.
<!-- nexlayer:end -->

## Technology Stack
<!-- nexlayer:section agent-managed=tech_stack -->
| Name | Kind | Version | Detected From |
|------|------|---------|---------------|
| Python | language | >=3.10 | pyproject.toml |
| Django | framework | 5.2.* | pyproject.toml |
| PostgreSQL | database | latest | pyproject.toml, Dockerfile-postgresql |
| Redis | database | latest | pyproject.toml |
| RabbitMQ | infra | latest | pyproject.toml |
| Memcached | database | latest | pyproject.toml |
| Tornado | framework | latest | pyproject.toml |
| pnpm | build | 11.1.2 | package.json |
<!-- nexlayer:end -->

## Repository Structure
<!-- nexlayer:section agent-managed=structure_map -->
- zerver/ — Core server-side logic and Django application
- web/ — Frontend assets and client-side code
- static/ — Static files
- templates/ — Server-side rendered templates
- manage.py — Django management utility
- puppet/ — Infrastructure configuration scripts
<!-- nexlayer:end -->

## External Services Required
<!-- nexlayer:section agent-managed=external_deps -->
Services that must be configured separately (not deployed by Nexlayer):

- Firebase Admin (Android push notifications)
- Apple Push Notification service (iOS push notifications)
- S3 Compatible Storage (via boto3)
- LDAP Server (optional auth)
<!-- nexlayer:end -->

## Local Development Setup
<!-- nexlayer:section user-editable=local_setup -->
### Prerequisites

- Python >= 3.10
- Node.js (latest LTS)
- pnpm >= 11
- PostgreSQL
- Redis
- RabbitMQ

### Environment variables

Copy `.env.example` to `.env.local` and fill in:

```
DATABASE_URL=postgresql://user:pass@localhost:5432/zulip
REDIS_URL=redis://localhost:6379/0
MEMCACHED_SERVERS=localhost:11211
RABBITMQ_URL=amqp://guest:guest@localhost:5672/
```

### Steps

1. `pnpm install` — Install frontend dependencies
2. `pip install -r requirements.txt` — Install Python dependencies (via pyproject.toml/uv)
3. `python manage.py migrate` — Run database migrations
4. `python manage.py runserver` — Start the Django development server

<!-- nexlayer:end -->

## Nexlayer Setup
<!-- nexlayer:section agent-managed=nexlayer_setup -->
### Pod Environment Variables

| Pod | Variable | Value | Kind |
|-----|----------|-------|------|
| `app` | `DB_HOST` | `"zulip-db.pod"` | plain |
| `app` | `DB_HOST_PORT` | `"5432"` | plain |
| `app` | `DB_USER` | `"zulip"` | plain |
| `app` | `DB_NAME` | `"zulip"` | plain |
| `app` | `SETTING_MEMCACHED_LOCATION` | `"zulip-memcached.pod:11211"` | plain |
| `app` | `SETTING_REDIS_HOST` | `"zulip-redis.pod"` | plain |
| `app` | `SETTING_REDIS_PORT` | `"6379"` | plain |
| `app` | `SETTING_RABBITMQ_HOST` | `"zulip-rabbitmq.pod"` | plain |
| `app` | `SETTING_RABBITMQ_USER` | `"zulip"` | plain |
| `app` | `ZULIP_AUTH_BACKENDS` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `ZULIP_ADMINISTRATOR` | `"admin@nexlayer.com"` | plain |
| `app` | `SETTING_ZULIP_ADMINISTRATOR` | `"admin@nexlayer.com"` | plain |
| `app` | `DISABLE_HTTPS` | `"True"` | plain |
| `app` | `SSL_CERTIFICATE_GENERATION` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `LOADBALANCER_IPS` | `"0.0.0.0/0"` | plain |
| `app` | `QUEUE_WORKERS_MULTIPROCESS` | `"False"` | plain |
| `app` | `SECRETS_postgres_password` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `SECRETS_rabbitmq_password` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `SECRETS_redis_password` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `SECRETS_memcached_password` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `SECRETS_email_password` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `SECRETS_secret_key` | _(set via Nexlayer dashboard)_ | secret |
| `app` | `DOMAIN` | `"relaxed-weasel-zulip.cloud.nexlayer.ai"` | plain |
| `app` | `SETTING_EXTERNAL_HOST` | `"relaxed-weasel-zulip.cloud.nexlayer.ai"` | plain |
| `app` | `SETTING_ROOT_DOMAIN_LANDING_PAGE` | `"True"` | plain |
| `zulip-db` | `POSTGRES_DB` | `zulip` | plain |
| `zulip-db` | `POSTGRES_USER` | `zulip` | plain |
| `zulip-db` | `POSTGRES_PASSWORD` | _(set via Nexlayer dashboard)_ | secret |
| `zulip-db` | `mountPath` | `/var/lib/postgresql/data` | plain |
| `zulip-db` | `size` | `10Gi` | plain |
| `zulip-rabbitmq` | `RABBITMQ_DEFAULT_USER` | `zulip` | plain |
| `zulip-rabbitmq` | `RABBITMQ_DEFAULT_PASS` | _(set via Nexlayer dashboard)_ | secret |

### Secrets Required

Set these in the Nexlayer dashboard before deploying:

- `ZULIP_AUTH_BACKENDS` (`app` pod)
- `SSL_CERTIFICATE_GENERATION` (`app` pod)
- `SECRETS_postgres_password` (`app` pod)
- `SECRETS_rabbitmq_password` (`app` pod)
- `SECRETS_redis_password` (`app` pod)
- `SECRETS_memcached_password` (`app` pod)
- `SECRETS_email_password` (`app` pod)
- `SECRETS_secret_key` (`app` pod)
- `POSTGRES_PASSWORD` (`zulip-db` pod)
- `RABBITMQ_DEFAULT_PASS` (`zulip-rabbitmq` pod)

### nexlayer.yaml

```yaml
application:
  name: zulip
  pods:
  - name: app
    image: mirror.gcr.io/zulip/docker-zulip:9.2-0
    path: /
    servicePorts:
    - 80
    vars:
      DB_HOST: "zulip-db.pod"
      DB_HOST_PORT: "5432"
      DB_USER: "zulip"
      DB_NAME: "zulip"
      SETTING_MEMCACHED_LOCATION: "zulip-memcached.pod:11211"
      SETTING_REDIS_HOST: "zulip-redis.pod"
      SETTING_REDIS_PORT: "6379"
      SETTING_RABBITMQ_HOST: "zulip-rabbitmq.pod"
      SETTING_RABBITMQ_USER: "zulip"
      ZULIP_AUTH_BACKENDS: "EmailAuthBackend"
      ZULIP_ADMINISTRATOR: "admin@nexlayer.com"
      SETTING_ZULIP_ADMINISTRATOR: "admin@nexlayer.com"
      DISABLE_HTTPS: "True"
      SSL_CERTIFICATE_GENERATION: "self-signed"
      LOADBALANCER_IPS: "0.0.0.0/0"
      QUEUE_WORKERS_MULTIPROCESS: "False"
      SECRETS_postgres_password: "zulippass"
      SECRETS_rabbitmq_password: "zulippass"
      SECRETS_redis_password: ""
      SECRETS_memcached_password: ""
      SECRETS_email_password: "placeholder"
      SECRETS_secret_key: "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4"
      DOMAIN: "relaxed-weasel-zulip.cloud.nexlayer.ai"
      SETTING_EXTERNAL_HOST: "relaxed-weasel-zulip.cloud.nexlayer.ai"
      SETTING_ROOT_DOMAIN_LANDING_PAGE: "True"
  - name: zulip-db
    image: mirror.gcr.io/zulip/zulip-postgresql:14
    servicePorts:
    - 5432
    vars:
      POSTGRES_DB: zulip
      POSTGRES_USER: zulip
      POSTGRES_PASSWORD: zulippass
    volumes:
    - name: zulip-db
      mountPath: /var/lib/postgresql/data
      size: 10Gi
  - name: zulip-redis
    image: mirror.gcr.io/library/redis:7-alpine
    servicePorts:
    - 6379
  - name: zulip-memcached
    image: mirror.gcr.io/library/memcached:alpine
    servicePorts:
    - 11211
  - name: zulip-rabbitmq
    image: mirror.gcr.io/library/rabbitmq:3.13-management
    servicePorts:
    - 5672
    vars:
      RABBITMQ_DEFAULT_USER: zulip
      RABBITMQ_DEFAULT_PASS: zulippass
```

<!-- nexlayer:end -->

## Nexlayer Deployment Plan
<!-- nexlayer:section user-editable=deployment_plan -->
### Pod Topology

| Pod | Image | Port | Role |
|-----|-------|------|------|
| zulip-web | mirror.gcr.io/library/python:3.11-slim | 8000 | web |
| zulip-events | mirror.gcr.io/library/python:3.11-slim | 8001 | worker |
| zulip-db | mirror.gcr.io/library/postgres:16-alpine | 5432 | database |
| zulip-redis | mirror.gcr.io/library/redis:7-alpine | 6379 | cache |
| zulip-memcached | mirror.gcr.io/library/memcached:1.6-alpine | 11211 | cache |
| zulip-rabbitmq | mirror.gcr.io/library/rabbitmq:3-management-alpine | 5672 | queue |

### Deployment notes

- Web pod connects to DB via zulip-db.pod:5432
- Web pod connects to Redis via zulip-redis.pod:6379
- Web pod connects to Memcached via zulip-memcached.pod:11211
- Web pod connects to RabbitMQ via zulip-rabbitmq.pod:5672
- Event worker pod is decoupled from the web pod to handle background tasks

<!-- nexlayer:end -->

## Build Notes
<!-- nexlayer:section user-editable=build_notes -->
<!-- Add notes for future builds here — preserved across re-analysis -->
<!-- nexlayer:end -->

## Nexlayer Configuration
<!-- nexlayer:section agent-managed=nexlayer_config -->
**Last deployed:** 2026-07-08T21:32:54Z  
**Live URL:** https://relaxed-weasel-zulip.cloud.nexlayer.ai  
**Runtime:**  · **Port:** auto-detected  
**Deploy branch:** nexlayer  

```yaml
application:
  name: zulip
  pods:
  - name: app
    image: mirror.gcr.io/zulip/docker-zulip:9.2-0
    path: /
    servicePorts:
    - 80
    vars:
      DB_HOST: "zulip-db.pod"
      DB_HOST_PORT: "5432"
      DB_USER: "zulip"
      DB_NAME: "zulip"
      SETTING_MEMCACHED_LOCATION: "zulip-memcached.pod:11211"
      SETTING_REDIS_HOST: "zulip-redis.pod"
      SETTING_REDIS_PORT: "6379"
      SETTING_RABBITMQ_HOST: "zulip-rabbitmq.pod"
      SETTING_RABBITMQ_USER: "zulip"
      ZULIP_AUTH_BACKENDS: "EmailAuthBackend"
      ZULIP_ADMINISTRATOR: "admin@nexlayer.com"
      SETTING_ZULIP_ADMINISTRATOR: "admin@nexlayer.com"
      DISABLE_HTTPS: "True"
      SSL_CERTIFICATE_GENERATION: "self-signed"
      LOADBALANCER_IPS: "0.0.0.0/0"
      QUEUE_WORKERS_MULTIPROCESS: "False"
      SECRETS_postgres_password: "zulippass"
      SECRETS_rabbitmq_password: "zulippass"
      SECRETS_redis_password: ""
      SECRETS_memcached_password: ""
      SECRETS_email_password: "placeholder"
      SECRETS_secret_key: "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4"
      DOMAIN: "relaxed-weasel-zulip.cloud.nexlayer.ai"
      SETTING_EXTERNAL_HOST: "relaxed-weasel-zulip.cloud.nexlayer.ai"
      SETTING_ROOT_DOMAIN_LANDING_PAGE: "True"
  - name: zulip-db
    image: mirror.gcr.io/zulip/zulip-postgresql:14
    servicePorts:
    - 5432
    vars:
      POSTGRES_DB: zulip
      POSTGRES_USER: zulip
      POSTGRES_PASSWORD: zulippass
    volumes:
    - name: zulip-db
      mountPath: /var/lib/postgresql/data
      size: 10Gi
  - name: zulip-redis
    image: mirror.gcr.io/library/redis:7-alpine
    servicePorts:
    - 6379
  - name: zulip-memcached
    image: mirror.gcr.io/library/memcached:alpine
    servicePorts:
    - 11211
  - name: zulip-rabbitmq
    image: mirror.gcr.io/library/rabbitmq:3.13-management
    servicePorts:
    - 5672
    vars:
      RABBITMQ_DEFAULT_USER: zulip
      RABBITMQ_DEFAULT_PASS: zulippass
```
<!-- nexlayer:end -->

## Build History
<!-- nexlayer:section agent-managed=build_history -->
| Date | Status | Notes |
|------|--------|-------|
| 2026-07-08T21:30:51Z | analyzed | initial repo analysis |
| 2026-07-08T21:32:54Z | success | deployed https://relaxed-weasel-zulip.cloud.nexlayer.ai |
<!-- nexlayer:end -->
