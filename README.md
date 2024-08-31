# Redis Cluster Watcher Deployment

This repository contains Kubernetes manifests for deploying a Redis Cluster Watcher on a Kubernetes cluster. The Redis Cluster Watcher is responsible for monitoring the Redis cluster, handling node failures, adding new pods to the cluster, and managing replicas, without using persistent volumes.

## Overview

The Redis Cluster Watcher continuously monitors the status of Redis nodes within the cluster. It performs the following actions:

- Identifies and removes failed nodes from the Redis cluster.
- Detects new Redis pods that are not part of the cluster and adds them.
- Manages replica assignment to ensure optimal data distribution and fault tolerance.

The setup is designed to run Redis in a non-persistent mode, meaning it does not use any persistent volumes (PV) or persistent volume claims (PVC). This makes the deployment lightweight and suitable for environments where data persistence is not required or handled externally.

## Deployment Components

### 1. Deployment

The `Deployment` resource in `redis-cluster-watcher-deployment.yaml` creates a single replica pod of the Redis Cluster Watcher. The pod uses a Docker image from the Bitnami repository, and the main script is executed in a loop every 60 seconds.

- **Namespace**: `redis-cluster`
- **Image**: `bitnami/redis-cluster:7.2.5-debian-12-r4`
- **Command**: Runs a custom Bash script to manage cluster nodes and replicas.

### 2. ConfigMap

The `ConfigMap` named `redis-cluster-recovery-script` contains the Bash script `redis-cluster-recovery.sh`, which is the main script executed by the watcher pod. This script performs the following tasks:

- Logs the current state of the cluster and any errors.
- Detects failed Redis nodes and attempts to remove them from the cluster.
- Finds new Redis pod IPs that are not yet part of the cluster and adds them.
- Assigns replicas to Redis master nodes that do not have any replicas assigned.

## Redis Configuration for Non-Persistent Volumes

The Redis cluster is configured to operate without persistent storage. This is achieved by setting specific environment variables and configuration options within the Helm chart values or Kubernetes manifests:

```yaml
redis:
  useAOFPersistence: "no"
  extraEnvVars:
    - name: REDIS_RDB_POLICY_DISABLED
      value: "yes"
```

### Key Configuration Options

- **useAOFPersistence**: Set to `"no"` to disable the Append-Only File (AOF) persistence.
- **REDIS_RDB_POLICY_DISABLED**: Set to `"yes"` to disable RDB (Redis Database Backup) persistence.

These settings ensure that Redis does not write any data to disk, providing a fully in-memory database experience.

## How to Deploy

1. Apply into redis-cluster namespace, deployment, and config map:

   ```bash
   kubectl apply -f redis-cluster-watcher-deployment.yaml
   ```

2. Monitor the Redis Cluster Watcher logs to ensure it is functioning correctly:

   ```bash
   kubectl logs -f deployment/redis-cluster-watcher -n redis-cluster
   ```

3. Confirm that the Redis cluster is running without persistent volumes by checking the Redis configuration and pod statuses.

## Conclusion

This deployment setup provides a Redis cluster that operates entirely in-memory, suitable for stateless applications or environments where data persistence is not a requirement. The Redis Cluster Watcher automates cluster maintenance tasks, ensuring a robust and fault-tolerant Redis cluster deployment.