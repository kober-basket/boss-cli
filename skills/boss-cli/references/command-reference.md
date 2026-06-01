# Command Reference

Use this reference when choosing exact `boss` commands, flags, and multi-step workflows.

## Install And Upgrade

```bash
uv tool install kober-boss-cli
uv tool upgrade kober-boss-cli
pipx install kober-boss-cli
pip install "kober-boss-cli[yaml]"
pip install "kober-boss-cli[browser]"
```

For local source testing:

```bash
uv build --no-sources --clear
uv tool install --force dist/kober_boss_cli-*.whl
```

## Auth Commands

| Command | Purpose |
| --- | --- |
| `boss status --json` | Check current auth and per-flow health |
| `boss login` | Extract browser cookies, fallback to QR login |
| `boss login --cookie-source chrome` | Prefer a specific browser |
| `boss login --qrcode` | QR-code login only |
| `boss logout` | Delete saved credentials and pause browser auto-login |

Supported browser sources include Chrome, Firefox, Edge, Brave, Arc, Chromium, Opera, Vivaldi, Safari, and LibreWolf when available through `browser-cookie3`.

## Job-Seeker Commands

| Command | Purpose | Example |
| --- | --- | --- |
| `boss search <keyword>` | Search jobs | `boss search "Python" --city 杭州 --salary 20-30K --json` |
| `boss recommend` | Personalized job recommendations | `boss recommend -p 2 --json` |
| `boss show <index>` | View a cached result after search/recommend | `boss show 3 --json` |
| `boss detail <securityId>` | View full job detail | `boss detail abc123 --json` |
| `boss export <keyword>` | Export search results | `boss export "Python" -n 50 -o jobs.csv` |
| `boss history` | View browsing history | `boss history --json` |
| `boss cities` | List supported cities | `boss cities` |
| `boss me` | View profile | `boss me --json` |
| `boss applied` | View applications | `boss applied -p 1 --json` |
| `boss interviews` | View interviews | `boss interviews --json` |
| `boss chat` | View communicated recruiters | `boss chat --json` |
| `boss greet <securityId>` | Greet/apply to one job | `boss greet abc123 --json` |
| `boss batch-greet <keyword>` | Batch greet search results | `boss batch-greet "golang" --city 杭州 -n 5 --dry-run` |

## Search Filters

| Filter | Flag | Common values |
| --- | --- | --- |
| City | `--city` | 北京, 上海, 杭州, 深圳, 全国 |
| Salary | `--salary` | 3K以下, 3-5K, 5-10K, 10-15K, 15-20K, 20-30K, 30-50K, 50K以上 |
| Experience | `--exp` | 不限, 在校/应届, 1年以内, 1-3年, 3-5年, 5-10年, 10年以上 |
| Degree | `--degree` | 不限, 大专, 本科, 硕士, 博士 |
| Industry | `--industry` | 互联网, 电子商务, 游戏, 人工智能, 金融, 教育培训, 医疗健康 |
| Company scale | `--scale` | 0-20人, 20-99人, 100-499人, 500-999人, 1000-9999人, 10000人以上 |
| Funding stage | `--stage` | 未融资, 天使轮, A轮, B轮, C轮, D轮及以上, 已上市, 不需要融资 |
| Job type | `--job-type` | 全职, 兼职, 实习 |

Run `boss cities` to inspect supported city names and codes.

## Recruiter Commands

Use recruiter commands only when the logged-in BOSS account is an employer/recruiter account.

| Command | Purpose | Example |
| --- | --- | --- |
| `boss recruiter jobs` | List posted jobs | `boss recruiter jobs --json` |
| `boss recruiter recommend` | Recommended candidates | `boss recruiter recommend --job <encryptJobId> --json` |
| `boss recruiter search <keyword>` | Search candidates | `boss recruiter search "golang" --city 深圳 --exp 3-5年 --json` |
| `boss recruiter greet <encryptGeekId>` | Initiate candidate contact | `boss recruiter greet <encryptGeekId> --job <encryptJobId> --json` |
| `boss recruiter batch-view <keyword>` | Batch view candidates | `boss recruiter batch-view "Python" -n 10 --dry-run` |
| `boss recruiter inbox` | Candidate inbox | `boss recruiter inbox --job <encryptJobId> --json` |
| `boss recruiter reply <friendId> <message>` | Reply to a candidate | `boss recruiter reply 123 "您好，方便聊聊吗？" --yes` |
| `boss recruiter chat <friendId>` | Chat history | `boss recruiter chat 123 --json` |
| `boss recruiter resume <encryptGeekId>` | View resume | `boss recruiter resume <encryptGeekId> --job <encryptJobId>` |
| `boss recruiter resume-download <id>` | Download resume as Markdown | `boss recruiter resume-download <id> --job <jobId>` |
| `boss recruiter request-resume <friendId>` | Request resume | `boss recruiter request-resume 123 --yes --json` |
| `boss recruiter exchange-phone <friendId>` | Request phone exchange | `boss recruiter exchange-phone 123 --yes --json` |
| `boss recruiter exchange-wechat <friendId>` | Request WeChat exchange | `boss recruiter exchange-wechat 123 --yes --json` |
| `boss recruiter invite-interview ...` | Invite interview | Use IDs returned by prior commands |
| `boss recruiter mark-unsuitable ...` | Mark candidate unsuitable | Use only after user confirmation |
| `boss recruiter job-close <encryptJobId>` | Take job offline | `boss recruiter job-close <id> --yes` |
| `boss recruiter job-reopen <encryptJobId>` | Reopen job | `boss recruiter job-reopen <id> --yes` |
| `boss recruiter labels` | Candidate labels | `boss recruiter labels --json` |
| `boss recruiter export` | Export candidates | `boss recruiter export -o candidates.csv` |

## Safe Workflow Examples

Search and inspect a job:

```bash
boss status --json
boss search "golang" --city 杭州 --salary 20-30K --json
boss show 1 --json
```

Export a filtered search:

```bash
boss export "Python" --city 杭州 --salary 20-30K -n 50 -o jobs.csv
```

Preview before batch greeting:

```bash
boss batch-greet "golang" --city 杭州 --salary 20-30K -n 10 --dry-run
```

Recruiter discovery:

```bash
boss recruiter jobs --json
boss recruiter recommend --job <encryptJobId> --json
boss recruiter resume <encryptGeekId> --job <encryptJobId>
```

## Troubleshooting

| Symptom | Action |
| --- | --- |
| `authenticated: false` | Run `boss login` |
| `环境异常` or `__zp_stoken__` expired | Run `boss logout && boss login` |
| `not_authenticated` envelope error | Re-login before retrying protected commands |
| Rate limited or code 9 | Wait; do not parallelize requests |
| Search has no results | Relax filters or confirm city with `boss cities` |
| Browser cookie extraction fails | Try `boss login --qrcode` |
| `PermissionError` on startup | Run from a readable directory or reinstall the patched package |
