## prometheus config (change namespace in the last line to your prometheus namespace)

```
apiVersion: v1
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'kubernetes-pods'
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      tls_config:
        insecure_skip_verify: true
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitor-system
```

## Pod example
```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    prometheus.io/path: /metrics
    prometheus.io/port: "443"
    prometheus.io/scheme: https
    prometheus.io/scrape: "true"
  labels:
    app: nginx
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

## Summary
1. support both https and http
2. support using service account token
