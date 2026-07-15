---
id: release-deploy-harness-image
title: Release and deploy the harness image — two version pins, recreate not restart
category: PROCESS
domain: harness
severity: high
created: 2026-07-15
applies_when:
  intents:
    - release
    - deploy
    - version bump
    - docker image
    - betterai upgrade
related:
  - gate-escape-hatches
---

## What this rule says

The harness version is pinned in exactly TWO places that must move together: `pyproject.toml` (`version`) and `app/installer/install_env.py` (`BETTERAI_IMAGE = "ghcr.io/nicov7/personal-harness-py:<ver>"`). Release = bump both on main (via PR), `docker build -t ghcr.io/nicov7/personal-harness-py:<ver> .`, push via `gh auth token | docker login ghcr.io -u <user> --password-stdin`. Deploy = update the `image:` line in `~/.betterai/docker-compose.yml`, then `docker compose up -d --force-recreate betterai` — NEVER `docker restart`, which applies neither env_file changes nor a new image (both bind at container creation). Verify new code is actually live by checking `GET /health` for the counter fields (`sessions`, `rss_kb`) that only new code serves, not just `status: ok`.

## Why it matters

A merged fix that is never deployed protects nobody: the BAI-701 deadlock fix sat on main while the 0.4.1 container kept wedging real sessions, and `docker restart` after an env flip silently changed nothing — two hours of live debugging that a deploy runbook prevents.

## When this applies

Any harness release, image upgrade, or live env change (`~/.betterai/.env` keys, gate toggles).

## What good looks like

Bump both pins in one PR → build + push tag → sed compose → force-recreate → `/health` shows new counters → one live gate round-trip passes.

## Anti-patterns

`docker restart` to apply env or image changes; bumping `pyproject.toml` but not `BETTERAI_IMAGE` (installer mints stale envs); trusting `status: ok` as proof of the new build; editing gates in `.env` without recreating.
