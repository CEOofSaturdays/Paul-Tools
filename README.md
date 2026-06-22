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
