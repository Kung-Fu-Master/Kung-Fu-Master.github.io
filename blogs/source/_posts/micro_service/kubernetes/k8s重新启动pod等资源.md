---
title: k8s重新启动pod等资源
tags: k8s
categories:
- microService
- kubernetes
---

## k8s重新启动deployment或daemon

<!-- more -->

```
$ kubectl rollout -h
Manage the rollout of a resource.

 Valid resource types include:

  *  deployments
  *  daemonsets
  *  statefulsets

Examples:
  # Rollback to the previous deployment
  kubectl rollout undo deployment/abc

  # Check the rollout status of a daemonset
  kubectl rollout status daemonset/foo

Available Commands:
  history     View rollout history
  pause       Mark the provided resource as paused
  restart     Restart a resource
  resume      Resume a paused resource
  status      Show the status of the rollout
  undo        Undo a previous rollout

Usage:
  kubectl rollout SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

## 实例

```
kubectl rollout restart deploy/<POD-NAME> -n <NAMESPACE>  // deploy是deployment资源的简称
```


## k8s重新启动pod
```
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -
```

