# Open-SWE — Google Cloud Run Deployment

This guide documents the production deployment of the **Open-SWE** monorepo to Google Cloud project **`skygrade`** (region `us-central1`).

---
## 1 · Architecture

| Service  | Cloud Run Name | Container | Port | Max Instances |
|----------|----------------|-----------|------|---------------|
| Agent    | `agent`        | `agent`   | 2024 | 25 |
| Agent v2 | `agent-v2`     | `agent-v2`| 2025 | 25 |
| Web UI   | `web-ui`       | `web`     | 8080 | 10 |

Docs preview optional (`docs`).

---
## 2 · Secrets

Secrets are stored in **Secret Manager** as *env-file* style secrets:

| Secret Name             | Source file |
|-------------------------|-------------|
| `open-swe-agent-env`    | `apps/open-swe/.env` |
| `open-swe-agent-v2-env` | `apps/open-swe-v2/.env` |
| `open-swe-web-env`      | `apps/web/.env` |

Grant access:
```bash
gcloud secrets add-iam-policy-binding open-swe-agent-env \
  --member="serviceAccount:agent@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
# …repeat for each secret/service…
```

---
## 3 · Artifact Registry
```bash
gcloud artifacts repositories create us-central1-docker \
  --repository-format=docker \
  --location=us-central1
```

---
## 4 · GitHub Actions CI/CD

Workflow file: `.github/workflows/deploy-cloudrun.yml`

Requires these **repo secrets**:

| Secret | Description |
|--------|-------------|
| `WIF_PROVIDER` | Workload Identity Provider resource name |
| `DEPLOY_SA`    | Service-account email with Cloud Run & Artifact Registry perms |

On each push to `main` the workflow:
1. Authenticates to GCP via Workload Identity Federation
2. Builds multi-arch images for `agent`, `agent-v2`, `web`
3. Pushes them to Artifact Registry
4. Deploys/updates Cloud Run services with proper env/secret mounts

---
## 5 · Custom Domain

Map **`skygrade.ai`** sub-domains:

| Hostname              | Cloud Run service |
|-----------------------|-------------------|
| app.skygrade.ai       | `web-ui` |
| agent.skygrade.ai     | `agent`  |
| agent-v2.skygrade.ai  | `agent-v2` |

Cloud Run will supply domain-mapping `CNAME` targets (e.g., `ghs.googlehosted.com`). Create those records in your DNS provider.

---
## 6 · Local Build & Test
```bash
# build multi-arch images locally
docker buildx create --use

# Agent
docker buildx build -f apps/open-swe/Dockerfile -t open-swe-agent:dev .

# Run
docker run --env-file apps/open-swe/.env -p 2024:2024 open-swe-agent:dev
```

---
## 7 · Scaling & Costs
* CPU 2 / RAM 4 Gi per agent container → ~25 heavy prompts in parallel*
* Autoscale 0 → 25 per service; cost ~"request-based" on Cloud Run

---
## 8 · Troubleshooting
* `gcloud run services logs tail <svc> --project codezilla --region us-central1`
* Check secret mount errors in Cloud Run revisions.
