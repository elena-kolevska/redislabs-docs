---
Title: Redis Enterprise Cluster Recovery for Kubernetes
description: 
weight: 40
alwaysopen: false
categories: ["Platforms"]
aliases: /rs/concepts/kubernetes/cluster-recovery
---
## Overview

When a Redis Enterprise cluster loses contact with more than half of its nodes either because of failed nodes or network split,
the cluster stops responding to client connections.
When this happens, you must recover the cluster to restore the connections.

You can also perform cluster recovery to reset cluster nodes, to troubleshoot issues, or in a case of active/passive failover.

The cluster recovery for Kubernetes automates these recovery steps:

1. Recreates a fresh Redis Enterprise cluster
1. Mounts the persistent storage with the recovery files from the original cluster to the nodes of the new cluster
1. Recovers the cluster configuration on the first node in the new cluster
1. Joins the remaining nodes to the new cluster.

{{% note %}}
To support cluster recovery, the cluster must have been [deployed using persistence]({{< relref "/platforms/kubernetes/kubernetes-persistent-volumes.md" >}}).<br>
To support database data recovery, databases must have been [configured using persistence]({{< relref "//rs/concepts/data-access/persistence.md" >}}).
{{% /note %}}

## Recovering a Cluster on Kubernetes

To recover a cluster on Kubernetes:

1. Edit the rec resource to set the clusterRecovery flag to true, run:

    ```src
    kubectl patch rec <cluster-name> --type merge --patch '{"spec":{"clusterRecovery":true}}'
    ```

1. Wait for the cluster to recover until it is in the Running state.

    To see the state of the cluster, run:

    ```src
    watch "kubectl describe rec | grep State"
    ```

1. To recover the cluster data, once the cluster is in Running state, for any cluster pod run:

    ```src
    kubeclt exec -it <pod-name> rladmin recover all
    ```

    This command recovers the data for all nodes in the cluster based on the cluster configuration in pod-0.

    If you want to recover based on the cluster configuration of another pod, copy the cluster configuration from that desired pod (/var/opt/redislabs/persist/ccs/ccs-redis.rdb) to pod-0.

1. If you are using sentinel discovery service, you must restart the sentinel_service on the master. To do this, log into the master pod and run:

    ```src
    supervisorctl restart sentinel_service
    ```
