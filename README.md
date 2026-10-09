# Enterprise Zero-Downtime PostgreSQL HA Cluster

This repository demonstrates a production-grade, highly available (HA) PostgreSQL cluster architecture designed for on-premise or bare-metal environments. It simulates a complete automated failover and self-healing data center infrastructure using Docker Compose.

## 🏗️ Architecture & Tech Stack

*   **State Management & Consensus:** [etcd](https://etcd.io/) (Raft consensus algorithm)
*   **High-Availability Manager:** [Patroni](https://patroni.readthedocs.io/en/latest/) (Leader election and automated failover)
*   **Database Engine:** PostgreSQL (Zalando Spilo image)
*   **Intelligent Routing (L4 Proxy):** [HAProxy](http://www.haproxy.org/)

## 🚀 Key Features

1.  **Distributed Consensus & Split-Brain Prevention:**
    - Utilizes a 3-node `etcd` cluster as the central nervous system to maintain cluster state securely.
    - Implements strict fencing and timeline generation (TL) mechanisms to guarantee zero data corruption (Split-Brain) during network partitions.

2.  **Automated Failover (Self-Healing):**
    - `Patroni` continuously monitors database health. Upon leader failure (e.g., node crash, power loss), Patroni detects the anomaly via `etcd` and promotes a healthy replica to leader within sub-seconds.
    - Dead nodes are automatically re-bootstrapped as replicas upon recovery, seamlessly syncing missing data.

3.  **Dynamic Traffic Routing:**
    - `HAProxy` acts as a TCP load balancer, constantly polling the Patroni REST API (`/master` endpoint) to dynamically route write/read traffic to the active leader node without requiring application-level reconfiguration.

## 🛠️ How to Run & Test Chaos (Disaster Recovery)

**1. Spin up the cluster:**
```bash
docker compose up -d
```

**2. Check the cluster status:**
```bash
docker compose exec pg1 patronictl topology
```

**3.Inject chaos**

```bash
docker kill <leader_container_id>
```

**4.Test HAProxy Routing:**

```bash
psql "postgresql://postgres:superadmin@<haproxy_ip>:5432/postgres?sslmode=require"
```
