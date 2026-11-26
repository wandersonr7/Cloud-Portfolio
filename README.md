# Wanderson Souza: Cloud and Infrastructure Portfolio

Projects in cloud operations, containers, CI/CD and infrastructure as code. Each project runs locally with Docker, is validated by GitHub Actions on every push, and documents its security and cost decisions.

## Featured projects

| Project | What it proves | Stack | Evidence |
| --- | --- | --- | --- |
| [Cloud Operations Lab](projects/cloud-operations-lab.md) | Diagnosing failures layer by layer and proving recovery | Docker, Compose, Flask, Bash, GitHub Actions | 9 incidents with root-cause reports; CI breaks, detects, fixes and re-verifies all 9 on every push |
| [OmniRoute Cloud](projects/omniroute-cloud.md) | Building a reliability layer in front of external APIs | Python, FastAPI, Docker, Terraform, GitHub Actions | 21 automated tests; CI runs pytest, a Docker build and `terraform validate` |

### Cloud Operations Lab

A Flask service in Docker that is broken on purpose. Each of the 9 incidents (wrong port mapping, failing health check, restart loop, CPU, memory, disk, DNS, internal networking, exposure on all interfaces) has a report covering symptoms, investigation, root cause, fix, verification and prevention.

`bash chaos/lab.sh drill 07` injects a DNS failure, confirms it is detected, applies the fix and checks reachability, Docker health, loopback-only binding, CPU, memory, log size and DNS before it passes.

Repository: [wandersonr7/CLOUD-OPERATIONS-LAB](https://github.com/wandersonr7/CLOUD-OPERATIONS-LAB)

### OmniRoute Cloud

An OpenAI-compatible API gateway that sits between an application and several AI providers. It adds timeouts, retries with exponential backoff, a circuit breaker per provider, health tracking and automatic failover. A local mock provider lets anyone run and test it without an API key or a cloud account.

The AWS target (ECR, ECS on Fargate, ALB, VPC, IAM, CloudWatch Logs, DynamoDB) is defined in Terraform and validated in CI. It is not deployed: resource creation is off by default and CI never runs `terraform apply`.

Repository: [wandersonr7/OMNIROUTE-CLOUD](https://github.com/wandersonr7/OMNIROUTE-CLOUD)

## Skills and where to find them

| Skill | Shown in |
| --- | --- |
| Linux process, memory, disk and `/proc` inspection | Cloud Operations Lab, incidents 03 to 06 |
| Docker networking, DNS and port publishing | Cloud Operations Lab, incidents 01, 07, 08 and 09 |
| Health checks and restart loops | Cloud Operations Lab, incidents 02 and 03 |
| Incident response and root-cause analysis | Cloud Operations Lab, all 9 reports |
| CI/CD with GitHub Actions | Both projects |
| Python and REST APIs (FastAPI, Flask) | OmniRoute Cloud, Cloud Operations Lab |
| Reliability patterns: retries, backoff, circuit breaker, failover | OmniRoute Cloud |
| Infrastructure as code (Terraform, AWS architecture) | OmniRoute Cloud |
| Secure defaults: loopback binding, no secrets in Git, no billable resources by default | Both projects |

## How projects are documented

Each project page covers:

1. The problem
2. Architecture and tools
3. Implementation
4. Validation and test results
5. Security and cost considerations
6. Cleanup
7. Lessons learned or design decisions

The [lab report template](templates/lab-report-template.md) holds the same structure for smaller exercises in [`labs/`](labs/README.md).

## Repository structure

| Area | Purpose |
| --- | --- |
| `projects/` | One case-study page per featured project |
| `labs/` | Small reproducible exercises |
| `docs/` | Learning roadmap |
| `templates/` | Documentation templates |

## Security

Credentials, access keys, private keys, tokens, account IDs and personal data are never committed. See [SECURITY.md](SECURITY.md).

## Contact

- GitHub: [@wandersonr7](https://github.com/wandersonr7)
