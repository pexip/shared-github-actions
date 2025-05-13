# Setup zizmor action

## Prerequisites

None.

## Inputs

```yaml
- uses: pexip/shared-github-actions/setup-zizmor-action@master
  with:
    # (Optional) zizmor version. Defaults to "latest".
    version: "1.1.1"
    # (Optional) If "false" skips installation if zizmor is already installed.
    # If "true" installs zizmor in any case. Defaults to "false".
    force: "false"
```

## Outputs

<!-- prettier-ignore-start -->
| Name      | Description                         | Example |
|-----------|-------------------------------------|---------|
| installed | Whether zizmor was installed or not | `true`  |
<!-- prettier-ignore-end -->
