# Pexip shared github-actions

GitHub Actions workflows and actions accessible to all Pexip workflows. This repository provides reusable composite actions for common CI/CD tasks including Docker builds, security scanning, Terraform deployments, and release automation.

## Table of Contents

- [Available Actions](#available-actions)
- [Quick Start](#quick-start)
- [Prerequisites](#prerequisites)
- [Examples](#examples)

## Available Actions

### Authentication

- **[auth-gcp](.github/actions/auth-gcp)** - Authenticate with Google Cloud Platform using service account key or workload identity federation
- **[auth-github](.github/actions/auth-github)** - Authenticate with GitHub Container Registry

### Docker

- **[docker-build](.github/actions/docker-build)** - Build and push Docker images with automatic tagging and metadata
- **[docker-security-scan](.github/actions/docker-security-scan)** - Security scan Docker images using Snyk

### Terraform

- **[terraform-deploy-gcp](.github/actions/terraform-deploy-gcp)** - Deploy infrastructure to GCP using Terraform (init, validate, plan, apply)
- **[terraform-deploy-openstack](.github/actions/terraform-deploy-openstack)** - Deploy infrastructure to OpenStack using Terraform

### Release

- **[release](.github/actions/release)** - Create GitHub releases with auto-generated notes and optional Jira integration

### Security Tools

- **[setup-zizmor-action](setup-zizmor-action)** - Install zizmor CLI tool for GitHub Actions security analysis

## Quick Start

### Using Actions in Your Workflow

Reference actions from this repository using the following pattern:

```yaml
uses: pexip/shared-github-actions/.github/actions/{action-name}@{ref}
```

### Example: Build and Push Docker Image

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - uses: pexip/shared-github-actions/.github/actions/auth-gcp@master
    with:
      repository: ${{ vars.DOCKER_REPO }}
      service_account_key: ${{ secrets.DEPLOY_SERVICE_ACCOUNT_KEY }}

  - uses: pexip/shared-github-actions/.github/actions/docker-build@master
    with:
      repository: ${{ vars.DOCKER_REPO }}
      image_name: my-application
      dockerfile: Dockerfile
```

### Example: Terraform Deployment

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - uses: pexip/shared-github-actions/.github/actions/auth-gcp@master
    with:
      repository: ${{ vars.DOCKER_REPO }}
      service_account_key: ${{ secrets.DEPLOY_SERVICE_ACCOUNT_KEY }}

  - uses: pexip/shared-github-actions/.github/actions/terraform-deploy-gcp@master
    with:
      directory: ./deploy
      token: ${{ secrets.GITHUB_TOKEN }}
```

### Example: Authenticate with Workload Identity Federation

Workload Identity Federation allows GitHub Actions to authenticate to GCP without using service account keys.

#### Prerequisites

1. **Create a Workload Identity Pool:**
   ```bash
   gcloud iam workload-identity-pools create "github-pool" \
     --project="${PROJECT_ID}" \
     --location="global" \
     --display-name="GitHub Actions Pool"
   ```

2. **Create a Workload Identity Provider:**
   ```bash
   gcloud iam workload-identity-pools providers create-oidc "github-provider" \
     --project="${PROJECT_ID}" \
     --location="global" \
     --workload-identity-pool="github-pool" \
     --display-name="GitHub provider" \
     --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner" \
     --attribute-condition="assertion.repository_owner == 'pexip'" \
     --issuer-uri="https://token.actions.githubusercontent.com"
   ```

3. **Create a Service Account:**
   ```bash
   gcloud iam service-accounts create "github-actions-sa" \
     --project="${PROJECT_ID}" \
     --display-name="GitHub Actions Service Account"
   ```

4. **Grant permissions to the Service Account:**
   ```bash
   # Example: Grant Artifact Registry and Cloud Run permissions
   gcloud projects add-iam-policy-binding ${PROJECT_ID} \
     --member="serviceAccount:github-actions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
     --role="roles/artifactregistry.writer"
   ```

5. **Allow the Workload Identity Pool to impersonate the Service Account:**
   ```bash
   gcloud iam service-accounts add-iam-policy-binding "github-actions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
     --project="${PROJECT_ID}" \
     --role="roles/iam.workloadIdentityUser" \
     --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/github-pool/attribute.repository/pexip/REPOSITORY_NAME"
   ```

6. **Get the Workload Identity Provider resource name:**
   ```bash
   gcloud iam workload-identity-pools providers describe "github-provider" \
     --project="${PROJECT_ID}" \
     --location="global" \
     --workload-identity-pool="github-pool" \
     --format="value(name)"
   ```
   Save this value as `WORKLOAD_IDENTITY_PROVIDER` variable in your repository.

#### Usage

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - uses: pexip/shared-github-actions/.github/actions/auth-gcp@master
    with:
      repository: ${{ vars.DOCKER_REPO }}
      workload_identity_provider: ${{ vars.WORKLOAD_IDENTITY_PROVIDER }}
      service_account: ${{ vars.SERVICE_ACCOUNT_EMAIL }}

  - uses: pexip/shared-github-actions/.github/actions/docker-build@master
    with:
      repository: ${{ vars.DOCKER_REPO }}
      image_name: my-application
      dockerfile: Dockerfile
```

### Example: Create a Release

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - uses: pexip/shared-github-actions/.github/actions/release@master
    with:
      version: v1.0.0
      github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Prerequisites

### Required Secrets

Configure these secrets in your repository settings:

- **`DEPLOY_SERVICE_ACCOUNT_KEY`** - GCP service account JSON key for authentication and Docker registry access
- **`SNYK_PEXIP_UNSORTED_ACCESS_TOKEN`** - Snyk API token for security scanning (if using docker-security-scan)
- **`GITHUB_TOKEN`** - Automatically provided by GitHub Actions

### Optional Secrets

- **`jira_webhook`** - Jira automation webhook URL for release integration

### Required Variables

Configure these variables in your repository settings:

- **`DOCKER_REPO`** - Docker repository URL (e.g., `europe-docker.pkg.dev/project-id/repo-name`)
- **`DOCKER_IMAGE`** - Docker image name
- **`DEPLOY_PROJECT_ID`** - GCP project ID for deployments

### Optional Variables (for Workload Identity Federation)

If using Workload Identity Federation instead of service account keys:

- **`WORKLOAD_IDENTITY_PROVIDER`** - Workload identity provider resource name (e.g., `projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/POOL_ID/providers/PROVIDER_ID`)
- **`SERVICE_ACCOUNT_EMAIL`** - Service account email to impersonate (e.g., `my-service-account@project-id.iam.gserviceaccount.com`)

## Examples

Complete workflow examples are located in the [examples](examples) folder:

- **[development.yml](examples/development.yml)** - Full development pipeline with Docker build, security scan, and Terraform deployment
- **[production.yml](examples/production.yml)** - Production deployment workflow triggered on main branch pushes or version tags
- **[release.yml](examples/release.yml)** - Release workflow with GitHub and Jira integration

These examples demonstrate common patterns for integrating multiple actions into complete CI/CD pipelines.
