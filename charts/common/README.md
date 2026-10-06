# Common library helper

Use this library to build helm applications in the uniform way

```yaml
# Chart.yaml
dependencies:
  - name: common
    version: 2.5.0
    repository: oci://ghcr.io/technicaldomain/helm
```

Each template of an application chart is one call with an (optionally empty) override that is
merged over the library's output:

```yaml
{{- template "common.deployment" (list . "application.deployment") -}}
{{- define "application.deployment" -}}
{{- end -}}
```

## Templates

| Template | Renders | Enabled by |
|---|---|---|
| `common.deployment` | Deployment (one container, `name: app`) | always |
| `common.configmap` | ConfigMap (`data` from the override) | always |
| `common.secret` | Secret (`data` from the override) | always |
| `common.service` | Service | `service.enabled` (default true) |
| `common.serviceaccount` | ServiceAccount | `serviceAccount.create` |
| `common.ingress` | Ingress | `ingress.enabled` |
| `common.httproute` | Gateway API HTTPRoute | `gateway.enabled` (default **true**) |
| `common.hpa` | HorizontalPodAutoscaler | `hpa.enabled` |
| `common.pdb` | PodDisruptionBudget (2.5.0) | `pdb.enabled` |
| `common.persistentvolumeclaim` | PersistentVolumeClaim | `persistence.enabled` without `existingClaim` |
| `common.serviceMonitor`, `common.podMonitor` | Prometheus Operator monitors | the chart's call |
| `common.vmServiceScrape` | VictoriaMetrics VMServiceScrape | the chart's call |

## Values added in 2.5.0

All of them are off (nothing rendered) unless set, so existing charts render unchanged.

```yaml
# Pod metadata and spec (common.deployment)
podLabels: {}                         # extra pod labels
podAnnotations: {}                    # extra pod annotations
automountServiceAccountToken: false   # rendered only when the key is set
enableServiceLinks: false             # rendered only when the key is set
terminationGracePeriodSeconds: 60
priorityClassName: ""
nodeSelector: {}
topologySpread:                       # one topologySpreadConstraint per key, selecting this release's pods
  topologyKeys: []                    # e.g. [topology.kubernetes.io/zone, kubernetes.io/hostname]
  maxSkew: 1
  whenUnsatisfiable: ScheduleAnyway

# Probes (common.container)
healthCheck:
  kind: http                          # http | tcp | disabled, for every probe
  scheme: HTTPS                       # httpGet scheme for every http probe (HTTP when unset)
  liveness:
    kind: tcp                         # a probe's own kind wins over healthCheck.kind
    scheme: HTTPS                     # a probe's own scheme wins over healthCheck.scheme

# The PersistentVolumeClaim mounted into the container (common.deployment)
persistence:
  enabled: true
  mountPath: /var/lib/app             # adds the volume and the volumeMount
  volumeName: data                    # default "data"

# The chart's ConfigMap (common.configmap, named like every resource) mounted as files
configmap:
  enabled: false                      # existing key: true also loads the ConfigMap as env (envFrom)
  mountPath: /etc/app                 # adds the volume and a read-only volumeMount
  volumeName: config                  # default "config"

# PodDisruptionBudget (common.pdb)
pdb:
  enabled: false
  maxUnavailable: 1                   # used when minAvailable is not set
  minAvailable: 1
  unhealthyPodEvictionPolicy: ""      # IfHealthyBudget | AlwaysAllow

# Monitors (common.serviceMonitor, common.podMonitor, common.vmServiceScrape)
monitoring:
  labels: {}                          # extra labels on the monitor (e.g. the Prometheus selector)
  endpoint: {}                        # extra fields of the scrape endpoint, e.g.
  #  scheme: https
  #  tlsConfig: {insecureSkipVerify: true}
  #  authorization: {type: Bearer, credentials: {name: app-metrics, key: token}}
```
