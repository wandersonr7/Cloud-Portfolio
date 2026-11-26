# OmniRoute Cloud

Repository: [wandersonr7/OMNIROUTE-CLOUD](https://github.com/wandersonr7/OMNIROUTE-CLOUD) (MIT license)

## Problem

An application that calls one AI provider directly fails when that provider is down, slow or rate-limited. OmniRoute adds a reliability layer: the application sends one request, and OmniRoute picks a provider, retries and fails over.

## Architecture and tools

```text
Application -> OmniRoute -> OpenAI-compatible | Anthropic | Mock
```

- API: OpenAI-compatible `POST /v1/chat/completions`, `GET /health` and `GET /health/providers`. The `X-OmniRoute-Provider` header selects a provider.
- Reliability: provider timeouts, retries with exponential backoff, a circuit breaker per provider, health tracking and automatic failover.
- Protection: in-memory rate limiting, request IDs, structured JSON logging, centralized error handling.
- Tools: Python 3.12, FastAPI, pytest, Docker, Docker Compose, Terraform, GitHub Actions.

AWS target, defined in Terraform: ECR, ECS on Fargate, Application Load Balancer with health checks, VPC with public subnets, Internet Gateway and route tables, security groups, IAM roles, CloudWatch Logs and DynamoDB.

## Implementation

- `app/` holds the API, provider adapters, routing, circuit breaker, rate limiter and logging setup.
- Configuration comes from environment variables. `.env.example` lists them with empty key values.
- A free-tier route (`X-OmniRoute-Provider: free`) tries only free providers (Groq, Gemini, Cerebras, Mistral and OpenRouter `:free` models). It never fails over to a paid provider. With no free key configured it returns `503` so the caller decides what to do.

## Validation

- 21 pytest tests cover API behavior, routing, failover, retries, the circuit breaker and the free-tier route.
- GitHub Actions installs dependencies, compiles the app, runs pytest, builds the Docker image, and runs `terraform fmt -check`, `terraform init -backend=false` and `terraform validate`.
- Failover demo: a request that asks for OpenAI with no OpenAI key falls back to the mock provider and still returns a response.

## Security considerations

- No API key is needed for the default local demo.
- `.env` files and Terraform state are ignored by Git. A scan of all 21 commits found no keys or tokens.
- Commit metadata uses the GitHub no-reply address instead of a personal email.
- The roadmap moves production secrets to AWS Secrets Manager.

## Cost considerations

- The AWS environment is not deployed. Terraform defaults to `create_network = false`, `create_alb = false` and `create_ecs_service = false`.
- CI validates Terraform without running `terraform apply`, so a push cannot create billable resources.
- The OpenRouter adapter is skipped unless its model ends in `:free`, so the free-tier route cannot bill.

## Cleanup

Local: `docker compose down`. Cloud: nothing to remove, because nothing is created by default.

## Design decisions

- Keep a mock provider so the project can be cloned, tested and reviewed with no paid service.
- Validate infrastructure in CI but never apply it, so the AWS design can grow without cost.
- Fail over only within the same cost tier: a free request never reaches a paid provider.
