---
name: boss-cli
description: "Use the installed kober-boss-cli `boss` command for BOSS 直聘 and zhipin.com workflows: authenticate with browser cookies or QR code, search jobs, inspect recommendations, view applications/interviews/chats/profile/history, greet or apply, export results, and run recruiter-side candidate/job/chat workflows. Invoke for any user request involving BOSS 直聘 automation, job-search data extraction, recruiter operations, or agent-friendly JSON/YAML output from the boss CLI."
---

# BOSS CLI

Use this skill when the user wants to operate BOSS 直聘 through the `boss` terminal command. Prefer the packaged CLI (`kober-boss-cli`) rather than copying code or installing older upstream packages.

## Quick Start

Check whether the command exists:

```bash
boss --version
```

If it is missing, install the current package:

```bash
uv tool install kober-boss-cli
```

For source-tree testing before release, build and install the local wheel:

```bash
uv build --no-sources --clear
uv tool install --force dist/kober_boss_cli-*.whl
```

## Authentication Workflow

Always check auth before commands that require a BOSS session:

```bash
boss status --json
```

Interpret `authenticated: true` as ready. If false or missing:

```bash
boss login
boss login --cookie-source chrome
boss login --qrcode
```

Use `boss login` when the user is already logged into zhipin.com in a browser. It extracts cookies from supported browsers and validates them. Use `--qrcode` when browser cookie extraction is unavailable.

`boss logout` deletes saved credentials and writes a local logout marker so later commands do not silently re-extract browser cookies. Run `boss login` again to re-enable browser extraction. An explicit `BOSS_COOKIES` environment variable still acts as a manual credential source.

Never ask the user to paste raw cookie values into chat logs. Treat all cookie output and `BOSS_COOKIES` values as secrets.

## Output Rules

Prefer explicit machine-readable output:

```bash
boss search "Python" --city 杭州 --json
boss me --json
boss recruiter jobs --json
```

Most commands with `--json` or `--yaml` return the standard envelope documented in `references/schema.md`. `boss status --json` is the exception: it returns a direct status object for backward compatibility.

Rich terminal tables and panels are written to stderr, so stdout remains safe for pipes when `--json` is used.

## Workflows

For job seekers:

1. Run `boss status --json`.
2. Search or inspect recommendations with `--json`.
3. Use `boss show <index>` after a search for quick detail lookup, or `boss detail <securityId> --json` when you already have the ID.
4. Export only after confirming filters: `boss export "Python" -n 50 -o jobs.csv`.
5. Use `boss greet <securityId> --json` or `boss batch-greet ... --dry-run` before sending multiple greetings.

For recruiter-side work:

1. Run `boss recruiter jobs --json` to discover available job IDs.
2. Use `boss recruiter recommend --job <encryptJobId> --json` or `boss recruiter search <keyword> --job <encryptJobId> --json`.
3. Use `boss recruiter resume`, `resume-download`, `inbox`, `chat`, and `reply` only with IDs returned by earlier commands.
4. Require explicit user confirmation before actions that contact candidates, exchange contact info, invite interviews, close jobs, or mark candidates unsuitable.

## References

- Read `references/command-reference.md` when choosing exact commands, flags, filters, or recruiter operations.
- Read `references/schema.md` when parsing JSON/YAML output or handling errors.

## Safety And Reliability

- Do not parallelize BOSS API requests; the CLI includes jitter, cooldown, and retry behavior to reduce account risk.
- Prefer `--dry-run` for batch operations, then ask the user before sending greetings or viewing many candidates.
- If `环境异常`, `__zp_stoken__`, or `not_authenticated` appears, run `boss logout && boss login` unless the user asks to stay logged out.
- If `boss` crashes before showing help with `PermissionError: Operation not permitted`, ask the user to run from a readable directory or reinstall a build containing the safe cwd startup patch.
