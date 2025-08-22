# Apigee CI/CD Pipeline Repo

This repo supports:
- ✅ API Proxy & Shared Flow deployments
- ✅ Multi-environment (dev, test, stage, prod)
- ✅ Jenkins pipeline with apigeecli
- ✅ Linting using apigeelint

### Prerequisites
- Jenkins with NodeJS & Pipeline plugin
- GCP service account credential in Jenkins (`GCP_SA_JSON`)
- apigeecli and apigeelint tools

### Deployment Parameters
- DEPLOY_TYPE: apiproxy / sharedflow
- TARGET_ENV: dev / test / stage / prod
- API_NAME: name of proxy or shared flow
