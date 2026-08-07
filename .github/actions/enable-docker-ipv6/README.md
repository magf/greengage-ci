# Enable Docker IPv6 Action

Configures the Docker daemon on the runner to hand out IPv6 addresses to containers.

## Usage

```yaml
- name: Enable Docker IPv6
  uses: greengagedb/greengage-ci/.github/actions/enable-docker-ipv6@v51
```

Run this **after** `maximize-disk-space` - that action restarts Docker as part of relocating its storage, which would otherwise undo this step.

## Inputs

Name            | Description                              | Required | Default
--------------- | ---------------------------------------- | -------- | -----------------
`fixed-cidr-v6` | IPv6 subnet Docker assigns to containers | No       | `2001:db8:1::/64`

## What it does

1. **Merges IPv6 settings into `/etc/docker/daemon.json`** - adds `ipv6: true` and `fixed-cidr-v6` without dropping any settings already in the file (e.g. registry mirrors).
2. **Restarts the Docker daemon** to pick up the new config.
3. **Verifies** a throwaway container actually receives a non-link-local IPv6 address, failing fast with a clear error if it doesn't.

## Why this is needed

`ubuntu-24.04`/`ubuntu-latest` GitHub-hosted runners have no real outbound IPv6 route - that part can't be fixed from a workflow. But GPDB's own `isIPv6Available()` check (used by both the interconnect unit tests run during the Docker build, and indirectly by test harnesses that resolve the runner's own hostname) only requires a **locally bindable** non-link-local IPv6 address plus an IPv6 loopback - it never needs internet routing. Docker can hand out such an address purely internally (a private ULA-like range), which is enough to satisfy that check without needing GitHub to add real IPv6 networking to hosted runners.
