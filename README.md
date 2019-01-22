# Bull Queue Exporter

**Prometheus exporter for Bull metrics.**

<a href="https://prometheus.io/">
    <img src="https://upload.wikimedia.org/wikipedia/en/thumb/3/38/Prometheus_software_logo.svg/115px-Prometheus_software_logo.svg.png" height="115">
</a>
<a href="https://github.com/OptimalBits/bull">
    <img src="https://github.com/OptimalBits/bull/blob/develop/support/logo@2x.png" height="115" />
</a>

## Metrics

| Metric                       | type    | description |
|------------------------------|---------|-------------|
| bull_queue_completed         | counter | Total number of completed jobs                          |
| bull_queue_complete_duration | summary | Processing time for completed jobs                      |
| bull_queue_active            | counter | Total number of active jobs (currently being processed) |
| bull_queue_delayed           | counter | Total number of jobs that will run in the future        |
| bull_queue_failed            | counter | Total number of failed jobs                             |
| bull_queue_waiting           | counter | Total number of jobs waiting to be processed            |

## Kubernetes Usage

### Environment variables for default docker image

| variable             | default                  | description                                |
|----------------------|--------------------------|--------------------------------------------|
| EXPORTER_REDIS_URL   | redis://localhost:6379/0 | Redis uri to connect                       |
| EXPORTER_PREFIX      | bull                     | prefix for queues                          |
| EXPORTER_STAT_PREFIX | bull_queue_              | prefix for exported metrics                |
| EXPORTER_QUEUES      | -                        | a space separated list of queues to check  |


### Example deployment

see: [k8s-sample.yaml](./docs/k8s-sample.yaml) for more options

```yaml
apiVersion: apps/v1

kind: Deployment
metadata:
  name: bull-exporter
  labels:
    app: bull
    role: exporter

spec:
  selector:
    matchLabels:
      app: bull
      role: exporter
  replicas: 1
  template:
    metadata:
      labels:
        app: bull
        role: exporter
    spec:
      containers:
        - name: bull-exporter
          image: uphabit/bull_exporter:latest
          env:
              # space delimited list of queues
            - name: EXPORTER_QUEUES
              value: "mail job_one video audio"

              # find the redis service in the cluster
            - name: EXPORTER_REDIS_URL
              value: redis://redis:6379/0
---
apiVersion: v1
kind: Service

metadata:
  name: bull-exporter
  labels:
    app: bull
    role: exporter
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/port: '5959'
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 9538
      targetPort: 9538
  selector:
    app: bull
    role: exporter

```