# gha-ld-gosec
This action runs the [gosec static code analyzer](https://github.com/securego/gosec) and then uploads the results to s3 and workflow artifacts. This is a required action for all launchdarkly critical repositories.

## Usage
Add the following file to your repository:
<h5 a><strong><code>.github/workflows/gosec.yml</code></strong></h5>

```yaml
name: Gosec
on:
  schedule:
    - cron: '0 8 * * *'
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  Gosec:
    runs-on: ubuntu-latest
    env:
      GO111MODULE: on
    steps:
      - uses: launchdarkly/gha-ld-gosec@v1
        with:
          aws-assume-role: ${{ secrets.ORG_SECURITY_GHA_ROLE_ARN }}

```

## Inputs
| name                | required | type   | default         | description |
| ------------------- | ---      | ------ | --------------- | ----------- |
| aws-assume-role     | yes      | string |                 | The ARN of an AWS IAM role to assume. Used to auth with AWS to upload results to S3. |
| gosec-args          | no       | string | `'--exclude-generated=true --severity=medium --concurrency=1 --fmt json --out=gosec-results.json --stdout --verbose=text --no-fail ./...'` | Override the arguments passed to the gosec command. |

## Release tags
This action maintains parallel tags for major (v1) and minor (v1.0.0) versions. This allows us to reference the major tag in github workflows and pick up minor changes automatically. While still maintaining minor versioning in the event that a repository needs to pin to a specific version for some reason.

To publish a new version `v1.0.1` we do the following after merging changes into the main branch:
```bash
# Checkout main branch and pull in latest changes and tags
git checkout main
git pull

# Create the minor version tag and push it to the remote repository
git tag v1.0.1
git push origin v1.0.1

# Delete and recreate the major version tag then delete and push it to the remote repository
git tag -d v1 && git tag v1
git push origin :v1 && git push origin v1
```
