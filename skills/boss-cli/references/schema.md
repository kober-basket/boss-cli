# Structured Output Reference

Use this reference when parsing `boss --json` or `boss --yaml` output.

## Status Output

`boss status --json` is intentionally not wrapped in the standard envelope. It returns a direct object:

```json
{
  "authenticated": true,
  "credential_present": true,
  "cookie_count": 4,
  "cookies": ["__zp_stoken__", "wbg", "wt2", "zp_at"],
  "search_authenticated": true,
  "recommend_authenticated": true
}
```

When logged out:

```json
{
  "authenticated": false,
  "credential_present": false
}
```

Use this command for the first authentication check because it is stable and cheap.

## Standard Success Envelope

Most other commands with `--json` or `--yaml` return:

```json
{
  "ok": true,
  "schema_version": "1",
  "data": {}
}
```

Payloads always live under `data`. The payload shape depends on the command and upstream BOSS API response.

## Standard Error Envelope

Structured command errors return:

```json
{
  "ok": false,
  "schema_version": "1",
  "data": null,
  "error": {
    "code": "not_authenticated",
    "message": "环境异常 (__zp_stoken__ 已过期)。请重新登录: boss logout && boss login"
  }
}
```

## Error Codes

| Code | Meaning | Agent action |
| --- | --- | --- |
| `not_authenticated` | Session expired, invalid, or missing | Run `boss logout && boss login`, unless user wants to stay logged out |
| `rate_limited` | Too many requests or upstream anti-abuse response | Wait; do not parallelize or retry aggressively |
| `invalid_params` | Missing or invalid arguments | Correct flags or IDs from previous command output |
| `api_error` | Upstream BOSS API returned an error | Report concise context and retry only if safe |
| `unknown_error` | Unexpected local or upstream failure | Re-run with `boss -v ...` if debugging is requested |

## YAML Behavior

When stdout is not a TTY, some commands default to YAML for agent readability. Prefer `--json` when writing parsing code. If `pyyaml` is not installed, `--yaml` falls back to JSON.

Install YAML support if needed:

```bash
pip install "kober-boss-cli[yaml]"
```

## Parsing Guidelines

- Check process exit code first.
- For `status`, read top-level `authenticated`.
- For enveloped commands, require `ok: true` before reading `data`.
- For failed enveloped commands, surface `error.code` and `error.message`.
- Do not log cookie values, raw `BOSS_COOKIES`, or full request headers.
