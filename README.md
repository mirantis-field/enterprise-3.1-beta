# Docker Enterprise 3.1 Beta Release Notes

**Version 3**

2020-04-06

Learn about new features, bug fixes, breaking changes, and known issues for the beta release of Docker Enterprise 3.1 and the corresponding versions of DTR 2.8.0 and UCP 3.3.0.

## Docker Trusted Registry (DTR) version 2.8.0

2020-04-06

### New features

* In previous versions of DTR, vulnerability reports and promotion policies used the CVSS v2.0 scoring system. Now, DTR can be configured to use either CVSS v2.0 or CVS v3.0. This is a global setting that can be changed at any time. 

### Bug fixes

* Pull mirroring policies now do a full pull mirror for a repository when the tag limit is increased, a pruning policy is deleted, or when a policy pulls a tag that has just been deleted
* Cache namespaces and repositories are now queried for scanned image requests to optimize redundant queries
* We added a repository event that will distinguish policy promotions from manual promotions that are done on a single image using the Promote button in the DTR web interface
* We added an alert in the bottom right side of the DTR web interface when a user scans a tag
* All cron jobs are now included in backups
* We added pagination for promotion policies in the DTR web interface
* We now provide the option to reduce backup size by not backing up the events table for online backups. Offline backups do not have this option. This adds a new flag to DTR CLI for the backup command: `--ignore-events-table`
* We fixed an issue that prevented license information from updating after the license is changed in the DTR web interface
* We improved the DTR web interface for organizations including the organization list, the organization viewer, the organization repo, and the new organization screens
* We added a type query parameter to the `api/v1/repositories/{namespace}/{repository}/tags` API endpoint to enable filtering on app, image, and plugin
* We added event parameter validation to include parameters for event or object type
* We fixed an issue where the constituent image platforms was not populated for the `/api/v1/repositories/{namespace}/{reponame}/tags`  and `/api/v1/repositories/{namespace}/{reponame}/tags/{reference}` API endpoints
* We fixed an issue with invoking `/api/v0/workers/{id}/capacity` API with an invalid `{id}`, which should cause a `404` error but returns `200` (OK) instead. 

## Universal Control Plane (UCP) version 3.3.0

2020-03-16

### Minimum system requirements for UCP

When running a cluster with a single Linux manager node, its system requirements are as follows:
* 4 vCPU
* 8GB RAM

Minimum system requirements for Windows Server 2019 worker node:
* 4 vCPU
* 8GB RAM

### New features

* In previous versions,  UCP did not include support for Kubernetes ingress, and that prevented the use of services running inside of the cluster from servicing requests that originated externally.  Now, a service running inside the cluster can be exposed to service requests that originate externally using Istio Ingress, specifically:
    * Route incoming requests to a specific service using:
        * Hostname
        * URL
        * Request Header
        * Additional istio ingress mechanisms
    * Use the following Istio ingress mechanisms:
        * Gateway
        * Virtual Service
        * Destination Rule 
        * Mixer (handler, instance, and rule)
    * For this release, only Istio Ingress is available
* Windows Server Node VXLAN overlay network data plane in Kubernetes
    * Windows Server and Linux are now normalized to both use VXLAN
    * Overlay networking applied to ensure seamless communication between cluster nodes
    * No longer requires BGP control plane (when using VXLAN) 
* In this release of Docker Enterprise we upgraded the Kubernetes version used by UCP from 1.14.8 (in UCP 3.2) to 1.17.2. For information about Kubernetes changes see [Kubernetes v1.17 Release Notes](https://kubernetes.io/docs/setup/release/notes/).

### Known issues

* Online install fails to pull required image on Windows Server
    * When installing UCP on a cluster with Windows nodes, you must manually pull the following images on all Windows Server 2019 nodes in your cluster before installing:
    `docker pull docker/ucp-kube-binaries-win:3.3.0-beta1`
    `docker pull docker/ucp-containerd-shim-process-win:3.3.0-beta1`
    * If you fail to pre-pull the images the Windows Server nodes will fail to come online
* In this release of Docker Enterprise we upgraded the Kubernetes version used by UCP from 1.14.8 (in UCP 3.2) to 1.17.2.
* The UCP web interface has inconsistencies and unexpected behavior when interacting with the system. To work around this, use the command line tools when necessary.
* You cannot configure VXLAN MTU and port on Windows Server. There is currently no workaround.
* Swarm overlay networking fails between Linux hosts and hosts not running a pod. The workaround is to schedule at least one Kubernetes pod for each Windows Server node
* About 4% of Windows Server nodes may not join the cluster. The workaround is to redeploy the failed nodes.
* You may see a 'Failed to compute desired number of replicas based on listed metrics' in the Istio logs. You may ignore this error.
* When reducing the number of gateways using the UCP web interface, the change will not take effect. The workaround is to toggle Istio off and back on again to lower the number of gateway replicas. Increasing replicas behaves appropriately, no workaround is needed for increases. CRDs from pre-existing Istio installation will be overwritten. To avoid this error, don’t install in clusters where Istio is already installed.
* Resource types (instance, rule, and handler) can’t be created through the UI. The workaround is to perform the operation via `kubectl`. 
* Windows Server nodes don’t become healthy when using the AWS or Azure cloud provider flags in the UCP install command (`docker/ucp install --cloud-provider aws`) and (`docker/ucp install --cloud-provider azure`).  The work around is not to use those commands and to add the `--skip-cloud-provider-check` flag (`docker/ucp install --skip-cloud-provider-check`).
* On Azure, the UCP installer may fail on step 36 of 36. Wait a minute and check the UCP UI to verify all nodes are healthy; the installer will continue when the nodes are ready.
