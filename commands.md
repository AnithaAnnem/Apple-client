# Nomad + Consul Integration Command Reference Sheet

## Nomad Cluster Commands

### Check Nomad Service Status

```bash
sudo systemctl status nomad
```

### Restart Nomad

```bash
sudo systemctl restart nomad
```

### Verify Nomad Version

```bash
nomad version
```

---

## Cluster Verification

### View Server Members

```bash
nomad server members
```

### Verify Raft Leader

```bash
nomad operator raft list-peers
```

### View Client Nodes

```bash
nomad node status
```

### View Detailed Node Information

```bash
nomad node status <node-id>
```

---

## Job Management

### Validate Job File

```bash
nomad job validate nginx.nomad
```

### Run Job

```bash
nomad job run nginx.nomad
```

### View All Jobs

```bash
nomad job status
```

### View Specific Job

```bash
nomad job status my-job1
```

### Stop Job

```bash
nomad job stop my-job1
```

### Plan Job Changes

```bash
nomad job plan nginx.nomad
```

---

## Allocation Commands

### View Allocations

```bash
nomad job status my-job1
```

### View Allocation Details

```bash
nomad alloc status <allocation-id>
```

Example:

```bash
nomad alloc status caa98fb0
```

### View Allocation Logs

```bash
nomad alloc logs <allocation-id>
```

### Follow Logs

```bash
nomad alloc logs -f <allocation-id>
```

### Execute Command Inside Allocation

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

---

## Docker Commands

### List Running Containers

```bash
sudo docker ps
```

### View All Containers

```bash
sudo docker ps -a
```

### Enter Running Container

```bash
sudo docker exec -it <container-id> /bin/sh
```

### View Container Logs

```bash
sudo docker logs <container-id>
```

### Follow Logs

```bash
sudo docker logs -f <container-id>
```

---

# Consul Commands

## Service Operations

### Check Consul Service Status

```bash
sudo systemctl status consul
```

### Restart Consul

```bash
sudo systemctl restart consul
```

### Verify Version

```bash
consul version
```

---

## Configuration Validation

### Validate Configuration

```bash
sudo consul validate /etc/consul.d/
```

---

## Cluster Verification

### View Cluster Members

```bash
consul members
```

### View Raft Peers

```bash
consul operator raft list-peers
```

### Check Leader

```bash
consul info | grep leader
```

---

## Service Discovery

### List Services

```bash
consul catalog services
```

### View Service Instances

```bash
consul catalog nodes -service nginx-service
```

### View Registered Nodes

```bash
consul catalog nodes
```

---

## DNS Verification

### Query Service DNS

```bash
dig @127.0.0.1 -p 8600 nginx-service.service.consul
```

### Using nslookup

```bash
nslookup nginx-service.service.consul 127.0.0.1
```

---

## Consul API

### List Services

```bash
curl http://127.0.0.1:8500/v1/catalog/services
```

### Service Details

```bash
curl http://127.0.0.1:8500/v1/catalog/service/nginx-service
```

---

# Nomad ACL Commands

## Enable ACL

Add in nomad.hcl

```hcl
acl {
  enabled = true
}
```

Restart Nomad

```bash
sudo systemctl restart nomad
```

---

## Bootstrap ACL

```bash
nomad acl bootstrap
```

Output:

```text
Accessor ID
Secret ID
Management Token
```

---

## Export ACL Token

```bash
export NOMAD_TOKEN=<SECRET_ID>
```

Verify:

```bash
nomad node status
```

---

## Unset ACL Token

```bash
unset NOMAD_TOKEN
```

---

## List ACL Policies

```bash
nomad acl policy list
```

### View Policy

```bash
nomad acl policy info <policy-name>
```

### List Tokens

```bash
nomad acl token list
```

---

# Linux Troubleshooting Commands

### View Open Ports

```bash
sudo ss -tulpn
```

### Check Specific Port

```bash
sudo ss -tulpn | grep 4646
```

```bash
sudo ss -tulpn | grep 8500
```

### Check Running Processes

```bash
ps -ef | grep nomad
```

```bash
ps -ef | grep consul
```

---

## View Logs

### Nomad Logs

```bash
sudo journalctl -u nomad -n 100 --no-pager
```

### Follow Nomad Logs

```bash
sudo journalctl -u nomad -f
```

### Consul Logs

```bash
sudo journalctl -u consul -n 100 --no-pager
```

### Follow Consul Logs

```bash
sudo journalctl -u consul -f
```

---

# Configuration Files

### Nomad

```bash
/etc/nomad.d/nomad.hcl
```

### Consul

```bash
/etc/consul.d/consul.hcl
```

### Nomad Service

```bash
/usr/lib/systemd/system/nomad.service
```

### Consul Service

```bash
/usr/lib/systemd/system/consul.service
```

---

# Web UI Access

### Nomad UI

```text
http://<PUBLIC-IP>:4646
```

### Consul UI

```text
http://<PUBLIC-IP>:8500
```

---

# Quick Health Checks

```bash
nomad server members
```

```bash
nomad operator raft list-peers
```

```bash
nomad node status
```

```bash
consul members
```

```bash
consul operator raft list-peers
```

```bash
consul catalog services
```

```bash
sudo docker ps
```

These seven commands are usually enough to verify that the Nomad cluster, Consul cluster, service registration, and deployed workloads are healthy.
