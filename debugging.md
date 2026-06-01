# Nomad & Consul CLI Debugging Notes

## 1. Check Nomad Server Members

Command:

```bash
nomad server members
```

Purpose:

* Displays all Nomad servers in the cluster.
* Shows leader and follower nodes.

When to Use:

* Verify cluster formation.
* Check leader election.
* Confirm all servers are online.

Example Output:

```text
Name                     Status  Leader
node1                    alive   true
node2                    alive   false
node3                    alive   false
```

Key Observation:

* Only one leader should exist.
* Remaining servers should be followers.
* All nodes should show alive.

---

## 2. Check Nomad Raft Peers

Command:

```bash
nomad operator raft list-peers
```

Purpose:

* Shows Raft consensus members.

When to Use:

* Leader election issues.
* Cluster quorum verification.

Key Observation:

* One node must be leader.
* Others should be followers.
* All should be voters.

---

## 3. Check Client Nodes

Command:

```bash
nomad node status
```

Purpose:

* Lists worker/client nodes.

When to Use:

* Verify nodes are available for scheduling.

Key Observation:

* Status should be ready.
* Eligibility should be eligible.

---

## 4. Inspect Specific Node

Command:

```bash
nomad node status <node-id>
```

Purpose:

* Shows detailed node information.

Useful Information:

* CPU
* Memory
* Drivers
* Running allocations

---

## 5. View Jobs

Command:

```bash
nomad job status
```

Purpose:

* Lists all jobs running in the cluster.

---

## 6. View Specific Job

Command:

```bash
nomad job status my-job1
```

Purpose:

* Shows deployment health.
* Shows allocation IDs.

Key Observation:

* Running count should match desired count.

---

## 7. Inspect Allocation

Command:

```bash
nomad alloc status <allocation-id>
```

Purpose:

* Shows exact task execution details.

Useful For:

* Failed jobs
* Scheduling issues
* Restart loops

---

## 8. View Allocation Logs

Command:

```bash
nomad alloc logs <allocation-id>
```

Purpose:

* Application logs.

Most Important Debugging Command.

---

## 9. Enter Running Allocation

Command:

```bash
nomad alloc exec -task job1-nginx-task <alloc-id> /bin/sh
```

Purpose:

* Enter container running under Nomad.

Use Cases:

* Verify files
* Verify ports
* Test application

---

## 10. Check Consul Members

Command:

```bash
consul members
```

Purpose:

* Lists Consul agents.

Key Observation:

* All nodes should be alive.

---

## 11. Check Consul Leader

Command:

```bash
consul operator raft list-peers
```

Purpose:

* Shows Consul leader.

Key Observation:

* One leader.
* Remaining followers.

---

## 12. View Registered Services

Command:

```bash
consul catalog services
```

Purpose:

* Shows services registered in Consul.

Example:

```text
consul
nginx-service
nomad
```

---

## 13. Check Service Instances

Command:

```bash
consul catalog nodes -service nginx-service
```

Purpose:

* Shows where service is running.

---

## 14. Verify DNS Resolution

Command:

```bash
dig @127.0.0.1 -p 8600 nginx-service.service.consul
```

Purpose:

* Verify Consul service discovery.

Expected Result:

```text
nginx-service.service.consul IN A 172.31.x.x
```

---

## 15. Check Docker Containers

Command:

```bash
sudo docker ps
```

Purpose:

* Verify actual container is running.

---

## 16. View Nomad Logs

Command:

```bash
sudo journalctl -u nomad -f
```

Purpose:

* Troubleshoot Nomad service.

---

## 17. View Consul Logs

Command:

```bash
sudo journalctl -u consul -f
```

Purpose:

* Troubleshoot Consul service.

```
```
