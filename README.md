# mattermost-arm64build

[![mattermost-arm64](https://github.com/vtolj/mattermost-arm64build/actions/workflows/mattermost-arm64.yml/badge.svg)](https://github.com/vtolj/mattermost-arm64build/actions/workflows/mattermost-arm64.yml)

Builds and publishes an ARM64 Docker image for Mattermost Team Edition to GHCR
## Why this exists

Mattermost's official Docker images are commonly published for `linux/amd64` only, which leaves ARM64 users (Apple Silicon, Raspberry Pi, ARM servers) without a ready-to-use image. This workflow fills that gap by producing a compatible ARM64 image from the official release artifacts and publishing it to GHCR
## What it does

- Finds the latest Mattermost release (or uses a manually supplied version)- Skips the build if the image tag already exists in GHCR (unless manually triggered)- Checks out the Mattermost source at the selected tag- Unpins the base image digest in the Mattermost Dockerfile to allow ARM64 builds- Builds and pushes the image using QEMU + Buildx for `linux/arm64`- Optionally generates an SBOM and signs the image with cosign (keyless)
## Image details

- Registry: `ghcr.io/vtolj/mattermost`
- Tags: `<version>` and `latest`
- Build args:
  - `MM_PACKAGE` points to the official Mattermost ARM64 tarball for the selected version  - `PUID=2000` and `PGID=2000` are set for the container user/group
## Support matrix

- Architecture: `linux/arm64` only- Edition: Mattermost Team Edition- Tags: latest released version and `latest`
## Artifact provenance

The build pulls the official Mattermost ARM64 release tarball from:
`https://releases.mattermost.com/<version>/mattermost-<version>-linux-arm64.tar.gz`

## Pull the image

Examples:
```
docker pull ghcr.io/vtolj/mattermost:latest
docker pull ghcr.io/vtolj/mattermost:<version>
```

If you prefer a link, check this repository's Packages tab for the published image
## Usage

Manual trigger:
1. Open the GitHub Actions workflow "mattermost-arm64"2. Click "Run workflow" and optionally provide inputs:
   - `version`: Mattermost version (empty = latest release)   - `sign`: enable keyless cosign signing   - `sbom`: generate a CycloneDX SBOM artifact
Scheduled run:
- Daily at 03:00 UTC (`0 3 * * *`)
## Verification

- Inspect the published image:
```
docker buildx imagetools inspect ghcr.io/vtolj/mattermost:<version>
```
- Confirm the tag exists by pulling the image
## Security and signing

If signing is enabled, images are signed with keyless cosign. To verify:
```
cosign verify ghcr.io/vtolj/mattermost:<version>
```

## Requirements

- GitHub Actions enabled for the repository- `GITHUB_TOKEN` with `packages: write` (default for GHCR publishing)- `id-token: write` permission for keyless cosign signing
## FAQ

Q: Why did the build not run on schedule?
A: The workflow skips the build if the tag already exists in GHCR. Trigger it manually to force a rebuild
Q: How do I build a specific version?
A: Use the "Run workflow" input `version` (without a leading `v`)
## Issues and requests

Open a GitHub issue in this repository with:
- The requested version/tag- Workflow run link (if applicable)- Any relevant error logs
## Notes

- This workflow builds a single-arch ARM64 image; it does not produce a multi-arch manifest- Image caching uses the GitHub Actions runner filesystem at `/tmp/.buildx-cache`