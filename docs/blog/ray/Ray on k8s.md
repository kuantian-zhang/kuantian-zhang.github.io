---
title: Ray on k8s 在部署模版
tags:
  - Ray
  - Ray on k8s
  - k8s
---
## Ray on k8s 在部署模版

### 使用 aliyun spot eci 启动 Ray Worker
```yaml
image:
  repository: rayproject/ray
  tag: 2.34.0
  pullPolicy: Always

head:
  resources:
    limits:
      cpu: "4"
      # To avoid out-of-memory issues, never allocate less than 2G memory for the Ray head.
      memory: "8G"
    requests:
      cpu: "1"
      memory: "2G"

worker:
  groupName: workergroup
  labels:
    alibabacloud.com/eci: "true"
  annotations:
    k8s.aliyun.com/eci-spot-strategy: "SpotAsPriceGo"
    k8s.aliyun.com/eci-ram-role-name: "infra-emr"
    sidecar.istio.io/inject: "false"
#  replicas: 3
#  minReplicas: 3 
  maxReplicas: 3
  resources:
    limits:
      cpu: "2"
      memory: "4G"
```

### 使用 aliyun spot eci 启动 Ray Per Job
模版见 [aliyun spot eci 启 ray perjob 模板](template.yaml)







