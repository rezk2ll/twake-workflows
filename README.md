# twake-workflows

Reusable GitHub Actions workflows for Twake apps.

## build-and-publish-cozy-app.yml

Lints, tests, builds, runs BundleMon (when a `.bundlemonrc` exists) and publishes with `yarn cozyPublish` on `master` and version tags.

Apps must use Yarn 4.

`.github/workflows/build-and-publish-cozy-app.yml` in the app:

```yaml
name: Build and publish Cozy app

on:
  pull_request:
  push:
    branches:
      - master
    tags:
      - '[0-9]+.[0-9]+.[0-9]+'
      - '[0-9]+.[0-9]+.[0-9]+-beta.[0-9]+'

jobs:
  ci-cd:
    uses: linagora/twake-workflows/.github/workflows/build-and-publish-cozy-app.yml@v1
    with:
      # Only for apps publishing with --postpublish mattermost
      mattermost-channel: '{"dev":"my-app","beta":"my-app,publication","stable":"my-app,publication"}'
    secrets:
      REGISTRY_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
      DOWNCLOUD_SSH_KEY: ${{ secrets.DOWNCLOUD_SSH_KEY }}
      MATTERMOST_HOOK_URL: ${{ secrets.MATTERMOST_HOOK_URL }}
```

Apps in the `linagora` organization can use `secrets: inherit` instead: it does not work across organizations.

## docker-publish.yml

Builds an image with Buildx and pushes it. By default it tags `latest` on the default branch and the tag name on tags; `tags` takes any [docker/metadata-action](https://github.com/docker/metadata-action#tags-input) rules. The layer cache is kept per image, and a new push to a pull request cancels the run for the previous commit.

```yaml
name: Docker

on:
  pull_request:
  push:
    branches:
      - main
    tags:
      - 'v*'

permissions:
  contents: read
  packages: write # only for ghcr.io

jobs:
  docker:
    uses: linagora/twake-workflows/.github/workflows/docker-publish.yml@v1
    with:
      image: ghcr.io/linagora/my-app
      push: ${{ github.event_name != 'pull_request' }}
      # All optional
      context: server
      platforms: linux/amd64,linux/arm64
      tags: |
        type=raw,value=latest,enable={{is_default_branch}}
        type=match,pattern=server-(v.*),group=1
```

For another registry (Docker Hub, Harbor), pass the `REGISTRY_USERNAME` and `REGISTRY_PASSWORD` secrets.

## Releasing

Merging to `master` ships nothing until a release is published.

1. In Releases, draft a new release.
2. Type a new tag `vX.Y.Z` created on publish from `master`. Never pick the major tag itself: with immutable releases, it could never move again.
3. Generate the release notes and publish.

`update-major-tag.yml` then moves the major tag (`v1`) to the release.

Publish a new major version (`v2.0.0`) when apps have to change their workflow: renaming a workflow file, an input, a secret or a job (it renames the check apps may require), or adding a required input.

## Adding to this repository

- Reusable workflows go directly in `.github/workflows/`.
- Pin third-party actions by commit SHA with a `# vX` comment.
