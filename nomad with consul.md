# Nomad + Consul Integration with ACL Authentication

## Overview

This project demonstrates:

* Deployment of a 3-node Nomad cluster
* Deployment of a 3-node Consul cluster
* Integration of Nomad with Consul
* Service registration and service discovery using Consul DNS
* Securing Nomad UI and CLI using ACL Tokens

---

# Architecture

```mermaid
+-----------------------------------------------------+
| Node-1 (172.31.4.101)                              |
| Nomad Server + Client                              |
| Consul Server                                      |
+-----------------------------------------------------+

+-----------------------------------------------------+
| Node-2 (172.31.5.251)                              |
| Nomad Server + Client                              |
| Consul Server                                      |
+-----------------------------------------------------+

+-----------------------------------------------------+
| Node-3 (172.31.10.174)                             |
| Nomad Server + Client                              |
| Consul Server                                      |
+-----------------------------------------------------+
```

---

# Prerequisites

* Ubuntu 24.04
* Docker Installed
* Nomad Installed
* Consul Installed
* Ports Opened Between Nodes

## Nomad Ports

| Port | Purpose     |
| ---- | ----------- |
| 4646 | HTTP API/UI |
| 4647 | RPC         |
| 4648 | Serf        |

## Consul Ports

| Port | Purpose    |
| ---- | ---------- |
| 8500 | UI/API     |
| 8600 | DNS        |
| 8300 | Server RPC |
| 8301 | LAN Gossip |
| 8302 | WAN Gossip |

---

# Step 1: Configure Consul Cluster

## Consul Configuration

File:

```bash
/etc/consul.d/consul.hcl
```

Example:

```hcl
datacenter = "dc1"

data_dir = "/opt/consul"

bind_addr = "172.31.4.101"

advertise_addr = "172.31.4.101"

client_addr = "0.0.0.0"

server = true

bootstrap_expect = 3

retry_join = [
  "172.31.4.101",
  "172.31.5.251",
  "172.31.10.174"
]

ui_config {
  enabled = true
}
```

Replace the IP address accordingly on each node.

---

## Validate Configuration

```bash
sudo consul validate /etc/consul.d/
```

Expected Output:

```bash
Configuration is valid!
```

---

## Start Consul

```bash
sudo systemctl enable consul

sudo systemctl restart consul

sudo systemctl status consul
```

---

## Verify Cluster

```bash
consul members
```

```bash
consul operator raft list-peers
```

Expected:

```text
1 Leader
2 Followers
```

---

# Step 2: Configure Nomad to Use Consul

File:

```bash
/etc/nomad.d/nomad.hcl
```

Add:

```hcl
consul {
  address = "127.0.0.1:8500"
}
```

Example:

```hcl
data_dir = "/opt/nomad/data"

bind_addr = "0.0.0.0"

advertise {
  http = "{{ GetPrivateIP }}"
  rpc  = "{{ GetPrivateIP }}"
  serf = "{{ GetPrivateIP }}"
}

server {
  enabled = true
  bootstrap_expect = 3

  retry_join = [
    "172.31.4.101",
    "172.31.5.251",
    "172.31.10.174"
  ]
}

client {
  enabled = true
}

consul {
  address = "127.0.0.1:8500"
}
```

---

## Restart Nomad

```bash
sudo systemctl restart nomad

sudo systemctl status nomad
```

---

# Step 3: Deploy Sample Nginx Service

Create job file:

```bash
vi nginx.nomad
```

```hcl
job "my-job1" {

  datacenters = ["dc1"]

  type = "service"

  group "group-job1" {

    count = 1

    network {
      port "http" {
        to = 80
      }
    }

    service {
      name = "nginx-service"
      port = "http"
      provider = "consul"
    }

    task "job1-nginx-task" {

      driver = "docker"

      config {
        image = "nginx:alpine"
        ports = ["http"]
      }

      resources {
        cpu    = 200
        memory = 128
      }
    }
  }
}
```

Deploy Job:

```bash
nomad job run nginx.nomad
```

---

# Step 4: Verify Job Deployment

List Jobs:

```bash
nomad job status
```

Check Job:

```bash
nomad job status my-job1
```

Check Allocation:

```bash
nomad alloc status <allocation-id>
```

Example:

```bash
nomad alloc status caa98fb0
```

---

# Step 5: Verify Docker Container

Login to the node where the allocation is running.

```bash
sudo docker ps
```

Expected:

```text
nginx:alpine container running
```

---

## Access Container

```bash
nomad alloc exec \
-task job1-nginx-task \
<allocation-id> \
/bin/sh
```

Example:

```bash
nomad alloc exec \
-task job1-nginx-task \
caa98fb0 \
/bin/sh
```

Verify:

```bash
hostname

nginx -v
```

---

# Step 6: Verify Service Registration

List Consul Services:

```bash
consul catalog services
```

Expected:

```text
consul
nomad
nginx-service
```

Check Service:

```bash
consul catalog nodes -service nginx-service
```

---

# Step 7: Verify Consul DNS

Query Service DNS:

```bash
dig @127.0.0.1 -p 8600 nginx-service.service.consul
```

Expected:

```text
nginx-service.service.consul. 0 IN A 172.31.10.174
```

Alternative:

```bash
nslookup nginx-service.service.consul 127.0.0.1
```

---

# Step 8: Enable Nomad ACL

Edit Nomad Configuration:

```bash
sudo vi /etc/nomad.d/nomad.hcl
```

Add:

```hcl
acl {
  enabled = true
}
```

---

## Restart Nomad

```bash
sudo systemctl restart nomad
```

---

# Step 9: Bootstrap ACL

Run from Leader Node:

```bash
nomad acl bootstrap
```

Example Output:

```text
Accessor ID = xxxxx

Secret ID = yyyyy

Name = Bootstrap Token

Type = management
```

Save the Secret ID securely.

---

# Step 10: Authenticate CLI

Export Token:

```bash
export NOMAD_TOKEN=<SECRET_ID>
```

Verify:

```bash
nomad node status
```

Expected:

```text
Cluster information displayed successfully
```

---

# Step 11: Verify ACL Enforcement

Remove Token:

```bash
unset NOMAD_TOKEN
```

Test:

```bash
nomad node status
```

Expected:

```text
Permission denied
```

---

# Step 12: Verify UI Authentication

Access Nomad UI:

```text
http://<PUBLIC-IP>:4646
```

Expected:

```text
ACL Token Required
```

Provide:

```text
Bootstrap Secret ID
```

After successful login:

* Servers
* Clients
* Jobs
* Allocations

will be accessible.

---

# Validation Results

| Validation               | Status |
| ------------------------ | ------ |
| 3 Node Nomad Cluster     | ✅      |
| Nomad Leader Election    | ✅      |
| Nginx Deployment         | ✅      |
| Docker Container Running | ✅      |
| Consul Cluster Formation | ✅      |
| Nomad-Consul Integration | ✅      |
| Service Registration     | ✅      |
| DNS Service Discovery    | ✅      |
| ACL Enabled              | ✅      |
| UI Authentication        | ✅      |
| CLI Authentication       | ✅      |

---

# Conclusion

Successfully implemented a highly available 3-node Nomad cluster integrated with a 3-node Consul cluster. Consul provides service registration and DNS-based service discovery for Nomad workloads. ACL authentication secures Nomad UI and CLI access through access tokens, ensuring controlled and authorized cluster operations.
