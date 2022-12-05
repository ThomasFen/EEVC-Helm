# Emotion Enabled Videoconferencing 

## Introduction

This chart bootstraps a deployment on a [Kubernetes](https://kubernetes.io/) cluster using the [Helm](https://helm.sh/) package manager with the goal of analyzing emotions of videoconference participants in real-time. For the most part, subcharts from [Bitnami](https://bitnami.com/) and the open-source community are parameterized by overriding their default values using `values.yaml`.




## Prerequisites
- Kubernetes 1.19+
- Helm 3.2.0+
- NGINX Ingress Controller 
## Installing the Chart
```
$ helm install ./ --dependency-update --generate-name 
```            
> __Tip__: Use install options like `--set redisAIProvider.faceRecognitionEnabled=false` or `--set redis.enabled=false` to configure the deployment without the need to edit `values.yaml` directly

:exclamation:After installation, you'll be notified about the need to add a [rewrite](https://coredns.io/plugins/rewrite/) to CoreDNS. Depending on the configuration, you're also presented with a new line to add to your system's hosts file.   
## Parameters 

### Subchart configuration for the authentication middleware

Overwrites some default values of Bitnami's Node.js chart. 
Referenced in [Jitsi's](subchart-configuration-for-jitsi) TOKEN_AUTH_URL. Utilizes Keycloak to provide Jitsi with JWTs.

ref: https://artifacthub.io/packages/helm/bitnami/node/18.1.17

| Name                                          | Description                                                       | Value                               |
| --------------------------------------------- | ----------------------------------------------------------------- | ----------------------------------- |
| `authentication.enabled`                      | Whether to install or not the Node.js based authentication chart  | `true`                              |
| `authentication.service.type`                 | Kubernetes Service type                                           | `ClusterIP`                         |
| `authentication.service.nodeIP`               | Node IP is needed when using service type NodePort                | `nil`                               |
| `authentication.service.loadBalancerIP`       | LoadBalancer IP is needed when using service type LoadBalancer    | `nil`                               |
| `authentication.extraEnvVarsCM`               | Name of existing ConfigMap containing extra environment variables | `fuh-realtime-emotions-auth-config` |
| `authentication.mongodb.enabled`              | Whether to install or not the MongoDB&reg; chart                  | `false`                             |
| `authentication.image.repository`             | NodeJS image repository                                           | `erge234/auth`                      |
| `authentication.image.tag`                    | NodeJS image tag (immutable tags are recommended)                 | `helm`                              |
| `authentication.image.pullPolicy`             | NodeJS image pull policy                                          | `Always`                            |
| `authentication.getAppFromExternalRepository` | Enable to download app from external git repository               | `false`                             |


### Subchart configuration of the emotionbackend

Overwrites some default values of Bitnami's Node.js chart. 
Used to run [Socket.IO](https://socket.io/). Sits between browsers running Jitsi and backend services.

ref: https://artifacthub.io/packages/helm/bitnami/node/18.1.17

| Name                                          | Description                                                       | Value                          |
| --------------------------------------------- | ----------------------------------------------------------------- | ------------------------------ |
| `emotionbackend.enabled`                      | Whether to install or not the Node.js based emotionbackend chart  | `true`                         |
| `emotionbackend.extraEnvVarsCM`               | Name of existing ConfigMap containing extra environment variables | `emotionbackend-configuration` |
| `emotionbackend.mongodb.enabled`              | Whether to install or not the MongoDB&reg; chart                  | `false`                        |
| `emotionbackend.image.repository`             | NodeJS image repository                                           | `erge234/authentication`       |
| `emotionbackend.image.tag`                    | NodeJS image tag (immutable tags are recommended)                 | `latest`                       |
| `emotionbackend.image.pullPolicy`             | NodeJS image pull policy                                          | `Always`                       |
| `emotionbackend.containerPorts.http`          | Specify the port where your application will be running           | `8010`                         |
| `emotionbackend.getAppFromExternalRepository` | Enable to download app from external git repository               | `false`                        |


### Subchart configuration of Jitsi

Overwrites some default values of the community supported 3rd party Jitsi chart

ref: https://github.com/jitsi-contrib/jitsi-helm/blob/v1.2.2/values.yaml

| Name                                           | Description                                                                                                                        | Value                                                     |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `jitsi.enabled`                                | Whether to install or not the Jitsi chart                                                                                          | `true`                                                    |
| `jitsi.enableAuth`                             | Enable authentication                                                                                                              | `true`                                                    |
| `jitsi.enableGuests`                           | Enable guest access                                                                                                                | `false`                                                   |
| `jitsi.publicURL`                              | Public URL for the web service                                                                                                     | `https://jitsi.emotion`                                   |
| `jitsi.web`                                    | Jitsi-Meet configuration                                                                                                           |                                                           |
| `jitsi.web.image.repository`                   | Jitsi image repository                                                                                                             | `erge234/web`                                             |
| `jitsi.web.image.tag`                          | Jitsi image tag (immutable tags are recommended)                                                                                   | `stable-6865-emotion`                                     |
| `jitsi.web.image.pullPolicy`                   | Jitsi image pull policy                                                                                                            | `Always`                                                  |
| `jitsi.web.extraEnvs.TOKEN_AUTH_URL`           | Redirect URL for JWT generation. Directs to [authentication middleware](#subchart-configuration-for-the-authentication-middleware) | `https://{{ .Values.global.authHostname }}/room/{room}`   |
| `jitsi.web.extraEnvs.EMOTIONBACKEND_URL`       | The URL of the [emotionbackend](#subchart-configuration-for-the-emotionbackend)                                                    | `https://{{ .Values.global.emotionbackendHostname }} `    |
| `jitsi.jvb`                                    | Jitsi Video Bridge (JVB) configuration                                                                                             |                                                           |
| `jitsi.jvb.UDPPort`                            | It may be required to change the default port to a value allowed by Kubernetes (30000-32768)                                       | `30000`                                                   |
| `jitsi.jvb.publicIP`                           | Use public IP of one of your node, or the public IP of a loadbalancer in front of the nodes                                        | `jitsi.emotion`                                           |
| `jitsi.jvb.service.type`                       | Kubernetes Service type of the JVB                                                                                                 | `NodePort`                                                |
| `jitsi.jvb.websockets.enabled`                 | Enable WebSocket support for JVB/Colibri                                                                                           | `true`                                                    |
| `jitsi.extraCommonEnvs.AUTH_TYPE`              | Select authentication type (internal, jwt or ldap)                                                                                 | `jwt`                                                     |
| `jitsi.extraCommonEnvs.JWT_APP_ID`             | Application identifier                                                                                                             | `jitsi-meet-emotion`                                      |
| `jitsi.extraCommonEnvs.JWT_APP_SECRET`         | Application secret known only to your token                                                                                        | `254uni5DFCY25hvb233bjHJBm6l34j5hb43hb3Fuy23ebwuyfMDWft2` |
| `jitsi.extraCommonEnvs.JWT_ACCEPTED_ISSUERS`   | (Optional) Set asap_accepted_issuers as a comma separated list                                                                     | `jitsi`                                                   |
| `jitsi.extraCommonEnvs.JWT_ACCEPTED_AUDIENCES` | (Optional) Set asap_accepted_audiences as a comma separated list                                                                   | `jitsi`                                                   |
| `jitsi.extraCommonEnvs.XMPP_CROSS_DOMAIN`      | Allow multiple domains on prosody                                                                                                  | `true`                                                    |


### Subchart configuration of Redis

Overwrites some default values of Bitnami's Redis-Cluster chart. 
It's intendend to use an image equipped with RedisAI and Redis Gears module.

ref: https://artifacthub.io/packages/helm/bitnami/redis-cluster/7.1.0

| Name                                     | Description                                                                            | Value                                                                                                                                                                                                            |
| ---------------------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `redis.enabled`                          | Whether to install or not the Redis chart                                              | `true`                                                                                                                                                                                                           |
| `redis.image.repository`                 | Redis image repository                                                                 | `erge234/redis-cluster`                                                                                                                                                                                          |
| `redis.image.tag`                        | Redis image tag (immutable tags are recommended)                                       | `6.2-debian-11`                                                                                                                                                                                                  |
| `redis.usePassword`                      | Use password authentication                                                            | `false`                                                                                                                                                                                                          |
| `redis.redis`                            | Redis&trade; statefulset parameters                                                    |                                                                                                                                                                                                                  |
| `redis.redis.configmap`                  | Additional Redis&trade; configuration for the nodes                                    | `loadmodule /usr/lib/redis/modules/redistimeseries.so
loadmodule /usr/lib/redis/modules/redisai.so
loadmodule /usr/lib/redis/modules/redisgears.so Plugin /var/opt/redislabs/modules/rg/plugin/gears_python.so
` |
| `redis.cluster.nodes`                    | The number of master nodes should always be >= 3, otherwise cluster creation will fail | `3`                                                                                                                                                                                                              |
| `redis.cluster.replicas`                 | Number of replicas for every master in the cluster                                     | `0`                                                                                                                                                                                                              |
| `redisAIProvider`                        | Job that loads AI models and Redis Gears after chart installation.                     |                                                                                                                                                                                                                  |
| `redisAIProvider.image.repository`       | Redis image repository                                                                 | `erge234/redis-ai-client`                                                                                                                                                                                        |
| `redisAIProvider.image.pullPolicy`       | Redis image pull policy                                                                | `Always`                                                                                                                                                                                                         |
| `redisAIProvider.faceRecognitionEnabled` | Whether to recognize the face before emotinon analysis.                                | `true`                                                                                                                                                                                                           |


### Subchart configuration of Keycloak

Overwrites some default values of Bitnami's Keycloak chart. 

ref: https://artifacthub.io/packages/helm/bitnami/keycloak/9.6.1

| Name                                                       | Description                                                                                     | Value                           |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------- |
| `keycloak.enabled`                                         | Whether to install or not the Jitsi chart                                                       | `true`                          |
| `keycloak.service.type`                                    | Kubernetes service type                                                                         | `ClusterIP`                     |
| `keycloak.service.nodeIP`                                  | Node IP is needed when using service type NodePort                                              | `nil`                           |
| `keycloak.service.loadBalancerIP`                          | LoadBalancer IP is needed when using service type LoadBalancer                                  | `nil`                           |
| `keycloak.auth`                                            | Keycloak authentication parameters                                                              |                                 |
| `keycloak.auth.adminUser`                                  | Keycloak administrator user                                                                     | `admin`                         |
| `keycloak.auth.adminPassword`                              | Keycloak administrator password for the new user                                                | `admin`                         |
| `keycloak.auth.tls.enabled`                                | Enable TLS encryption. Required for HTTPs traffic.                                              | `true`                          |
| `keycloak.auth.tls.autoGenerated`                          | Generate automatically self-signed TLS certificates. Currently only supports PEM certificates   | `true`                          |
| `keycloak.keycloakConfigCli`                               | Configuration for keycloak-config-cli                                                           |                                 |
| `keycloak.keycloakConfigCli.enabled`                       | Whether to enable keycloak-config-cli job                                                       | `true`                          |
| `keycloak.keycloakConfigCli.existingConfigmap`             | ConfigMap with keycloak-config-cli configuration. This will override `keycloakConfigCli.config` | `keycloak-config-cli-configmap` |
| `keycloak.postgresql.auth.password`                        | Password for the custom user to create                                                          | `admin`                         |
| `keycloakConfigCli.configuration.files/jitsi-emotion.yaml` | The configmap referenced in `keycloak.keycloakConfigCli.existingConfigmap`. Realm configuration | `nil`                           |


### Subchart configuration of Grafana

Overwrites some default values of Bitnami's Grafana chart.

ref: https://artifacthub.io/packages/helm/bitnami/grafana/8.1.0

| Name                                   | Description                                                                                                              | Value                                               |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| `grafana.enabled`                      | Whether to install or not the Grafana chart                                                                              | `false`                                             |
| `grafana.plugins`                      | Grafana plugins to be installed in deployment time separated by commas                                                   | `redis-app,redis-datasource,volkovlabs-image-panel` |
| `grafana.dashboardsProvider.enabled`   | Enable the use of a Grafana dashboard provider                                                                           | `true`                                              |
| `grafana.dashboardsConfigMaps`         | Array with the names of a series of ConfigMaps containing dashboards files                                               |                                                     |
| `grafana.datasources.secretDefinition` | The contents of a secret defining a custom datasource file. Only used if datasources.secretName is empty or not defined. |                                                     |
| `grafana.grafana.extraEnvVars`         | Array containing extra env vars to configure Grafana                                                                     |                                                     |


### Ingress configuration

Configure the ingress resource that allows you to access the
Fuh-Realtime-Emotions installation. Set up the URLs

ref: https://kubernetes.io/docs/user-guide/ingress/

| Name                                              | Description                                                                                                                      | Value                                                    |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `ingress.enabled`                                 | Enable ingress record generation for fuh-realtime-emotions                                                                       | `true`                                                   |
| `ingress.ingressClassName`                        | IngressClass that will be be used to implement the Ingress (Kubernetes 1.18+)                                                    | `nginx`                                                  |
| `ingress.apiVersion`                              | Force Ingress API version (automatically detected if not set)                                                                    | `""`                                                     |
| `ingress.annotations`                             | Additional annotations for the Ingress resource. To enable certificate autogeneration, place here your cert-manager annotations. | `{}`                                                     |
| `ingress.tls`                                     | Enable TLS configuration for all defined hosts (SAN certificate)                                                                 | `true`                                                   |
| `ingress.selfSigned`                              | Create a TLS secret for this ingress record using self-signed certificates generated by Helm                                     | `true`                                                   |
| `ingress.additionalHosts`                         | Ingress rules for the services that make up Fuh-Realtime-Emotions.                                                               |                                                          |
| `ingress.additionalHosts.keycloak.hostname`       | When the ingress is enabled, a host pointing to this will be created.                                                            | `keycloak.emotion`                                       |
| `ingress.additionalHosts.jitsi.hostname`          | When the ingress is enabled, a host pointing to this will be created.                                                            | `{{ (splitList "//" .Values.jitsi.publicURL) | last  }}` |
| `ingress.additionalHosts.emotionbackend.hostname` | When the ingress is enabled, a host pointing to this will be created.                                                            | `{{ .Values.global.emotionbackendHostname }}`            |
| `ingress.additionalHosts.authentication.hostname` | When the ingress is enabled, a host pointing to this will be created.                                                            | `{{ .Values.global.authHostname }}`                      |


### Global parameters  

Global authHostname and emotionbackendHostname are needed to have a single 
source of truth and for the subcharts to have access to it without the need for modifications

| Name                            | Description                                                                                    | Value                    |
| ------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------ |
| `global.authHostname`           | Determines the hostname of the authentication service in the [Ingress](#ingress-configuration) | `authentication.emotion` |
| `global.emotionbackendHostname` | Determines the hostname of the authentication service in the [Ingress](#ingress-configuration) | `emotionbackend`         |

