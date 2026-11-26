# Cloud Operations Lab

Repository: [wandersonr7/CLOUD-OPERATIONS-LAB](https://github.com/wandersonr7/CLOUD-OPERATIONS-LAB) (MIT license)

## Problem

Most infrastructure projects show how to deploy a service. This one covers what happens after: the service stops answering, Docker reports the wrong state, or the application stays up while the host underneath is degraded.

## Architecture and tools

A small Flask application (`GET /` and `GET /health`) runs in Docker with a 256 MiB memory limit and a Docker health check. The port is published on `127.0.0.1:8080` only.

```text
Client -> 127.0.0.1:8080 -> Docker published port -> container :8080 -> Flask
```

Tools: Python 3.12, Flask, Docker, Docker Compose, Bash, Make, GitHub Actions.

## Implementation

Nine incidents, each injected in a controlled way:

| # | Incident | How it is injected |
| --- | --- | --- |
| 01 | Service unreachable | Compose override maps host `8080` to container `8081` |
| 02 | Health check failure | Health check pointed at port `9999` |
| 03 | Restart loop | Entrypoint set to a file that does not exist |
| 04 | High CPU | Busy-loop Python process in the container |
| 05 | Memory pressure | Process allocating 180 MiB under the 256 MiB limit |
| 06 | Disk and log growth | 64 MiB log file written to `/var/log` |
| 07 | DNS failure | `/etc/resolv.conf` pointed at an unreachable nameserver |
| 08 | Internal network failure | Dependency container whose HTTP process is killed |
| 09 | Exposed on all interfaces | Port published on `0.0.0.0` |

`chaos/lab.sh` has `up`, `break`, `detect`, `fix`, `verify` and `drill` commands. A drill runs the whole cycle for one incident.

## Validation

- `verify` checks host reachability, Docker health, loopback-only binding, CPU below 50%, memory below 150 MiB, `/var/log` below 50 MiB and DNS resolution.
- A drill passes only when the failure is detected and every signal returns to baseline.
- GitHub Actions builds the image, starts it with the memory limit and checks `/health`. A second job replays all 9 incidents in parallel, one job per incident.

Measured during the incidents (from the reports):

| Incident | During | After the fix |
| --- | --- | --- |
| 04 High CPU | about 99.82% CPU | 0.01% CPU |
| 05 Memory | 213.3 MiB of 256 MiB (83.31%) | 28.97 MiB (11.32%) |
| 06 Disk | `/var/log` at 65M | `/var/log` at 212K |

## Security considerations

- Incident 09 came from a security review, not an outage: every health signal was green while the service listened on `0.0.0.0` and `[::]`. The fix binds the port to `127.0.0.1`, and `verify` now checks it on every run.
- The memory limit keeps memory experiments from affecting the host.

## Cost considerations

Everything runs in local Docker and on the free GitHub Actions tier for public repositories. No cloud resources are created.

## Cleanup

`docker compose down` stops and removes the lab. The temporary dependency container from incident 08 is removed by the drill.

## Lessons learned

The reports record signals that must be checked separately:

```text
Application health != Container health
Container running   != Service listening
Health endpoint ok  != CPU, memory or disk normal
DNS failure         != Network outage
Health check ok     != Safely exposed
```
