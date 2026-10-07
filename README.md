# UC4P2-Language-pilot-deployment-dev

Kubernetes manifests for the **UC4P2 Language Pilot** API on the DataGEMS dev cluster.

Mirrors the layout of [dg-query-disambiguation-deployment-dev](https://github.com/datagems-eosc/dg-query-disambiguation-deployment-dev).

## Manifests

| File | Resource |
|------|----------|
| `manifests/deployment.yaml` | API Deployment (`dg-uc4p2-language-pilot`) |
| `manifests/service.yaml` | ClusterIP Service (port 8080) |
| `manifests/configmap.yaml` | URLs, LLM, QDMR, and OIDC client env |
| `manifests/ingress.yaml` | Ingress at `datagems-dev.scayle.es/language-pilot` |
| `manifests/secret.yaml` | VaultStaticSecret → K8s Secret |

**Namespace:** `zhaw`

## Prerequisites

1. **Container image** published from [UC4P2-Language-pilot](https://github.com/datagems-eosc/UC4P2-Language-pilot):
   ```bash
   git tag v0.1.12 && git push origin v0.1.12
   ```
   Image: `ghcr.io/datagems-eosc/uc4p2-language-pilot:v0.1.12`

2. **Vault secrets** at path `zhaw/uc4p2-language-pilot` (KV v2):
   - `SCAYLE_USERNAME`
   - `SCAYLE_PASSWORD`
   - `DG_USERNAME`
   - `DG_PASSWORD`
   - `OPENAI_API_KEY` (optional)

3. **Cluster secrets** already present:
   - `ghcr-pull-secret`
   - `vault-auth-zhaw`

The API calls **query-disambiguation** (same namespace) and **Cross-Dataset Discovery** (public ingress).

## Deploy

```bash
kubectl apply -f manifests/
```

## Verify

```bash
kubectl -n zhaw get pods -l app=dg-uc4p2-language-pilot
kubectl -n zhaw port-forward svc/dg-uc4p2-language-pilot 8080:8080
curl http://localhost:8080/health
```

**Public URL** (after ingress):

```text
https://datagems-dev.scayle.es/language-pilot/health
https://datagems-dev.scayle.es/language-pilot/swagger
https://datagems-dev.scayle.es/language-pilot/tree
```

```bash
curl -s -X POST "https://datagems-dev.scayle.es/language-pilot/ThematicExploration" \
  -H "Content-Type: application/json" \
  -d '{"query": "How did a marriage look like in the 1800s compared to now?"}'
```

## Update image tag

Edit `manifests/deployment.yaml`:

```yaml
image: ghcr.io/datagems-eosc/uc4p2-language-pilot:v0.1.12
```

After changing the ConfigMap:

```bash
kubectl apply -f manifests/configmap.yaml
kubectl -n zhaw rollout restart deployment/dg-uc4p2-language-pilot
```
