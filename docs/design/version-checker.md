# Kubernetes Version Checker

[aronchick@google.com](mailto:aronchick@google.com)<BR>
Last Updated: 2016-05-22

<!-- BEGIN MUNGE: GENERATED_TOC -->

<!-- END MUNGE: GENERATED_TOC -->

## Objective

Define a system for a Kubernetes cluster to check a defined endpoint
for the most recent version of the software, and notify users of the
opportunity to upgrade (both on the master and the nodes).

A non-goal of this proposal is to provide the method to upgrade either
the master or the nodes. This proposal only covers the notification.

## Background

### The Problem

A substantial challenge for Kubernetes administrators is to ensure their
clusters keep up with current stable releases of Kubernetes, both on the
master and on the individual nodes. While an administrator can manually
write scripts to query the API server and kubelet versions in a cluster,
and then query the Github to check the source code version, we can do better
by providing a single service built into Kubernetes that automatically checks
and alerts the administrator when their infrastructure requires an update.

### Goals
Our primary goal is to reduce the clusters are out of date with the most recent
stable version, and causing problems for customers. Customers repeatedly report
that they would update their clusters if they knew they were out of date, but 
they often miss the notifications via email or RSS that a new version is available.
This information is not available where they are - specifically interacting via
the CLI or the UI.

Our secondary goal is to make it absurdly easy for administrators to understand
the state of their cluster at any time. To do this, the Kubernetes project should
have a hosted service that enables the server pull down the most recent version,
and run a comparison with current masters and nodes. Further, we will build into
the API and CLI a simple method to report on the most recent version and whether or not
the current master and nodes are running on the latest version. 

Our third goal is to ensure that we provide this service in a way that users can 
completely trust (and do not turn off). This means we must be completely 
transparent that we are not collecting any identifying information, and that all
associated logs will be anonymized immediately.

### Non-Goals

Out of scope is enabling users to upgrade once people have been informed their nodes are out of date.

### User Experience

On cluster setup (both on GKE and k8s), the administrator is presented with a dialog as the last step that reads:

```
Would you like to have Kubernetes check for a new version once a week? 

[If yes]

If you would like to change this setting at any time, run the following command:
  kubectl config set autoupdate=off
```

### Implementation
The version checker will run as a cluster add-on. However, syntactic sugar will be provided to appear as though the system has integrated version checking in kubectl.

On midnight on Sunday (+/- rand(60 minutes)), the version checker pod will contact a hosted endpoint (e.g. http://version.k8s.io) which will contain a string (e.g. "1.2.4"). The pod will download and store that information as the "latest" version. 

Question: Should the cluster attach cluster UUID to the query? Is there any reason that this would help the system?

A user can also manually run a check by executing the following command:

```
$ kubectl check-version
[Checking with http://version.k8s.io]
Latest Version: 1.1.4

Kubernetes Master
IP: 192.168.14.5
Current Version: 1.1.3
Status: Needs Update

Kubernetes Nodes:
NAME              IP            VERSION STATUS
clstr-node-iqes   192.168.15.4  1.1.4   Up To Date
clstr-node-op7r   192.168.15.5  1.1.4   Up To Date
clstr-node-ur03   192.168.15.6  1.1.3   Needs Update
clstr-node-xwdi   192.168.15.7  1.0.4   Needs Update
```

Kubernetes will also set a flag such that at the next time kubectl is run, the system will output a line of information to the end user (once, and once only).

```
$ kubectl get pods
[Notice] There are updates available for your cluster. Please run kubectl check-version for more information.

<remainder of output>
```

<!-- BEGIN MUNGE: GENERATED_ANALYTICS -->
<!-- END MUNGE: GENERATED_ANALYTICS -->
