# shared-github-actions
Github-actions worflows and actions accessible to all Pexip workflows

## Examples

The examples are located in the '/examples' folder.

## Automatically generated release notes

The release workflow automatically generates release notes based on how pull requests are labeled.
The example configuration expects the following labels to be used:

* bug
* change
* feature

This is controlled through the '.github/release.yml' configuration file

```yaml
changelog:
  categories:
    - title: New
      labels:
        - '*'
      exclude:
        labels:
          - bug
          - change
    - title: Changes
      labels:
        - change
    - title: Bug Fixes
      labels:
        - bug
```


