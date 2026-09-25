# elio-uptime

A scheduled check of Elio's two public health endpoints. It holds no application code.

- `/api/health` says which build is running.
- `/api/health/ready` says whether the app can reach its database.

GitHub Actions requests `/api/health` every 10 minutes and both endpoints every 30 minutes. Readiness runs less often because each check wakes the database, which then stays up for 5 minutes. If either fails twice in a row, the workflow opens one issue labelled `outage` and assigns it to the repository owner. The first passing run afterwards closes the issue.

## How fast it notices

The schedule is nominal. GitHub starts scheduled runs late, often by 5 to 15 minutes or more, and can skip them when it is under load. Plan on a health failure being noticed within 20 to 30 minutes, and a database failure within about 45. Once the app's own error reporting is enabled, failed requests from real users will be reported sooner than this.

## Drills

- Instant: run the workflow by hand (Actions, uptime, Run workflow) and set `base_url` to a URL that fails. The resulting issue is labelled `outage-drill`, so it never closes a real outage.
- Real schedule: set the repository variable `ELIO_BASE_URL` to a failing URL, wait for the next scheduled run, then delete the variable.

## Secret

The readiness check sends the repository secret `ELIO_PROBE_TOKEN` in an `x-elio-probe-token` header, and only to production. The Worker runs a database check only for that token; anyone else gets the health answer with `db: "not-checked"`, which this check treats as a failure. Set the same value in both places.

## Settings

Repository variables, all optional:

- `ELIO_BASE_URL`: target to probe instead of production.
- `CHECK_READINESS`: set to `false` to skip `/api/health/ready`.

## Is it still running?

GitHub disables scheduled workflows in a public repository after 60 days with no activity, and a disabled monitor is silent. The workflow pushes one empty commit on the first of each month to prevent that. To check, run

```bash
gh workflow list --repo JNGARE/elio-uptime --all
```

The `uptime` row must say `active`. `disabled_inactivity` means the schedule stopped: re-enable it with `gh workflow enable uptime --repo JNGARE/elio-uptime`. The Actions tab also shows the time of the last scheduled run.
