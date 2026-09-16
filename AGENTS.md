# Dashboard conventions

Grafana dashboard JSONs, one kebab-case file per dashboard at the repo root.
Dashboards are provisioned onto the Wiremind clusters by kube-prometheus-stack
(`wiremind-services-configuration`): the Grafana sidecar downloads the raw JSON
from `master`, so merging here deploys everywhere the dashboard is referenced.

- **Two JSON formats are accepted, both provisioned the same way**. Grafana
  13.x (13.2.0 fleet-wide) file-provisions both the classic v1 model
  (`schemaVersion` + `panels`/`gridPos`) and the v2 resource model
  (`apiVersion: dashboard.grafana.app/v2`, `kind: Dashboard`,
  `spec.elements`/`spec.layout`). Grafana stores a v2 file as `storedVersion:
  v2` with `managedBy: classic-file-provisioning`; seven dashboards in this repo
  already use it. Export from the UI with **Export as JSON → Kubernetes
  resource** (v2) or **Classic** (v1); keep the format the file already has.
  In v2, the stable identifier is `metadata.name` (same role as `uid`), and
  the datasource reference is `"datasource": {"name": "${datasource}"}`. Strip
  volatile `metadata` fields (`generation`, `creationTimestamp`, `resourceVersion`)
  before committing.
- **Never hardcode a datasource**: declare a `datasource` template variable
  (`type: datasource`, `query: prometheus`) and reference `${datasource}`
  everywhere. Enable the cleanup hook: `git config core.hooksPath .githooks`.
- **Multi-cluster by default**: queries go through Thanos, so add a `cluster`
  query variable (`label_values(<metric>, cluster)`, multi + include-all) and
  filter every query with `cluster=~"$cluster"`. Add `namespace` (chained on
  `$cluster`) when it makes sense.
- Give each dashboard a stable `uid` and `tags`; keep the `uid` unchanged when
  updating, it is part of dashboard URLs.
- On stat/gauge/bar-gauge panels, set the field default
  `displayName: ${__series.name}` if the series name must stay visible when a
  query returns a single series (Grafana hides it by default).
- To provision a new dashboard, add a `dashboards:` entry pointing at the raw
  `master` URL in `helmfile/services/monitoring/kube-prometheus-stack/values/base/kube-prometheus-stack.chart.yaml.gotmpl`
  in `wiremind-services-configuration`.
