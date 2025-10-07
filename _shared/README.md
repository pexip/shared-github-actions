# Shared Action Modules

This directory contains shared JavaScript modules used by multiple composite actions to avoid code duplication.

## terraform-pr-comment.js

Shared module for posting Terraform plan results to pull request comments.

### Features

- **Single source of truth**: Eliminates code duplication across terraform-deploy-gcp and terraform-deploy-openstack
- **Reliable plan reading**: Reads from terraform.plan.txt file instead of unreliable stdout
- **Smart updates**: Only updates comments when content actually changes (MD5 hash comparison)
- **Size management**: Automatically truncates large plans with warnings
- **Security warnings**: Detects and warns about potential sensitive values
- **Destroy warnings**: Highlights when resources will be destroyed
- **Platform-specific branding**: Different emojis and labels for GCP vs OpenStack
- **Error handling**: Graceful failures that don't break the workflow

### Usage

```javascript
const prComment = require('${{ github.action_path }}/../_shared/terraform-pr-comment.js');

await prComment.createOrUpdatePRComment({
  github,
  context,
  core,
  directory: './terraform',
  platform: 'gcp', // or 'openstack'
  outcomes: {
    fmt: 'success',
    init: 'success',
    validate: 'success',
    trivy: 'success',
    plan: 'success'
  },
  validationOutput: 'Validation output here...'
});
```

### Maintenance

When updating this module, remember that changes will affect both:
- `.github/actions/terraform-deploy-gcp/action.yml`
- `.github/actions/terraform-deploy-openstack/action.yml`

Test changes with both platforms before committing.
