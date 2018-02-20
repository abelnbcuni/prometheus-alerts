# Prometheus Alerts

This Repository will be the version control for the different custom alerts and the configurations files required to make the alerts work.

## Custom Alert Requirements

* To create intelligent alerts that will be triggered when failing thresholds are reached and that will provide complete and pertinent information.
* It is neccessary to see the origination of the error (foundation, component, job, etc).

## Creating Custom Alerts

Custom alerts can be created in an .alerts or .yaml file. To create an Alert you need the declare the following sections:

1. **"expr:"(expression)** - The Prometheus query that will be evaluated to decide if an alert should be fired.

2. **"for:"(optional)** - That tells Alertmanager how long the expression has to be in an error state before the alert is fired.

3. **"labels:"** - That allows additional labels to be attached to the alert. Any existing conflicting labels will be overwritten. (Can be templated).

4. **"annotations:"** - These allow longer error messages and possibly even helpful links to be bundled with the alert. (Can be templated).

### Templating
Label and annotation values can be templated using console templates(go templating language). The `$labels` variable holds the label key/value pairs of an alert instance and `$value` holds the evaluated value of an alert instance.

**Helpful Links**

* [Prometheus Console Templates Doc](https://prometheus.io/docs/visualization/consoles/)
* [Go Templating Language Docs](https://godoc.org/text/template)

## Alerts Version 1.x vs. 2.x

Alerts in Prometheus Version 1.x uses a JSON like format, since Prometheus 2.x it moved to a YAML format. There is a command line tool that allows you to upgrade from the older rules format to the newer format.

```
promtool update rules <1.x rules file>
```


### Sample Alert (Version 1.x)

```

ALERT BOSH_Job_System_Disk_Low
  IF avg(bosh_job_system_disk_percent{bosh_job_name!~"^compilation.*",bosh_deployment!="bosh-health-check"}) by(environment, bosh_name, bosh_deployment, bosh_job_name, bosh_job_index) > 40
  FOR 10m
  LABELS {
    service = "bosh-job",
    severity = "warning",
  }
  ANNOTATIONS {
    summary = "BOSH Job `{{$labels.environment}}/{{$labels.bosh_name}}/{{$labels.bosh_deployment}}/{{$labels.bosh_job_name}}/{{$labels.bosh_job_index}}/{{$labels.bosh_job_ip}}` is running out of system disk",
    description = "BOSH Job `{{$labels.environment}}/{{$labels.bosh_name}}/{{$labels.bosh_deployment}}/{{$labels.bosh_job_name}}/{{$labels.bosh_job_index}}/{{$labels.bosh_job_ip}}` has used more than 75% of its system disk for 10 mins %>: {{$value}}%",
  }

```

### Sample Alert (Version 2.x)

```YAML

alert: BOSH_System_VM_Down
expr: max(bosh_job_healthy{bosh_deployment!="bosh-health-check",bosh_job_name!~"^compilation.*"})
  BY (environment, bosh_name, bosh_deployment, bosh_job_name, bosh_job_index,
  bosh_job_ip) < 1
for: 10m
labels:
  service: bosh-job-vm
  severity: critical
annotations:
  description: BOSH System VM `{{$labels.environment}}/{{$labels.bosh_deployment}}/{{$labels.bosh_job_name}}/{{$labels.bosh_job_index}}/{{$labels.bosh_job_ip}}`
    has been down for more than 10 mins
  summary: BOSH System VM `{{$labels.environment}}/{{$labels.bosh_deployment}}/{{$labels.bosh_job_name}}/{{$labels.bosh_job_index}}/{{$labels.bosh_job_ip}}`
    has been down for more than 10 mins

```

New Alerts must be added to the prometheus.yml (see below) - prometheus.yml needs the path to the custom alert rule file as well as a job release for any additional new exporters

```YAML
...
prometheus:
        rule_files:
          - /var/vcap/jobs/bosh_alerts/*.alerts
          - /var/vcap/jobs/cloudfoundry_alerts/*.alerts
          - /var/vcap/jobs/consul_alerts/*.alerts
          - /var/vcap/jobs/haproxy_alerts/*.alerts
          - /var/vcap/jobs/kubernetes_alerts/*.alerts
          - /var/vcap/jobs/mysql_alerts/*.alerts
          - /var/vcap/jobs/p_rabbitmq_alerts/*.alerts
          - /var/vcap/jobs/p_redis_alerts/*.alerts
          - /var/vcap/jobs/postgres_alerts/*.alerts
          - /var/vcap/jobs/probe_alerts/*.alerts
          - /var/vcap/jobs/prometheus_alerts/*.alerts
          - /var/vcap/jobs/rabbitmq_alerts/*.alerts
          - /var/vcap/jobs/redis_alerts/*.alerts
          - <Path To Custom Alert>/<Alertname>.alerts
...

```
