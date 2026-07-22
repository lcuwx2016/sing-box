# Fork maintenance

## Sources

- Official baseline: [`SagerNet/sing-box:testing`](https://github.com/SagerNet/sing-box/tree/testing)
- XHTTP source: [`flyzstu/sing-box:dev-next`](https://github.com/flyzstu/sing-box/tree/dev-next)
- Integration and development releases: `main`
- Official stable mirror: `official-stable`

## Policy

- Merge official `testing` updates into `main` after validating them on an `integration/*` branch.
- Preserve the existing XHTTP implementation and selectively cherry-pick future XHTTP changes instead of merging its entire branch again.
- After the official `v1.14.0` (or newer) release is synced, rebuild the maintained XHTTP-only patch set on `integration/stable-xhttp` and create or update a pull request to `main`.
- Update `official-stable` daily from the latest non-prerelease official GitHub Release and mirror its original tag.
- Do not mirror alpha, beta, or release-candidate tags.
- Pull requests to `main` use GitHub Actions to verify Linux amd64, Windows amd64, Android ARM64, and Android ARMv7. They do not publish releases.
- Development releases and container images are manual workflows; pushing `main` does not build or publish them.
