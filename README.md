# Prometheus Alerts

This Repository will be the version control for the different custom alerts and the configurations files required to make the alerts work.

## Custom Alert Requirements

* To create intelligent alerts that will be triggered when failing thresholds are reached and that will provide complete and pertinent information.
* It is neccessary to see the origination of the error (foundation, component, job, etc).

## Creating Custom Alerts

Custom alerts can be created in a .alerts file. To create an Alert you need the declare the following sections:

1. **"expr"(expression)** - The Prometheus query that will be evaluated to decide if an alert should be fired.

2. **"for"(optional)** - That tells Alertmanager how long the expression has to be in an error state before the alert is fired.

3. **"labels"** - That allows additional labels to be attached to the alert. Any existing conflicting labels will be overwritte. (Can be templated).

4. **"annotations"** - These allow longer error messages and possibly even helpful links to be bundled with the alert. (Can be templated).

### Templating
Label and annotation values can be templated using console templates(go templating language). The `$labels` variable holds the label key/value pairs of an alert instance and `$value` holds the evaluated value of an alert instance.

**Helpful Links**

* [Prometheus Console Templates Doc](https://prometheus.io/docs/visualization/consoles/)
* [Go Templating Language Docs](https://godoc.org/text/template)


### Sample Alert

```

ALERT BOSHJobUnhealthy
  IF max(bosh_job_healthy{bosh_job_name!~"^compilation.*",bosh_deployment!="bosh-health-check"}) by(environment, bosh_name, bosh_deployment, bosh_job_name, bosh_job_index) < 1
  FOR <%= p('bosh_alerts.job_unhealthy.evaluation_time') %>
  LABELS {
    service = "bosh-job",
    severity = "warning",
  }
  ANNOTATIONS {
    summary = "BOSH Job `{{$labels.environment}}/{{$labels.bosh_name}}/{{$labels.bosh_deployment}}/{{$labels.bosh_job_name}}/{{$labels.bosh_job_index}}` is unhealthy",
    description = "BOSH Job `{{$labels.environment}}/{{$labels.bosh_name}}/{{$labels.bosh_deployment}}/{{$labels.bosh_job_name}}/{{$labels.bosh_job_index}}` is being reported unhealthy",
  }

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
