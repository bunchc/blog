---
title: "Kube Dashboard on Master Node"
date: 2017-01-26
layout: "post"
categories: kubernetes, k8s, raspberry pi, rPI
---

Putting this here so I don't forget.

First we need to make it so we can schedule pods to master. So, let's taint the node (thanks [Sam](https://yaple.net/)!)

```
kubectl taint nodes --all dedicated-
```

Then add this to the kubernetes-dashboard.yaml, replacing master-node for your environment.

```
    spec:
      containers:
      - name: kubernetes-dashboard
        image: gcr.io/google_containers/kubernetes-dashboard-arm:v1.5.1
*** some stuff ***
      nodeSelector:
        kubernetes.io/hostname: master-node
```

