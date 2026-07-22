# Fork maintenance

## Sources

- Official baseline: [`SagerNet/sing-box:testing`](https://github.com/SagerNet/sing-box/tree/testing)
- XHTTP source: [`flyzstu/sing-box:dev-next`](https://github.com/flyzstu/sing-box/tree/dev-next)
- Integration and development releases: `main`
- Official stable mirror: `official-stable`

## Policy

- Merge official `testing` updates into `main` after validating them on an `integration/*` branch.
- Preserve the existing XHTTP implementation and selectively cherry-pick future XHTTP changes instead of merging its entire branch again.
- Build stable custom releases from an official stable tag on `release/vX.Y.Z-xhttp` branches.
- Update `official-stable` daily from the latest non-prerelease official GitHub Release and mirror its original tag.
- Do not mirror alpha, beta, or release-candidate tags.
- Use GitHub Actions to verify Linux amd64, Windows amd64, Android ARM64, and Android ARMv7 before updating `main`.
