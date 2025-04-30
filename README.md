# wiremind-grafana-dashboards

This is a set of useful Grafana Dashboards used and maintained at Wiremind, which we think will be useful for other people.

## Contribute

To export an existing Grafana dashboard:

- Export from Grafana UI using the "Export the dashboard to use in another instance" toggle
- Please add the pre-commit hook before committing so that datasources are automatically cleaned-up:

`git config core.hooksPath .githooks`
