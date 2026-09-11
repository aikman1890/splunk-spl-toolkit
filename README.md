# splunk-spl-toolkit

A collection of Splunk SPL searches, alert stanzas, and a Simple XML dashboard
built for SRE workflows: error-rate tracking, MTTR measurement, capacity
forecasting, and brute-force login detection.

## Layout

```text
splunk-spl-toolkit/
├── README.md
├── LICENSE
├── searches/
│   ├── error-rate-tracking.spl    # HTTP 5xx / app error % by service over time
│   ├── mttr-measurement.spl       # Avg time-to-restore by priority from ticket events
│   ├── capacity-forecasting.spl   # predict disk/CPU growth, 30-day horizon
│   └── failed-logins.spl          # Brute-force detection: >N failures per src_ip
├── alerts/
│   └── savedsearches.conf         # Alert stanzas: error-rate spike, failed-login burst
└── dashboards/
    └── sre-overview.xml           # Simple XML dashboard wiring the searches
```

## Assumptions

The searches assume the following field conventions. Adjust the `index=`,
`sourcetype=`, and field names to match your environment:

| Assumption | Search | Fields |
|------------|--------|--------|
| `index=web sourcetype=access_combined` — standard Apache/Nginx combined logs | error-rate-tracking | `service`, `status`, `uri_path` |
| `index=tickets sourcetype=ticket_events` — one event per ticket transition | mttr-measurement | `ticket_id`, `priority`, `status`, `opened_time`, `resolved_time` |
| `index=metrics sourcetype=perf` — periodic host perf samples | capacity-forecasting | `host`, `mount`, `disk_used_pct`, `cpu_pct` |
| `index=auth sourcetype=secure` — Linux auth logs | failed-logins | `src_ip`, `user`, `action` |

Token names used by the dashboard (`$timerange$`, `$service$`, `$priority$`)
match the input tokens defined in `dashboards/sre-overview.xml`. The
`.spl` files under `searches/` are standalone — each carries a commented
"Dashboard token variant" showing the tokenized form used in the dashboard.

## Installing the searches

1. Copy each `.spl` file's contents into Splunk: **Search → Save As → Report**.
2. For the dashboard, import `dashboards/sre-overview.xml`:
   **Dashboards → New Dashboard → Upload** (or drop it into
   `$SPLUNK_HOME/etc/apps/<app>/local/data/ui/views/sre_overview.xml`).
3. For the alerts, merge `alerts/savedsearches.conf` into your app's
   `local/savedsearches.conf` and reload (`splunk btool` / debug refresh).

Before enabling alerts in production, set `action.email.to` to a real
distribution list — the stanzas ship with a placeholder.

## License

MIT — see [LICENSE](LICENSE).
