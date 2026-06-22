# Paul Agent Tools

This repository is the public release bucket for Paul agent tooling.

Source code lives outside this repository. Published GitHub Releases contain
the packaged `paul` CLI binaries, checksums, and the Paul agent tools release
skill bundle.

## Releases

Install and update flows use this repository by default:

```bash
paul install-check
paul update
```

Release assets are attached to GitHub Releases:

- `paul_darwin_amd64.tar.gz`
- `paul_darwin_arm64.tar.gz`
- `paul_linux_amd64.tar.gz`
- `paul_linux_arm64.tar.gz`
- `paul-agent-tools-release-skill.tar.gz`
- `checksums.txt`

## Agent Skill

The CLI-use skill is checked in at `skills/paul/SKILL.md`.

Use it when an agent needs to operate Paul through the `paul` CLI, authenticate
the CLI, inspect or transition work items, manage calendars or settings, or
understand Paul's domain to project to task to sub-task model.

## Agent Install Prompt

Copy this prompt into an agent session when you want that agent to install and
use Paul:

```text
Use the public Paul Agent Tools repository at https://github.com/CEOofSaturdays/Paul-Tools.

Install or update the `paul` CLI from GitHub Releases, then verify the release assets.

If `paul` is already installed, run:

paul update
paul install-check

If `paul` is not installed, install the latest release for macOS/Linux:

REPO="CEOofSaturdays/Paul-Tools"
TAG="$(curl -fsSL "https://api.github.com/repos/${REPO}/releases/latest" | sed -nE 's/.*"tag_name": "([^"]+)".*/\1/p' | head -n 1)"
OS="$(uname -s | tr '[:upper:]' '[:lower:]')"
ARCH="$(uname -m)"
case "$ARCH" in
  arm64 | aarch64) ARCH="arm64" ;;
  x86_64 | amd64) ARCH="amd64" ;;
esac
INSTALL_DIR="${INSTALL_DIR:-/usr/local/bin}"
tmpdir="$(mktemp -d)"
curl -fsSL "https://github.com/${REPO}/releases/download/${TAG}/paul_${OS}_${ARCH}.tar.gz" -o "$tmpdir/paul.tar.gz"
tar -xzf "$tmpdir/paul.tar.gz" -C "$tmpdir" paul
install -m 0755 "$tmpdir/paul" "$INSTALL_DIR/paul" || sudo install -m 0755 "$tmpdir/paul" "$INSTALL_DIR/paul"
paul install-check

Confirm `paul install-check` reports `ok: true`, then read and follow the
CLI-use skill at `skills/paul/SKILL.md` in CEOofSaturdays/Paul-Tools before
operating Paul.

Do not set a GitHub token for normal Paul install or update flows; this repository is public.
```
