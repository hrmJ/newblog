---
title: "A nice little shortcut for accessing a kubepod's log"
datePublished: Fri May 27 2022 08:27:09 GMT+0000 (Coordinated Universal Time)
cuid: cl3o6ksnt03x6e0nv7zprga3b
slug: a-nice-little-shortcut-for-accessing-a-kubepods-log
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/MPKQiDpMyqU/upload/v1653639896994/SmdgEtrW5.jpeg
tags: bash, kubernetes, logging

---

When developing inside a kubernetes based setup I usually have two places to look at logs for individual pods. First, since my setup uses [skaffold](skaffold.dev), I can view the combined output of all pods via the skaffold command's output. That, however, aggregates multiple pods' logs and can be a bit messy, so I tend to reach for the individual pod's logs. To do that, I usually first run

```
kubectl get pods --all-namespaces
```

and look for the name of the pod after which I run 

```
kubectl logs -f POD_NAME
```

Recently, I got tired of  going through the process of looking up individual pods' current names and came up with this handy bash script for automating the thing (note that I'm running kubernetes under microk8s, but plain kubectl should also do):


```
#!/bin/bash

PODNAME=`microk8s kubectl get pods --all-namespaces | grep $1 | awk '{print $2}'`
microk8s kubectl logs -f $PODNAME

```

saved this at `.local/bin/podlog` and now I can just run  `podlog some-pod-name`  and have a direct access at the pod's logs