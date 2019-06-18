# Helm Chart for SnowAlert

## Introduction

This [Helm](https://github.com/kubernetes/helm) chart installs [SnowAlert](https://github.com/snowflakedb/SnowAlert) in a Kubernetes cluster. To learn more about SnowAlert, check out [this medium](https://medium.com/hashmapinc/snowalert-data-driven-security-analytics-using-snowflake-data-warehouse-3f046b779d54) post.

## Prerequisites

- Kubernetes cluster 1.10+
- Helm 2.8.0+

## Install Chart

### Clone the repo

```bash
git clone git@github.com:henriblancke/snowalert-helm.git
```

### Configure the chart

The below configuration can be changed via the `--set` flag during installation or configured by editing the `values.yaml` directly (clone/download the chart first).  

#### Configure how you want to expose the SnowAlert UI:

- **Ingress**: The ingress controller must be installed in the Kubernetes cluster.  
- **ClusterIP**: Exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster.
- **NodePort**: Exposes the service on each Node’s IP at a static port (the NodePort). You’ll be able to contact the NodePort service, from outside the cluster, by requesting `NodeIP:NodePort`. 
- **LoadBalancer**: Exposes the service externally using a cloud provider’s load balancer.  

#### Configure the secrets
The SnowAlert helm charts assumes a couple of secrets are generated before it is installed. Run `docker run -it snowsec/snowalert ./install`, keep your `snowalert-<SF_ACCOUNT>.envs` file and use it to set other environment variables in `values.yaml`.

##### 1. The snowflake private key
Create the secret: `kubectl create secret generic snowflake --from-literal=private_key_password=${YOUR_PRIVATE_KEY_PWD} --from-literal=private_key=${YOUR_PRIVATE_KEY}`. Replace the placeholders with your `PRIVATE_KEY_PASSWORD` and `PRIVATE_KEY` that you can find in your `snowalert-<SF_ACCOUNT>.envs` file. Set `private_key_password=""` if `PRIVATE_KEY_PASSWORD` is empty empty in your `.envs` file (the key needs to be in the secret).

##### 2. Jira secret (optional)
Create the secret: `kubectl create secret generic jira --from-literal=api_key=${JIRA_API_KEY}`. Replace the placeholders with what you find in your `snowalert-<SF_ACCOUNT>.envs` file.

##### 3. Slack secret (optional)
Create the secret: `kubectl create secret generic slack --from-literal=api_token=${SLACK_API_TOKEN}`. Replace the placeholders with what you find in your `snowalert-<SF_ACCOUNT>.envs` file.

### Install the chart

Install the SnowAlert helm chart with release name `my-release`:

```bash
helm install --name my-release ./snowalert-helm
```

## Remove Chart

To uninstall/delete the `my-release` deployment:

```bash
helm delete --purge my-release
```

## Configuration

The following options are supported. See values.yaml for more detailed documentation and examples:

| Parameter                                   | Description                                            | Default                  |
|---------------------------------------------|------------------------------------------------------- |--------------------------|
| **webui**                                   |                                                        |                          |
| `webui.image.repository`                    | The SnowAlert webui docker image                       | `snowsec/snowalert-webui`|
| `webui.image.tag`                           | The SnowAlert webui docker image tag                   | `1.8.0-rc`               |
| `webui.service.type`                        | How you want to expose the ui                          | `ClusterIP`              |
| `webui.service.port`                        | The port you want to expose the ui on                  | `80`                     |
| `webui.ingress`                             | Configuration to expose ui through ingress             | `not enabled`            |
| `webui.resources`                           | Resource configurations for SnowAlert webui pods       | `no limits`              |
| `webui.annotations`                         | Add custom annotations to the webui deployment         | `{}`                     |
| `webui.nodeSelector`                        | Node selector for webui deployment                     | `{}`                     |
| `webui.tolerations`                         | Tolerations for the webui pods                         | `[]`                     |
| `webui.affinity`                            | Affinity configuration for webui pods                  | `{}`                     |
| `webui.labels`                              | Additional labels to append to webui deployment        | `{}`                     |
| **cron**                                    |                                                        |                          |
| `cron.image.repository`                     | The SnowAlert docker image                             | `snowsec/snowalert`      |
| `cron.image.tag`                            | The SnowAlert docker image tag                         | `1.8.0-rc`               |
| `cron.schedule`                             | Cron schedule for for SnowAlert job                    | `5 * * * *`              |
| `cron.resources`                            | Resource configurations for SnowAlert cronjob          | `no limits`              |
| `cron.annotations`                          | Add custom annotations to the cronjob                  | `{}`                     |
| `cron.nodeSelector`                         | Node selector for the cronjob                          | `{}`                     |
| `cron.tolerations`                          | Tolerations for the cronjob pods                       | `[]`                     |
| `cron.affinity`                             | Affinity configuration for cronjob pods                | `{}`                     |
| `cron.labels`                               | Additional labels to append to cronjob yaml            | `{}`                     |
| **credentials**                             |                                                        |                          |
| `snowflake.snowflake_account`               | Your snowflake account number                          | `vpXXXX`                 |
| `snowflake.sa_user`                         | The snowflake user you want to use for SnowAlert       | `null`                   |
| `snowflake.sa_role`                         | The snowflake role you want to use for SnowAlert       | `null`                   |
| `snowflake.sa_database`                     | The database you want to use for SnowAlert             | `snowalert`              |
| `snowflake.sa_warehouse`                    | The warehouse you want to use for SnowAlert            | `snowalert`              |
| `private_key_secret`                        | Reference to the `name` and `key` of the k8s secret    | `null`                   |
| `private_key_secret_password`               | Reference to the `name` and `key` of the k8s secret    | `null`                   |
| `jira.enabled`                              | Integrate jira with SnowAlert                          | `false`                  |
| `jira.url`                                  | Your jira url                                          | `https://jira.atlassian.net` |
| `jira.user`                                 | The jira user you created for SnowAlert                | `snowalert`              |
| `jira.project`                              | The jira project SnowAlert will create tickets in      | `SA`                     |
| `jira.api_key_secret`                       | Reference to the `name` and `key` of the k8s secret    | `jira|api_key`           |
| `slack.enabled`                             | Enable SnowAlert slack integration                     | `false`                  |
| `slack.api_token_secret`                    | Reference to the `name` and `key` of the k8s secret    | `slack|api_token`        |
| **security**                                |                                                        |                          |
| `rbac.create`                               | Specifies whether RBAC resources should be created     | `true`                   |
| `rbac.apiVersion`                           | RBAC api version (v1, v1beta1, or v1alpha1)            | `v1`                     |
| `serviceAccount.create`                     | Specifies whether a ServiceAccount should be created   | `true`                   |
| `serviceAccount.name`                       | The name of the ServiceAccount to use.                 | `null`                   |

