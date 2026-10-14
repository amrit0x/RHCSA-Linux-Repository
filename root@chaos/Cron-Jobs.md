# Cron Jobs – Important Commands (RHCSA Notes)

Cron is used to schedule and automatically execute tasks at specified times in Linux.

## 1. Important `crontab` commands

| Command | Description |
| --- | --- |
| `crontab -e` | Edit current user's cron jobs |
| `crontab -l` | List current user's cron jobs |
| `crontab -r` | Remove all current user's cron jobs |
| `crontab -i` | Ask confirmation before removing cron jobs (used with `-r`) |
| `crontab -u amox -e` | Edit amox's cron jobs (as root) |
| `crontab -u amox -l` | List amox's cron jobs (as root) |
| `crontab -u amox -r` | Remove amox's cron jobs (as root) |
| `crontab -` | Install cron jobs from standard input |

For safer deletion, use:

```
crontab -i -r
```

## 2. Cron service commands

| Command | Description |
| --- | --- |
| `systemctl status crond` | Check cron service |
| `systemctl start crond` | Start cron service |
| `systemctl stop crond` | Stop cron service |
| `systemctl restart crond` | Restart cron service |
| `systemctl enable crond` | Enable at boot |
| `systemctl enable --now crond` | Start now and enable at boot |
| `journalctl -u crond` | View cron service logs |

## 3. Cron schedule format

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

### Important scheduling examples

| Cron expression | Meaning |
| --- | --- |
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour |
| `0 8 * * *` | Daily at 8 AM |
| `30 9 * * *` | Daily at 9:30 AM |
| `0 0 * * *` | Every midnight |
| `0 8 * * 1` | Every Monday at 8 AM |
| `0 0 1 * *` | First day of every month |
| `0 0 1 1 *` | Every January 1 |

## 4. Special cron directories (RHEL)

| Directory / File | Purpose |
| --- | --- |
| `/etc/crontab` | System-wide cron configuration |
| `/etc/cron.d/` | Additional system cron jobs |
| `/etc/cron.hourly/` | Hourly jobs |
| `/etc/cron.daily/` | Daily jobs |
| `/etc/cron.weekly/` | Weekly jobs |
| `/etc/cron.monthly/` | Monthly jobs |
| `/var/spool/cron/` | User crontab storage |
| `/etc/cron.allow` | Users allowed to use crontab |
| `/etc/cron.deny` | Users denied access, subject to allow/deny rules |

## 5. Practical example (RHCSA)

Task: Create a cron job that writes the date to `/tmp/date.txt` every 5 minutes.

Step 1 — Edit:

```
crontab -e
```

Step 2 — Add:

```
*/5 * * * * /usr/bin/date >> /tmp/date.txt
```

Step 3 — Verify:

```
crontab -l
```

Step 4 — Check output after the job runs:

```
cat /tmp/date.txt
```

Important: `>>` appends output to the file, while `>` overwrites it.

## 6. Cron shortcuts

| Shortcut | Meaning |
| --- | --- |
| `@reboot` | At system startup |
| `@hourly` | Every hour |
| `@daily` | Every day |
| `@weekly` | Every week |
| `@monthly` | Every month |
| `@yearly` | Every year |

Example:

```
@daily /usr/bin/date >> /tmp/daily.txt
```

RHCSA exam tip: Focus especially on `crontab -e`, `crontab -l`, `crontab -r`, `crontab -u`, cron time syntax, and `systemctl status crond`. These are the most useful commands for practical cron configuration.
