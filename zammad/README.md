# Zammad Helm Chart

This directory contains a Kubernetes chart to deploy [Zammad](https://zammad.org/) ticket system.

## Prerequisites Details

- Kubernetes 1.8+
- Cluster with at least 4GB of free RAM

## Chart Details

This chart will do the following:

- Install Zammad statefulset
- Install Elasticsearch, Memcached & PostgreSQL as requirements

## Installing the Chart

To install the chart use the following:

```console
helm repo add zammad https://zammad.github.io
helm upgrade --install zammad zammad/zammad --namespace zammad
```

## Configuration

The following table lists the configurable parameters of the zammad chart and their default values.

| Parameter                                          | Description                                      | Default                         |
| -------------------------------------------------- | ------------------------------------------------ | ------------------------------- |
| `image.repository`                                 | Container image to use                           | `zammad/zammad-docker-compose`  |
| `image.tag`                                        | Container image tag to deploy                    | `3.3.0-19`                      |
| `image.pullPolicy`                                 | Container pull policy                            | `IfNotPresent`                  |
| `service.type`                                     | Service type                                     | `ClusterIP`                     |
| `service.port`                                     | Service port                                     | `8080`                          |
| `ingress.enabled`                                  | Enable Ingress                                   | `false`                         |
| `ingress.annotations`                              | Additional ingress annotations                   | `""`                            |
| `ingress.path`                                     | Ingress path                                     | `""`                            |
| `ingress.hosts`                                    | Ingress hosts                                    | `""`                            |
| `ingress.tls`                                      | Ingress TLS                                      | `[]`                            |
| `envConfig.elasticsearch.schema`                   | Elasticsearch schema                             | `http`                          |
| `envConfig.elasticsearch.host`                     | Elasticsearch host                               | `zammad-master`                 |
| `envConfig.elasticsearch.port`                     | Elasticsearch port                               | `9200`                          |
| `envConfig.elasticsearch.user`                     | Elasticsearch user                               | `""`                            |
| `envConfig.elasticsearch.pass`                     | Elasticsearch pass                               | `""`                            |
| `envConfig.memcached.host`                         | Memcached host                                   | `zammad-memcached`              |
| `envConfig.memcached.port`                         | Memcached port                                   | `11211`                         |
| `envConfig.postgresql.host`                        | PostgreSql host                                  | `zammad-postgresql`             |
| `envConfig.postgresql.port`                        | PostgreSql port                                  | `5432`                          |
| `envConfig.postgresql.pass`                        | PostgreSql pass                                  | `""`                            |
| `envConfig.postgresql.user`                        | PostgreSql user                                  | `zammad`                        |
| `envConfig.postgresql.db`                          | PostgreSql database                              | `zammad_production`             |
| `autoWizard.enabled`                               | enable autowizard                                | `false`                         |
| `autoWizard.config`                                | autowizard json config                           | `""`                            |
| `persistence.enabled`                              | Enable persistence                               | `true`                          |
| `persistence.accessModes`                          | Access modes                                     | `["ReadWriteOnce"]`             |
| `persistence.size`                                 | Volume size                                      | `15Gi`                          |
| `persistence.storageClass`                         | storage class                                    | `""`                            |
| `persistence.annotations`                          | annotations                                      | `{}`                            |
| `resources.nginx`                                  | Resource usage of Zammad's nginx container       | `{}`                            |
| `resources.railsserver`                            | Resource usage of Zammad's railsserver container | `{}`                            |
| `resources.scheduler`                              | Resource usage of Zammad's scheduler container   | `{}`                            |
| `resources.websocket`                              | Resource usage of Zammad's websocket container   | `{}`                            |
| `nodeSelector`                                     | Node Selector                                    | `{}`                            |
| `tolerations`                                      | Tolerations                                      | `[]`                            |
| `affinity`                                         | Affinity                                         | `{}`                            |
| `elasticsearch.enabled`                            | Use Elasticsearch dependency                     | `true`                          |
| `elasticsearch.image`                              | Elasticsearch docker image                       | `zammad/zammad-docker-compose`  |
| `elasticsearch.imageTag`                           | Elasticsearch docker image tag                   | `zammad-elasticsearch-3.3.0-19` |
| `elasticsearch.clusterName`                        | Elasticsearch cluster name                       | `zammad`                        |
| `elasticsearch.replicas`                           | Elasticsearch replicas                           | `1`                             |
| `elasticsearch.clusterHealthCheckParams`           | Workaround to get ES test work in GitHubCI       | `"timeout=1s"`                  |
| `memcached.enabled`                                | Use Memcached dependency                         | `true`                          |
| `memcached.replicaCount`                           | Memcached replicas                               | `1`                             |
| `postgresql.enabled`                               | Use PostgreSQL dependency                        | `true`                          |
| `postgresql.postgresqlUsername`                    | PostgreSQL user                                  | `zammad`                        |
| `postgresql.postgresqlPassword`                    | PostgreSQL password                              | `zammad`                        |
| `postgresql.postgresqlDatabase`                    | PostgreSQL DB                                    | `zammad_production`             |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

### Important note for  NFS filesystems

For persistent volumes, NFS filesystems should work correctly for **Elasticsearch** and **PostgreSQL**; however, errors will occur if Zammad itself uses an NFS-based persistent volume.  Websockets will break completely.  This is particularly bad news for receiving notifications from the application and using the Chat module.

Don't use an NFS-based storage class for Zammad's persistent volume.

This is relevant to **EFS** for AWS users, as well.

## Using zammad

Once the zammad pod is ready, it can be accessed using the ingress or port forwarding. 
To use port forwarding:

```console
kubectl -n zammad port-forward service/zammad 8080
```

Open your browser on <http://localhost:8080>

## Upgrading

### From chart version 1.x

This has changed:
- requirement chart condition variable name was changed
- the lables have changed
- the persistent volume claim was changed to persistent volume claim template
  - import your filebackup here
- all requirement charts has been updated to the latest versions
  - Elasticsearch
    - docker image was changed to elastic/elasticsearch
    - version was raised from 5.6 to 7.6
    - reindexing will be done automatically
  - Postgres
    - bitnami/postgresql chart is used instead of stable/postgresql
    - version was raised from 10.6.0 to 11.7.0
    - there is no automated upgrade path
    - you have to import a backup manually
  - Memcached
    - bitnami/memcached chart is used instead of stable/memcached
    - version was raised from 1.5.6 to 1.5.22
    - nothing to do here

**Before the update backup Zammad files and make a PostgreSQL backup, as you will need these backups later!**

- If your helm release was named "zammad" and also installed in the namespace "zammad" like:

```
helm upgrade --install zammad zammad/zammad --namespace=zammad --version=1.2.1
```

- Do the upgrade like this:

```
helm delete --purge zammad
kubectl -n zammad delete pvc data-zammad-postgresql-0 data-zammad-elasticsearch-data-0 data-zammad-elasticsearch-master-0
helm upgrade --install zammad zammad/zammad --namespace=zammad --version=2.0.3
```

- Import your file and SQL backups inside the Zammad & Postgresql containers
