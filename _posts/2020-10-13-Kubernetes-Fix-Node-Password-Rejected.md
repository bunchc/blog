---
title: "[Kubernetes] Fix: Node Password Rejected..."
date: 2020-10-13
layout: "post"
categories: "k3s, k8s, kubernetes, rancher, fix, troubleshooting"
---

# [Kubernetes] Fix: Node Password Rejected...

`Node password rejected, duplicate hostname or contents of...`

Judging by the error message, my Kubernetes cluster was having about the same Monday as I was.

What led here was trying to add a node back into the cluster after having rebuilt it.

## Some steps...

To fix this, we need to uninstall the `k3s-agent` from the node in question, remove the local password file on said node, and finally, remove the node entry from the primary node.

### On the node

On the node having the issue, we log in and uninstall the k3s agent and remove the local password file. As no workloads have been scheduled to the node, due to the "Node password rejected..." error, this should be non-impacting. The following commands show how to do this:

Uninstall k3s:

```shell
$ sudo /usr/local/bin/k3s-agent-uninstall.sh
```

Remove local password file:

```shell
$ sudo rm -f /etc/rancher/node/password
```

### On the primary node

Log into primary node, and check for the node in `/var/lib/rancher/k3s/server/cred/node-passwd`:

```shell
# cat /var/lib/rancher/k3s/server/cred/node-passwd
fb4544f0d6cb016cc0c261e77ac214a4,swarm-08,swarm-08,
2a08234cdcf20072b2643eb934b04080,swarm-07,swarm-07,
e107b05b351cc5284e6b6babbe87145e,swarm-04,swarm-04,
d89f5ac3a70ccdbe3da828c63285b49d,swarm-03,swarm-03,
3a1882f29f1ac1e2f4029a580aa5836e,swarm-01,swarm-01,
2b2a5926b30407a7335fe01b1e6122b1,swarm-05,swarm-05,
bf92afadf05e21ee2bc22fb147760174,swarm-09,swarm-09,
2864b53210d3785b36ee304fc163a45d,swarm-02,swarm-02,
46a16d8dc4d227258f19caa2557b4bac,swarm-06,swarm-06,

# Count the number of entries
# cat /var/lib/rancher/k3s/server/cred/node-passwd | wc -l
9
```

Next we use `sed` to test removing the line:

```shell
# sed '/swarm-02/c\' /var/lib/rancher/k3s/server/cred/node-passwd
fb4544f0d6cb016cc0c261e77ac214a4,swarm-08,swarm-08,
2a08234cdcf20072b2643eb934b04080,swarm-07,swarm-07,
e107b05b351cc5284e6b6babbe87145e,swarm-04,swarm-04,
d89f5ac3a70ccdbe3da828c63285b49d,swarm-03,swarm-03,
3a1882f29f1ac1e2f4029a580aa5836e,swarm-01,swarm-01,
2b2a5926b30407a7335fe01b1e6122b1,swarm-05,swarm-05,
bf92afadf05e21ee2bc22fb147760174,swarm-09,swarm-09,
46a16d8dc4d227258f19caa2557b4bac,swarm-06,swarm-06,
```

Looks good, but once more to be sure:

```shell
# sed '/swarm-02/c\' /var/lib/rancher/k3s/server/cred/node-passwd | wc -l
8
```

Now that we have verified our `sed` syntax, we can remove the line and restart the `k3s` service:

```shell
# sed -i '/swarm-02/c\' /var/lib/rancher/k3s/server/cred/node-passwd

# sudo systemctl restart k3s
```

Finally, we reinstall `k3s` on the node using [`k3sup`](https://github.com/alexellis/k3sup) from [Alex Ellis](https://twitter.com/alexellisuk):

```shell
$ k3sup join \
  --ip ${nodeIP} \
  --server-ip ${primaryNodeIP} \
  --user ${k3sUser} \
  --k3s-version "v1.18.9+k3s1"
```

> **Note:** I tag the specific version of k3s to use as my cluster is running on Raspberry Pi Model 2 B, which are not quite strong enough to stay super current.

Once the installation is finished, you can check that the node has indeed joined the cluster happily:

```shell
kubectl get node
NAME       STATUS                     ROLES    AGE     VERSION
swarm-03   Ready                      <none>   72d     v1.18.9+k3s1
swarm-08   Ready                      <none>   72d     v1.18.9+k3s1
swarm-06   Ready                      <none>   72d     v1.18.9+k3s1
swarm-04   Ready                      <none>   72d     v1.18.9+k3s1
swarm-01   Ready,SchedulingDisabled   master   72d     v1.18.9+k3s1
swarm-02   Ready                      <none>   7m12s   v1.18.9+k3s1
swarm-09   Ready                      <none>   72d     v1.18.9+k3s1
swarm-05   Ready                      <none>   72d     v1.18.9+k3s1
swarm-07   Ready                      <none>   68d     v1.18.9+k3s1
```

## Addendum

You may also need to use the following commands to completely eject the node:

> **Note:** These commands alone did not fix the "Node password rejected..." issue, as the `kubectl delete node` command did not clear out the node entry in `/var/lib/rancher/k3s/server/cred/node-passwd`

```shell
# Drain the node
$ kubectl drain [nodeName] --force --delete-local-data

# Delete the node
$ kubectl delete node [nodeName]
```

## Resources

* [https://stackoverflow.com/questions/11245144/replace-whole-line-containing-a-string-using-sed/11245501](https://stackoverflow.com/questions/11245144/replace-whole-line-containing-a-string-using-sed/11245501)
* [https://github.com/rancher/k3s/issues/802](https://github.com/rancher/k3s/issues/802)
