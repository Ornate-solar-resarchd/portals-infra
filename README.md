# portals-infra

Shared GitHub Actions workflows for unityess.cloud portals deployed on the Mac Mini.

## What's here

`.github/workflows/_deploy-template.yml` — reusable workflow that:

1. Fixes the launchd LaunchAgent's sparse `PATH`
2. Sets up an isolated `DOCKER_CONFIG` with no credential helper
3. Symlinks `docker-compose` and `docker-buildx` plugins from Docker Desktop
4. Shims `docker-credential-osxkeychain` to bypass the locked keychain
5. Pulls the latest code from `main`
6. Rebuilds only the specified services
7. Runs a health check (3 attempts, 5s apart, accepts 200/301/302)
8. Logs everything to `~/portals/logs/deploy/<portal>-<timestamp>.log`

## How to use it from a portal repo

Each portal repo's `.github/workflows/deploy.yml` becomes a thin wrapper:

```yaml
name: Deploy <Portal Name>

on:
  push:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: deploy-<portal>
  cancel-in-progress: false

jobs:
  deploy:
    uses: ResarchandDornate/portals-infra/.github/workflows/_deploy-template.yml@main
    with:
      portal:       <short-name>
      portal_dir:   <folder-under-~/portals/>
      services:     "service1 service2"
      health_url:   "https://<portal>.unityess.cloud/"
      runner_label: <runner-label>
```

## Portal matrix

| Portal        | portal_dir              | services                                | health_url                            | runner_label  |
|---------------|-------------------------|-----------------------------------------|---------------------------------------|---------------|
| compliance    | oem_compliance_portal   | compliance-backend compliance-frontend  | https://compliance.unityess.cloud/    | compliance    |
| marketing     | ornate_marketing_app    | marketing-api marketing-web             | https://marketing.unityess.cloud/     | marketing     |
| inverter      | inverter_monitoring_app | inverter-backend                        | https://inverter.unityess.cloud/      | inverter      |
| bess-api      | bess_backend_app        | bess-api bess-worker                    | https://bess.unityess.cloud/          | bess-api      |
| bess-frontend | ornate_bess_frontend    | bess-frontend                           | https://bess-app.unityess.cloud/      | bess-frontend |
| master-portal | master-portal-v2        | master-portal master-portal-api         | https://portal.unityess.cloud/        | master-portal |
| procurement   | ornate_procurement      | procurement-api procurement-web         | https://procurement.unityess.cloud/   | procurement   |
| bess-portal   | unityess-bess-portal    | bess-portal-api bess-portal-web         | https://bess-portal.unityess.cloud/   | bess-portal   |

## Updating deploy logic

Edit `.github/workflows/_deploy-template.yml`, commit, push to `main`. The change applies to every portal's next deploy automatically — no need to update each portal repo.
