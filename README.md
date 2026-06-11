# Site Operations

BIAN Service Domain microservice — part of the [bian-platform](../../bian-platform/) landscape.

| | |
|---|---|
| **Business Area** | Business Support |
| **Business Domain** | Technology and Operations |
| **Functional Pattern** | Operate |
| **Asset Type** | Site Operation |
| **Control Record** | Site Operation Operating Session |
| **K8s Namespace** | `bian-business-support` |
| **Stack** | Java 21 · Spring Boot 3 · Resilience4j · Cilium mesh |

> ⚠️ **Phase 1 (shallow):** real REST API over an in-memory store. Phase 2 replaces the store with per-domain persistence and real domain logic. This repo was stamped from `bian-platform/generator` — regenerate rather than hand-editing boilerplate.

## BIAN Semantic API

| Method | Path | BIAN action term |
|---|---|---|
| GET | `/v1/service-domain` | — (SD metadata) |
| POST | `/v1/site-operation-operating-session/initiate` | Initiate |
| GET | `/v1/site-operation-operating-session` | Retrieve (list) |
| GET | `/v1/site-operation-operating-session/{crId}/retrieve` | Retrieve |
| PUT | `/v1/site-operation-operating-session/{crId}/update` | Update |
| PUT | `/v1/site-operation-operating-session/{crId}/control` | Control — body `{"action": "suspend"\|"resume"\|"terminate"}` |

OpenAPI UI: `/swagger-ui.html` · Health: `/actuator/health` · Metrics: `/actuator/prometheus`

**API contract:** [`api/openapi.yaml`](api/openapi.yaml) — owned by **this repo** (contract-per-repo; no central contracts repo). The runtime spec at `/v3/api-docs` must stay compatible with it; Phase 2 adds contract tests that enforce this.

## Run locally

```bash
mvn spring-boot:run
curl localhost:8080/v1/service-domain

# lifecycle smoke test
curl -X POST localhost:8080/v1/site-operation-operating-session/initiate -H 'content-type: application/json' -d '{"note":"hello"}'
```

## Build & containerize

```bash
mvn -B verify
docker build -t bian/sd-site-operations:0.1.0 .
```

## Deploy (Helm → K8s with Cilium mesh)

```bash
helm upgrade --install sd-site-operations ./helm -n bian-business-support
```

Exposed through the platform Gateway at path prefix `/sd-site-operations` (Cilium Gateway API). Mesh policy (`CiliumNetworkPolicy`) allows: gateway ingress, same-area peers, Prometheus — everything else denied.
