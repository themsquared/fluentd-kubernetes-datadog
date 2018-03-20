This page describes the Datadog Kubernetes [Fluentd](http://www.fluentd.org/) plugin.

The plugin runs as a Kubernetes [DaemonSet](http://kubernetes.io/docs/admin/daemons/); it runs an instance of the plugin on each host in a cluster. Each plugin instance pulls system, kubelet, docker daemon, and container logs from the host and sends them, in JSON or text format, to an HTTP endpoint on a hosted collector in the [Datadog](http://www.datadoghq.com) service.

**Note** This plugin is community-supported. For support, add a request in the issues tab. 

- [Step 1  Create a Datadog Account](#step-1--create-a-datadog-account)
- [Step 2  Create a Kubernetes secret](#step-2--create-a-kubernetes-secret)
- [Step 3  Install the Datadog Kubernetes FluentD plugin](#step-3--install-the-datadog-kubernetes-fluentd-plugin)
  * [Option A  Install plugin using kubectl](#option-a--install-plugin-using-kubectl)
  * [Option B  Helm chart](#option-b--helm-chart)
- [Environment variables](#environment-variables)
    + [Override environment variables using annotations](#override-environment-variables-using-annotations)
    + [Exclude data using annotations](#exclude-data-using-annotations)
- [Step 4 Set up Heapster for metric collection](#step-4-set-up-heapster-for-metric-collection)
  * [Kubernetes ConfigMap](#kubernetes-configmap)
  * [Kubernetes Service](#kubernetes-service)
  * [Kubernetes Deployment](#kubernetes-deployment)
- [Log data](#log-data)
  * [Docker](#docker)
  * [Kubelet](#kubelet)
  * [Containers](#containers)
- [Taints and Tolerations](#taints-and-tolerations)


![deployment](https://github.com/themsquared/fluentd-kubernetes-datadog/blob/master/screenshots/kubernetes.png)

# Step 1  Create a Datadog account

Instructions to create Datadog Account here...

# Step 2  Create a Kubernetes secret

Create a secret in Kubernetes with the HTTP source URL. If you want to change the secret name, you must modify the Kubernetes manifest accordingly.

`kubectl create secret generic datadog --from-literal=dd_api_key=<<Your Datadog API Key>>`

You should see the confirmation message 

`secret "datadog" created.`

# Step 3  Install the Datadog Kubernetes FluentD plugin

Follow the instructions in Option A below to install the plugin using `kubectl`. If you prefer to use a Helm chart, see Option B. 

Before you start, see [Environment variables](#environment-variables) for information about settings you can customize, and how to use annotations to override selected environment variables and exclude data from being sent to Datadog.

## Option A  Install plugin using kubectl

See the sample Kubernetes DaemonSet and Role in [fluentd.yaml](/daemonset/rbac/fluentd.yaml).

1. Clone the [GitHub repo](https://github.com/Datadog/fluentd-kubernetes-sumologic).

2. In `fluentd-kubernetes-datadog`, install the chart using `kubectl`.

Which `.yaml` file you should use depends on whether or not you are running RBAC for authorization. RBAC is enabled by default as of Kubernetes 1.6.

**Non-RBAC (Kubernetes 1.5 and below)** 

`kubectl create -f /daemonset/nonrbac/fluentd.yaml` 

**RBAC (Kubernetes 1.6 and above)** <br/><br/>`kubectl create -f /daemonset/rbac/fluentd.yaml`


**Note** if you modified the command in Step 2 to use a different name, update the `.yaml` file to use the correct secret.

Logs should begin flowing into Datadog within a few minutes of plugin installation.

## Option B  Helm chart
If you use Helm to manage your Kubernetes resources, [[TODO]] Add Link here.

# Environment variables

Environment | Variable Description
----------- | --------------------
`CONCAT_SEPARATOR` |The character to use to delimit lines within the final concatenated message. Most multi-line messages contain a newline at the end of each line. <br/><br/> Default: ""
`EXCLUDE_CONTAINER_REGEX` |A regular expression for containers. Matching containers will be excluded from Datadog. The logs will still be sent to FluentD.
`EXCLUDE_FACILITY_REGEX`|A regular expression for syslog [facilities](https://en.wikipedia.org/wiki/Syslog#Facility). Matching facilities will be excluded from Datadog. The logs will still be sent to FluentD.
`EXCLUDE_HOST_REGEX`|A regular expression for hosts. Matching hosts will be excluded from Datadog. The logs will still be sent to FluentD.
`EXCLUDE_NAMESPACE_REGEX`|A regular expression for `namespaces`. Matching `namespaces` will be excluded from Datadog. The logs will still be sent to FluentD.
`EXCLUDE_PATH`|Files matching this pattern will be ignored by the `in_tail` plugin, and will not be sent to Kubernetes or Datadog. This can be a comma-separated list as well. See [in_tail](http://docs.fluentd.org/v0.12/articles/in_tail#excludepath) documentation for more information. <br/><br/> For example, defining `EXCLUDE_PATH` as shown below excludes all files matching `/var/log/containers/*.log`, <br/><br/>`...`<br/><br/>`env:`<br/>   - `name: EXCLUDE_PATH`<br/>         `value: "[\"/var/log/containers/*.log\"]"`
`EXCLUDE_POD_REGEX`|A regular expression for pods. Matching pods will be excluded from Datadog. The logs will still be sent to FluentD.
`EXCLUDE_PRIORITY_REGEX`|A regular expression for syslog [priorities](https://en.wikipedia.org/wiki/Syslog#Severity_level). Matching priorities will be excluded from Datadog. The logs will still be sent to FluentD.
`EXCLUDE_UNIT_REGEX` |A regular expression for `systemd` units. Matching units will be excluded from Datadog. The logs will still be sent to FluentD.
`FLUENTD_SOURCE`|Fluentd can tail files or query `systemd`. Allowable values: `file`, `Systemd`. <br/><br/>Default: `file` 
`FLUENTD_USER_CONFIG_DIR`|A directory of user-defined fluentd configuration files, which must be in the  `*.conf` directory in the container.
`FLUSH_INTERVAL` |How frequently to push logs to Datadog.<br/><br/>Default: `30s`
`KUBERNETES_META`|Include or exclude Kubernetes metadata such as `namespace` and `pod_name` if using JSON log format. <br/><br/>Default: `true`
`LOG_FORMAT`|Format in which to post logs to Datadog. Allowable values:<br/><br/>`text`—Logs will appear in Datadog in text format.<br/>`json`—Logs will appear in Datadog in json format.<br/>`json_merge`—Same as json but if the container logs in json format to stdout it will merge in the container json log at the root level and remove the log field.<br/><br/>Default: `json`
`MULTILINE_START_REGEXP`|The regular expression for the `concat` plugin to use when merging multi-line messages. Defaults to Julian dates, for example, Jul 29, 2017.
`NUM_THREADS`|Set the number of HTTP threads to SuDatadogmo. It might be necessary to do so in heavy-logging clusters. <br/><br/>Default: `1`
`READ_FROM_HEAD`|Start to read the logs from the head of file, not bottom. Only applies to containers log files. See in_tail doc for more information.<br/><br/>Default: `true` 
`SOURCE_CATEGORY` |Set the `_sourceCategory` metadata field in Datadog. <br/><br/>Default: `"%{namespace}/%{pod_name}"`
`SOURCE_CATEGORY_PREFIX`|Prepends a string that identifies the cluster to the `_sourceCategory` metadata field in Datadog.<br/><br/>Default:  `kubernetes/`
`SOURCE_CATEGORY_REPLACE_DASH` |Used to replace a dash (-) character with another character. <br/><br/>Default:  `/`<br/><br/>For example, a Pod called `travel-nginx-3629474229-dirmo` within namespace `app` will appear in Datadog with `_sourceCategory=app/travel/nginx`.
`SOURCE_HOST`|Set the `_sourceHost` metadata field in Datadog.<br/><br/>Default: `""`
`SOURCE_NAME`|Set the `_sourceName` metadata field in Datadog. <br/><br/> Default: `"%{namespace}.%{pod}.%{container}"`
`AUDIT_LOG_PATH`|Define the path to the [Kubernetes Audit Log](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/) <br/><br/> Default: `/mnt/log/kube-apiserver-audit.log`

The following table show which  environment variables affect which Fluentd sources.

| Environment Variable | Containers | Docker | Kubernetes | Systemd |
|----------------------|------------|--------|------------|---------|
| `EXCLUDE_CONTAINER_REGEX` | ✔ | ✘ | ✘ | ✘ |
| `EXCLUDE_FACILITY_REGEX` | ✘ | ✘ | ✘ | ✔ |
| `EXCLUDE_HOST_REGEX `| ✔ | ✘ | ✘ | ✔ |
| `EXCLUDE_NAMESPACE_REGEX` | ✔ | ✘ | ✔ | ✘ |
| `EXCLUDE_PATH` | ✔ | ✔ | ✔ | ✘ |
| `EXCLUDE_PRIORITY_REGEX` | ✘ | ✘ | ✘ | ✔ |
| `EXCLUDE_POD_REGEX` | ✔ | ✘ | ✘ | ✘ |
| `EXCLUDE_UNIT_REGEX` | ✘ | ✘ | ✘ | ✔ |

### Override environment variables using annotations
You can override the `LOG_FORMAT`, `SOURCE_CATEGORY` and `SOURCE_NAME` environment variables, per pod, using [Kubernetes annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/). For example:

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    app: mywebsite
  template:
    metadata:
      name: nginx
      labels:
        app: mywebsite
      annotations:
        datadoghq.com/format: "text"
        datadoghq.com/sourceCategory: "mywebsite/nginx"
        datadoghq.com/sourceName: "mywebsite_nginx"
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### Exclude data using annotations

You can also use the `datadoghq.com/exclude` annotation to exclude data from Datadog. This data is sent to FluentD, but not to Datadog.

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    app: mywebsite
  template:
    metadata:
      name: nginx
      labels:
        app: mywebsite
      annotations:
        datadoghq.com/format: "text"
        datadoghq.com/sourceCategory: "mywebsite/nginx"
        datadoghq.com/sourceName: "mywebsite_nginx"
        datadoghq.com/exclude: "true"
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

# Step 4 Set up Datadog Agent for metric collection

The recommended way to collect metrics from Kubernetes clusters is to use the Datadog Kubernets configuration as defined below:

1. Install Heapster in your Kubernetes cluster and configure a Graphite Sink to send the data in Graphite format to Sumo. For instructions, see 
https://github.com/kubernetes/heapster/blob/master/docs/sink-configuration.md#graphitecarbon. Assuming you have used the below YAML files to configure your system, then the sink option in graphite would be `--sink=graphite:tcp://sumo-graphite.kube-system.svc:2003`.  You may need to change this depending on the namespace you run the deployment in, the name of the service or the port number for your Graphite source.

# Log data
After performing the configuration described above, your logs should start streaming to Datadog in `json` or text format with the appropriate metadata. If you are using `json` format you can auto extract fields, for example `_sourceCategory=some/app | json auto`.

## Docker
![Docker Logs](/screenshots/docker.png)

## Kubelet
Note that Kubelet logs are only collected if you are using systemd.  Kubernetes no longer outputs the kubelet logs to a file.
![Docker Logs](/screenshots/kubelet.png)

## Containers
![Docker Logs](/screenshots/container.png)

# Taints and Tolerations
By default, the fluentd pods will schedule on, and therefore collect logs from, any worker nodes that do not have a taint and any master node that does not have a taint beyond the default master taint. If you would like to schedule pods on all nodes, regardless of taints, uncomment the following line from fluentd.yaml before applying it.

```
tolerations:
           #- operator: "Exists"
```
