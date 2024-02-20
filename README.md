# Atlas Helm Charts
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/atlas)](https://artifacthub.io/packages/search?repo=atlas)

## Overview

Welcome to Atlas the Universal Helm Chart repository! This Helm Chart is designed to provide a flexible and customizable solution for deploying applications on Kubernetes. It supports various features, including custom volume mounting, secret mounting, multiple ingress hosts and paths, and more.

## Features

- **Custom Volume Mounting:** Easily attach custom volumes to your pods for persistent data storage.

- **Secret Mounting:** Securely inject secrets into your application through Kubernetes secrets.

- **Multiple Ingress Configuration:** Define multiple ingress hosts and paths for better routing and accessibility.

- **Configurable Options:** Fine-tune your deployment with a wide range of customizable options and parameters.

- **Scalability:** Scale your application horizontally by adjusting replica counts and resource allocations.

- **Compatibility:** Works seamlessly with different Kubernetes distributions and cloud providers.

## Prerequisites

- [Helm](https://helm.sh/docs/intro/install/) installed on your Kubernetes cluster.

## Installation

```bash
helm repo add atlas https://privyinfra.github.io/atlas/stable/  
helm repo update
helm install my-atlas atlas/atlas [--version=1.0.0]
```

## Configuration

The Helm Chart provides extensive configuration options to tailor the deployment to your specific needs. Review the [`values.yaml`](./charts/universal-chart/values.yaml) file for a comprehensive list of available parameters.

Example:

```bash
helm install my-atlas  atlas/atlas -f my-values.yaml
```

## Usage

Describe how users can customize and use your Helm Chart in their own projects. Provide examples and best practices for configuration.
### Parameters
#### Common Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|nameOverride|Override helm chart name|atlas|
|fullnameOverride|Override helm chart name|atlas|
|namespace|The namespace in which the App will be created|""|
|replicaCount|Number of App replicas to deploy|1|
|volumes|mount sa / pvc to container, sometimes pair with volumeMounts|[`detail`](#volumes-config-example)|
|volumeMounts|mount volume to destination path|[`detail`](#volumeMounts-config-example)|
|hostAliases|add hostaliases for dns local|[`detail`](#hostAliases-config-example)|


#### gracePeriodSeconds Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|gracePeriodSeconds.enabled|Enable grace period second |true|
|gracePeriodSeconds.value|Amount of time before pod killed while on not ready state |60|

#### secrets Configuration Parameters
Secret management
|Name|Description|Value|
| ------ | ------ | ------ | 
|secrets.configs.enabled|Enable config|false|
|secrets.configs.path|Path config|secrets/configs/atlas-example|
|secrets.configs.autorestart.enabled|Enable config auto restart|false|
|secrets.creds.enabled|Enable creds|false|
|secrets.creds.path|Path config|secrets/creds/atlas-example|
|secrets.creds.autorestart.enabled|Enable config|false|


#### liveness Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|liveness.port |liveness port|8000|
|liveness.type |type of liveness (httpGet | exec | tcpSocket)|tcpSocket|
|liveness.path |declare liveness path for grpc (exec) and http (httpGet) only|/ping|
|liveness.initialDelay|Amount of time before start check|30|
|liveness.periodSeconds|Amount of time use to check|10|
|liveness.successThreshold|Minimum successes to be considered successful after having failed|1|
|liveness.failureThreshold|Threshold amount to set container not ready|3|

#### service Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|service.type |type of service|ClusterIP|
|service.configs|config for service|[`detail`](#service-config-example)|
|service.headless |enable headless if app service use GRPC protocol|false|

#### imagePullSecrets Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|imagePullSecrets.name |secret name to pull image from gcr |"gcr-json-key"|


#### image Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|image.pullPolicy |policy when pull image |"IfNotPresent"|
|image.repository | repository url |""|
|image.tag |app image tag |""|

#### interceptor Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|interceptor.enabled |enable interceptor |false|
|interceptor.args |command that execute when container run |"echo 'test-interceptor'" |

#### migrations Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|migrations.enabled |enable migrations |false|
|migrations.type |type of migration use init-container or pre-upgrade-job |init-container |
|migrations.args |command that execute when migration run |"cd /opt/atlas ; envsubst < config/app.yaml.dist > config/app.yaml; echo 'Initializing Service...'"|

#### autoscaling Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|autoscaling.enabled |enabled autoscaling |false|
|autoscaling.minReplicas |replica minimal when less trrafic |1|
|autoscaling.maxReplicas |replica maximal when high trrafic  |2|
|autoscaling.averageUtilMem |amount of memory to trigger scaling |80|
|autoscaling.averageUtilCpu |amount of cpu to trigger scaling |80|


#### resources Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|resources.limits.cpu |limit cpu usage  |400m|
|resources.limits.memory |limit memory usage  |256Mi|
|resources.requests.cpu |set request cpu |300m|
|resources.requests.memory |set request memory |192Mi|

#### ingress Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|ingress.enabled |enabled ingress |false|
|ingress.config |ingress config |[`datail`](#ingress-config-example)|
|ingress.tls |ingress tls config |[`datail`](#ingress-tls-config-example)|


#### annotations Configuration Parameters
|Name|Description|Value|
| ------ | ------ | ------ | 
|annotations.ingress |add annotation for ingress|[`detail`](#annotation-ingress-config-example)|
|annotations.deployment |add annotation for deployment||
|annotations.hpa |add annotation for hpa||
|annotations.preupgradejob |add annotation for preupgradejob|[`detail`](#annotation-pre-upgrade-job-config-example)|
|annotations.serviceaccount |add annotation for serviceaccount||
|annotations.vault |add annotation for vault|[`detail`](#annotation-vault-config-example)|


### Parameter Detail
#### volumes config example
```
- name: service-account
  secret:
    secretName: atlas-sa
- name: data
  persistentVolumeClaim:
    claimName: volumes-pvc
```
#### volumeMounts config example
```
- name: service-account
  mountPath: /mnt
- name: data
  mountPath: /data
```
#### service config example
```
- name: http
  port: 80
  targetPort: 8000
  protocol: TCP
- name: grpc
  port: 80
  targetPort: 8000
  protocol: TCP
- name: http
  port: 3000
  targetPort: 3000
  protocol: TCP
```

#### ingress config example
```
- ingressName: "internal"
  className: "internal-nginx"
  rules:
    - host: atlas.privyinfra.id
      http:
        paths:
          - path: /rc(?:\/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: rc-atlas
                port:
                  number: 80
- ingressName: "b2b-ingress"
  className: "internal-nginx"
  rules:
    - host: atlas.privyinfra.id
      http:
        - path: /(.*)
          pathType: Prefix
          backend:
            service:
              name: atlas
              port:
```
#### ingress tls config example
```
tls: []
 - secretName: chart-example-tls
   hosts:
     - chart-example.local
```

#### annotation ingress config example
```
"nginx.ingress.kubernetes.io/proxy-body-size": "10M"
"nginx.ingress.kubernetes.io/use-regex": "true"
"nginx.org/client-max-body-size": "10M"
```
#### annotation pre upgrade job config example
```
"helm.sh/hook": pre-install,pre-upgrade
"helm.sh/hook-weight": "-5"
"helm.sh/hook-delete-policy": hook-succeeded
```

#### annotation vault config example
```
"helm.sh/hook": pre-install,pre-upgrade
"helm.sh/hook-weight": "-10"
```

#### hostAliases config example
```
- ip: "8.8.8.8"
  hostnames:
  - "google.com"
  - "www.google.com"
- ip: "1.1.1.1"
  hostnames:
  - "cloudflare.com"
```

### Example Deployments

```yaml
# Example values.yaml
image:
  repository: "atlas-example" 
  tag: "stable"
fullnameOverride: "atlas-example"
nameOverride: "atlas-example"
namespace: atlas
replicaCount: 1
service:
  type: ClusterIP
  configs:
    - name: http
      port: 80
      targetPort: 6969
      protocol: TCP
    - name: grpc
      port: 4054
      targetPort: 4053
      protocol: TCP
  headless: true
liveness:
  type: exec
  port: 4054
migrations:
  enabled: true
  type: init-container
  args:
  - "cd /go/src/atlas-example && ./main db:migrate"
secrets:
  configs:
    enabled: true
    autorestart:
      enabled: true
    path: secrets/configs/atlas-example
  creds:
    enabled: true
    autorestart:
      enabled: true
    path: secrets/creds/atlas-example
volumes:
  - name: service-account
    secret:
      secretName: atlas-example-sa
volumeMounts:
  - name: service-account
    mountPath: /mnt
resources: 
  limits:
    cpu: 300m
    memory: 500Mi
  requests:
    cpu: 200m
    memory: 384Mi
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 2
  averageUtilMem: 80
  averageUtilCpu: 80
ingress:
  enabled: true
  configs:
  - ingressName: "internal-ingress"
    className: "internal-nginx"
    rules:
      - host: atlas.geekinthecloud.xyz
        http:
          paths:
            - path: /rc(?:\/|$)(.)
              pathType: Prefix
              backend:
                service:
                  name: rc-atlas
                  port:
                    number: 80
  - ingressName: "external-ingress"
    className: "internal-nginx"
    rules:
      - host: atlas.geekinthecloud.xyz
        http:
          - path: /(.)
            pathType: Prefix
            backend:
              service:
                name: atlas
                port:
                  number: 80
```

## Contributing

We welcome contributions! If you find a bug or have an idea for improvement, please open an issue or submit a pull request.

## License

This project is licensed under the [MIT License](./LICENSE).
```

Remember to replace the placeholder URLs, names, and images with your actual details. Customize the sections as needed for your specific Helm Chart.
