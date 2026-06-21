# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a configuration and rules repository for [OpenClash](https://github.com/vernesong/OpenClash), an OpenWrt network traffic routing plugin. It provides:
- Subscription conversion templates (INI format) for Subconverter
- Custom traffic routing rule sets in multiple formats
- Shell scripts for OpenClash installation/maintenance on OpenWrt
- GitHub Actions for automatic rule generation and sync

## Repository Structure

- `rule/` — Source-of-truth rule lists (`.list`) and their auto-generated variants (`.yaml`, `.mrs`)
- `cfg/` — Subconverter INI templates (`Custom_Clash.ini` is the main/recommended one)
- `game_rule/` — Hand-curated game traffic rules (YAML)
- `shell/` — Bash scripts for OpenWrt: CPU detection, OpenClash install/update
- `py/` — Python maintenance scripts (not for end users)
- `overwrite/` — OpenClash remote overwrite config examples (git submodule: `Giveupmoon/OpenClash_Overwrite`)
- `wiki/` — Auto-synced GitHub Wiki backup (do not edit directly; managed by workflow)
- `doc/` — Documentation assets synced to GitHub Wiki

## Rule File Pipeline

**Do not manually edit** `*_Domain.yaml`, `*_IP.yaml`, `*_Classical.yaml`, `*_Classical_IP.yaml`, or `.mrs` files — these are auto-generated from the corresponding `.list` files by the `auto-generate-rules` GitHub Actions workflow whenever a `.list` file is pushed to `main`.

The pipeline: `.list` → `_Domain.yaml` / `_IP.yaml` / `_Classical.yaml` / `_Classical_IP.yaml` → `.mrs` (via `mihomo convert-ruleset`)

Covered rule bases: `Custom_Direct`, `Custom_Proxy`, `Steam_CDN`, `Encrypted_DNS`

**To add/modify rules**: edit only the `.list` source file. Push to `main` to trigger auto-generation.

## Rule File Formats

| Suffix | Type | Use case |
|---|---|---|
| `.list` | Raw list | Subconverter `ruleset=` references |
| `_Classical.yaml` | Classical (domain+IP) | `rule-providers` |
| `_Classical_IP.yaml` | Classical (IP only) | `rule-providers` |
| `_Domain.yaml` | Domain type | `rule-providers` (faster lookup) |
| `_IP.yaml` | IP-CIDR type | `rule-providers` (faster lookup) |
| `.mrs` | Mihomo binary | `rule-providers` (fastest load) |

Performance note: Domain/IP-CIDR types outperform Classical for rule matching; `.mrs` loads faster than `.yaml` but does not affect runtime matching speed.

## Python Scripts

```bash
# Regenerate Game_Download_CDN.list from v2fly upstream (run locally or via workflow)
python py/generate_game_cdn.py
```

This script downloads `v2fly/domain-list-community` game platform data and converts it to Clash `.list` format. The workflow `auto-update-game-cdn` runs this every 8 hours automatically.

## GitHub Actions Workflows

Key automated workflows (all target `main` branch):

| Workflow | Trigger | What it does |
|---|---|---|
| `auto-generate-rules` | Push to `rule/*.list` | Generates all YAML/MRS variants from `.list` files |
| `auto-update-game-cdn` | Every 8h | Runs `py/generate_game_cdn.py`, commits if changed |
| `auto-update-mainland` | `cfg/Custom_Clash.ini` change | Generates `Custom_Clash_Mainland.ini` |
| `purge-jsdelivr` | Changes in `cfg/`, `rule/`, etc. | Flushes jsDelivr CDN cache (60s debounce) |
| `sync_custom_clash` | `cfg/Custom_Clash.ini` change | Syncs template to downstream repo |
| `push-doc-to-wiki` | `doc/**` change | Syncs docs to GitHub Wiki |

Optional repo variable `WORK_BRANCH`: when set, workflows operate on that branch instead of `main`.

## Commit Message Convention

Auto-generated commits follow: `chore(rules): auto generate <filename> from <source>`

Manual commits in this repo use conventional commits with scopes like `feat`, `chore`, `fix`.
