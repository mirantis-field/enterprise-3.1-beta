# Docker Enterprise exercises

Welcome to the Docker Enterprise 19.3 Beta release! We created these exercises to validate and demo the key features of the release. We also share them with you, our beta customers, to help you understand how to use the new features. These exercises are not intended to be used in your production environment.

The exercises focus on four key scenarios.

1. Kubernetes 1.17 support
2. Kubernetes on Windows
3. Istio Ingress for Kubernetes
4. NVIDIA GPU support

The exercises in this document are not set up to execute sequentially. You may directly jump to the ones that are relevant to your needs.

## Prerequisites

To complete these exercises, you will need a Docker Hub account as well as an Amazon AWS account.
You will use a local system such as a laptop running MacOS or Windows 10 to initiate these exercises.
You will provision AWS instances and deploy Docker Enterprise on those instances as you go along.
Note that while the instructions here use AWS instances, you can also do them on any of the platforms supported by Docker Enterprise.

You will not be able to upgrade from the beta release of Docker Enterprise to the General Availability release.

## Installing a Linux-only UCP cluster

All the exercises in this document require a setup of UCP that has one or more Linux instances. This section describes
how to install such a cluster of one or more Linux instances. You will see references to this section within the setup
instructions for each of the exercises below.

1. Create the first Linux instance using the steps at https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine.
   Choose the Ubuntu 16.04 or 18.04 AMI, unless an exercise specifies a different one.

2. Log into your Linux instance and install Docker Engine - Enterprise, as described in https://docs.docker.com/ee/docker-ee/ubuntu/.

3. Install UCP 3.3.0 beta version on this first Linux instance:

   a. Download the UCP offline bundle using the following command:

   ```bash
   $ curl -o ucp_images.tar.gz https://packages.docker.com/caas/ucp_images_3.3.0-beta1.tar.gz
   ```

   b. Load the UCP image using the following command:

   ```bash
   $ docker load < ucp_images.tar.gz
   ```

   c. Run the following command to install UCP. Substitute your password for `<password>` and the public IP address of your VM for the `<public IP>` placeholder.

   ```bash
   $ docker container run \
     --rm \
     --interactive \
     --tty \
     --name ucp \
     --volume /var/run/docker.sock:/var/run/docker.sock \
     docker/ucp:3.3.0-beta1 \
     install \
     --admin-password <password> \
     --debug \
     --force-minimums \
     --san <public IP>
   ```

   d. You now have a single-node UCP cluster with that Linux instance as its Manager node.

4. If you need to add additional Linux instances to the cluster do the following:

   a. Use your browser on your local system to log into the UCP installation above.

   b. Navigate to the nodes list and click on “Add Node” at the top right of the page.

   c. In the Add Node page select "Linux" as the node type. Choose 'Worker' for the node role.

   d. Optionally, you may also select and set custom listen and advertise addresses.

   e. A command line will be generated that includes a `join-token`. It should look something like:

   ```bash
   docker swarm join ... --token <join-token> ...
   ```

   Copy over this command line from the UI for use later.

5. For each additional Linux instance that you need to add to the cluster for your exercise, do the following:

   a. Create the instance using the steps at https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine,
   once again using the Ubuntu 16.04 or 18.04 AMIs, unless otherwise specified.

   b. Log into your Linux instance and install Docker Engine - Enterprise, as described in https://docs.docker.com/ee/docker-ee/ubuntu/.

   c. Download the UCP offline bundle using the following command:

   ```bash
   $ curl -o ucp_images.tar.gz https://packages.docker.com/caas/ucp_images_3.3.0-beta1.tar.gz
   ```

   d. Load the UCP image using the following command:

   ```bash
   $ docker load < ucp_images.tar.gz
   ```

   d. Add your Linux instance to the UCP cluster by running the swarm-join commandline generated above.

## Installing CLIs on your local system

All the exercises in this document also require you to set up both the 'kubectl' and 'docker' CLIs on your local system,
in order to work with your UCP cluster. You will see references to this section within the setup instructions for each of
the exercises below. Perform these steps after you are done installing the UCP cluster for your exercise:

1. Download the Docker CLI client using the instructions [here](https://docs.docker.com/ee/ucp/user-access/cli/#get-the-docker-cli-client).
2. Download the CLI client bundle from your UCP instance using the procedure [here](https://docs.docker.com/ee/ucp/user-access/cli/).
3. Download and install `kubectl` using [this procedure](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

# Exercise 1: Kubernetes 1.17 support

In this release of Docker Enterprise we upgraded the Kubernetes version used by UCP from 1.14.8 (in UCP 3.2) to 1.17.2.
For information about Kubernetes 1.17, including a comprehensive list of new features,
see [Kubernetes 1.17 Release Notes](https://kubernetes.io/docs/setup/release/notes).

## Changes to the Kubernetes API

The Kubernetes API underwent significant change with v1.16.
For detailed information, refer to the release notes at https://kubernetes.io/blog/2019/09/18/kubernetes-1-16-release-announcement/.
Also, to learn about deprecated APIs that were removed in v1.16, refer to https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/.

## Setting up for this exercise

First follow the instructions in the section on [installing a Linux only UCP cluster](#installing-a-linux-only-ucp-cluster)
to install a **three-node Linux-only UCP cluster** using the Ubuntu 18.04 AMI.

Next follow the instructions in the section on [installing CLIs on your local system](#installing-clis-on-your-local-system)
to install and set up the 'docker' and 'kubectl' CLIs on your local system to access your UCP cluster.

## Checking Kubernetes versions

```bash
$ kubectl version

Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.3", GitCommit:"06ad960bfd03b39c8310aaf92d1e7c12ce618213", GitTreeState:"clean", BuildDate:"2020-02-11T18:14:22Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17+", GitVersion:"v1.17.2-docker-d-2", GitCommit:"ae40ab25c94b30bfd3034be3056cd84318e97a2f", GitTreeState:"clean", BuildDate:"2020-02-05T20:42:57Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"linux/amd64"}

```

```bash
$ docker version

Client: Docker Engine - Enterprise
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        2ee0c57
 Built:             Wed Nov 13 07:33:38 2019
 OS/Arch:           darwin/amd64
 Experimental:      true

Server: Docker Enterprise 3.1
 Engine:
  Version:          19.03.8-beta1
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.16
  Git commit:       61dfab2
  Built:            Tue Mar 10 21:43:48 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
 Universal Control Plane:
  Version:          3.3.0-beta1
  ApiVersion:       1.40
  Arch:             amd64
  BuildTime:        Fri Mar 13 17:34:14 UTC 2020
  GitCommit:        f54d47f
  GoVersion:        go1.13.8
  MinApiVersion:    1.20
  Os:               linux
 Kubernetes:
  Version:          1.17+
  buildDate:        2020-02-05T20:42:57Z
  compiler:         gc
  gitCommit:        ae40ab25c94b30bfd3034be3056cd84318e97a2f
  gitTreeState:     clean
  gitVersion:       v1.17.2-docker-d-2
  goVersion:        go1.13.6
  major:            1
  minor:            17+
  platform:         linux/amd64
 Calico:
  Version:          v3.12.0
  cni:              v3.12.0
  kube-controllers: v3.12.0
  node:             v3.12.0
```

## Kubelet no longer runs as a container

As a result of the deprecation and removal of the “containerized” flag in Kubernetes, kubelet as well as kube-proxy no longer runs as containers.
For detailed information, refer to https://github.com/kubernetes/kubernetes/issues/74148.

However, UCP continues to support capturing log output in container logs for ease of viewing and aggregating using logging drivers.

```bash
$ docker logs <node-name>/ucp-kubelet

time="2020-03-15T05:58:32Z" level=debug msg="importing image 'docker.io/docker/ucp-hyperkube:3.3.0-beta1' into containerd"
time="2020-03-15T05:58:48Z" level=debug msg="unpacking docker.io/docker/ucp-hyperkube:3.3.0-beta1 (sha256:da3f0fc2cf2fa0ecdc71480429c5352e9b83fe13866d2142726d8c01dca829af)"
time="2020-03-15T05:58:55Z" level=info msg="container not found, skipping cleanup..."
systemd-resolved is running, so using --resolv-conf=/run/systemd/resolve/resolv.conf for kubelet.
Starting Kubelet...
Flag --resolv-conf has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Flag --resolv-conf has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
I0315 05:58:55.927675    7587 server.go:416] Version: v1.17.2-docker-d-2
I0315 05:58:55.927972    7587 plugins.go:100] No cloud provider specified.
I0315 05:58:56.013296    7587 server.go:641] --cgroups-per-qos enabled, but --cgroup-root was not specified.  defaulting to /
I0315 05:58:56.014533    7587 container_manager_linux.go:265] container manager verified user specified cgroup-root exists: []
...
```

```bash
$ docker logs <node-name>/ucp-kube-proxy

W0315 05:57:12.850193       1 server.go:213] WARNING: all flags other than --config, --write-config-to, and --cleanup are deprecated. Please begin using a config file ASAP.
W0315 05:57:17.205905       1 server_others.go:323] Unknown proxy mode "", assuming iptables proxy
E0315 05:57:17.431107       1 node.go:124] Failed to retrieve node info: nodes "ryan-3199a0-ubuntu-0" not found
E0315 05:57:19.213620       1 node.go:124] Failed to retrieve node info: nodes "ryan-3199a0-ubuntu-0" not found
...
```

# Exercise 2: Kubernetes on Windows Server

Starting with version 3.3.0, UCP now supports the Kubernetes orchestrator on Windows Server nodes.

This exercise walks you through the process of setting up a Docker Enterprise cluster for Windows workloads.
Windows Server nodes can only be set up as worker nodes (not managers) in a Docker Enterprise cluster.

UCP uses a CNI plugin that supports 2 modes of encapsulation. Prior versions of UCP used IPIP (or IP-in-IP)
encapsulation which is not compatible with Kubernetes on Windows Server. UCP 3.3.0+ defaults to VXLAN encapsulation which is
compatible with both Linux and Windows Server Kubernetes. You can not change modes during upgrade at this time.
You must, therefore, perform a fresh install if you want Windows Server Kubernetes support.

For this exercise we will be using a three-node cluster consisting of one Linux manager and two Windows Server 2019 worker AWS instances.

## Setting up for this exercise

First follow the instructions in the section on [installing a Linux only UCP cluster](#installing-a-linux-only-ucp-cluster)
to install a **one-node Linux-only UCP cluster** using the Ubuntu 18.04 AMI.

Next follow the instructions in the section on [installing CLIs on your local system](#installing-clis-on-your-local-system)
to install and set up the 'docker' and 'kubectl' CLIs on your local system to access your UCP cluster.

## Adding Windows Server nodes

Having installed a one-node Linux-only UCP cluster, we are now ready to add additional Windows workers to this cluster using these steps:

1. Use your browser on your local system to log into the UCP installation above.
2. Navigate to the nodes list and click on “Add Node” at the top right of the page.
3. In the Add Node page select "Windows" as the node type.
   You will notice that Windows Server nodes are only allowed to have worker roles.
4. Optionally, you may also select and set custom listen and advertise addresses.
5. A command line will be generated that includes a `join-token`. It should look something like:

   `docker swarm join ... --token <join-token> ...`

   Copy over this command line from the UI for use later.

6. Create a Windows Server instance using the steps in https://aws.amazon.com/getting-started/tutorials/launch-windows-vm/.
   Choose one of the Windows Server 2019 AMIs that does not include containers. You will install Docker Engine in the next step.
7. Log into your Windows VM and run the following commands in a powershell window with administrator rights. Do not use ISE.
   The `Enable-WindowsOptionalFeature` command may require you to restart your computer. If you are asked to do that,
   you should continue with the next step after the restart.

   ```powershell
   # Enable the Windows Feature for containers
   Enable-WindowsOptionalFeature `
     -All `
     -FeatureName containers `
     -Online;

   # Restart server if needed
   Restart-Computer;

   # Allow downloaded script files to run in the current session
   Set-ExecutionPolicy `
     -ExecutionPolicy RemoteSigned `
     -Force `
     -Scope Process;

   # Download the installation script
   Invoke-WebRequest `
     -OutFile 'install.ps1' `
     -Uri 'https://docker-ee-windows.s3-us-west-2.amazonaws.com/get-mirantis.ps1' `
     -UseBasicParsing;

   # Execute the installation script
   ./install.ps1 -dockerVersion '19.03.5';

   # Clean up installation script
   Remove-Item -Path 'install.ps1';
   ```

8. Refresh by logging out of and then back into the VM.
9. Download and load the required Windows container images in to the Docker engine on your Windows Server node:

   ```powershell
   # Download bundled UCP images
   Invoke-WebRequest `
     -OutFile 'ucp_images.tar.gz' `
     -Uri 'https://packages.docker.com/caas/ucp_images_win_2019_3.3.0-beta1.tar.gz' `
     -UseBasicParsing;

   # Load the archive of UCP images into Docker
   docker load --input 'ucp_images.tar.gz';

   # Verify images were successfully loaded
   docker images;

   # Clean up bundled UCP images
   Remove-Item -Path 'ucp_images.tar.gz';
   ```

10. Add your Windows Server VM to the UCP cluster by running the swarm-join commandline generated above.
11. Repeat steps 6 - 10 in order to create a second Windows Server VM and have it join the cluster.

## Validating the cluster setup

1. You can use either the UCP web interface or the command line to view your clusters.

   - To use the web interface, log into UCP, and navigate to the node list view. All nodes should be green.
   - Check the node status using this command on your local system:

     `kubectl get nodes`.

   Your nodes should all have a status value of "Ready", as in the following example.

   ```shell
   NAME                   STATUS   ROLES    AGE     VERSION
   ryan-135716-win-0      Ready    <none>   2m16s   v1.17.2
   ryan-7d985f-ubuntu-0   Ready    master   4m55s   v1.17.2-docker-d-2
   ryan-135716-win-1      Ready    <none>   1m12s   v1.17.2
   ```

2. Now that you have confirmed that the nodes are ready, the next step is to change the orchestrator to kubernetes for the pods.
   You can change the orchestrator using the UCP web interface, either by changing the default orchestrator in the Administrator settings,
   or by using the web interface to toggle the orchestrator after joining a node. The equivalent CLI command is

   `docker node update <node name> --label-add com.docker.ucp.orchestrator.kubernetes=true`

3. Optionally, you can deploy a workload on the cluster to make sure everything is working as expected.

## Troubleshooting

If you can't join your Windows Server node to the cluster, confirm that the correct processes are running on the node.

`PS C:\> Get-Process tigera-calico`

You should see something like this.

```powershell
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    276      17    33284      40948      39.89   8132   0 tigera-calico
```

The next troubleshooting step is to check the kubelet process.

`PS C:\> Get-Process kubelet`

You should see something like this.

```powershell
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    524      23    47332      73380     828.50   6520   0 kubelet
```

After that check the kube-proxy process.

`PS C:\> Get-Process kube-proxy`

You should see something like this.

```powershell
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    322      19    25464      33488      21.00   7852   0 kube-proxy
```

If any of the process checks show that something isn't working, the next step is to check the logs.
For kubelet and kubeproxy, the logs are placed under `C:\k\logs>`. Tigera/Calico logs are placed under `C:\TigeraCalico\logs`.

> Note: Log retrieval on Windows Server nodes will be aligned with that on Linux nodes in the GA version

## An example workload on Windows Server

This section captures an example workload that serves to illustrate Kubernetes on Windows Server capabilities.
The following procedure deploys a complete web application on IIS servers as Kubernetes Services. The example
workload includes an MSSQL database and a loadbalancer.

Specifically, the procedure covers:

- Namespace creation
- Pod and Deployment scheduling
- Kubernetes service provisioning
- Addition of application workloads
- Connectivity of Pods, Nodes and Services

1. Create a Namespace.

   ```
   $ kubectl create -f demo-namespace.yaml
   ```

   ```yaml
   # demo-namespace.yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: demo
   ```

2. Create a web service as a Kubernetes service.

   ```bash
   $ kubectl create -f win-webserver.yaml
   service/win-webserver created
   deployment.apps/win-webserver created

   $ kubectl get service --namespace demo
   NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
   win-webserver   NodePort   10.96.29.12   <none>        80:35048/TCP   12m
   ```

   ```yaml
   # win-webserver.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: win-webserver
     namespace: demo
     labels:
       app: win-webserver
   spec:
     ports:
       # the port that this service should serve on
       - port: 80
         targetPort: 80
     selector:
       app: win-webserver
     type: NodePort
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
       app: win-webserver
       namespace: demo
     name: win-webserver
     namespace: demo
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: win-webserver
     template:
       metadata:
         labels:
           app: win-webserver
         name: win-webserver
       spec:
         affinity:
           podAntiAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
               - labelSelector:
                   matchExpressions:
                     - key: app
                       operator: In
                       values:
                         - win-webserver
                 topologyKey: "kubernetes.io/hostname"
         containers:
           - name: windowswebserver
             image: mcr.microsoft.com/windows/servercore:ltsc2019
             command:
               - powershell.exe
               - -command
               - "<#code used from https://gist.github.com/wagnerandrade/5424431#> ; $$listener = New-Object System.Net.HttpListener ; $$listener.Prefixes.Add('http://*:80/') ; $$listener.Start() ; $$callerCounts = @{} ; Write-Host('Listening at http://*:80/') ; while ($$listener.IsListening) { ;$$context = $$listener.GetContext() ;$$requestUrl = $$context.Request.Url ;$$clientIP = $$context.Request.RemoteEndPoint.Address ;$$response = $$context.Response ;Write-Host '' ;Write-Host('> {0}' -f $$requestUrl) ;  ;$$count = 1 ;$$k=$$callerCounts.Get_Item($$clientIP) ;if ($$k -ne $$null) { $$count += $$k } ;$$callerCounts.Set_Item($$clientIP, $$count) ;$$ip=(Get-NetAdapter | Get-NetIpAddress); $$header='<html><body><H1>Windows Container Web Server</H1>' ;$$callerCountsString='' ;$$callerCounts.Keys | % { $$callerCountsString+='<p>IP {0} callerCount {1} ' -f $$ip[1].IPAddress,$$callerCounts.Item($$_) } ;$$footer='</body></html>' ;$$content='{0}{1}{2}' -f $$header,$$callerCountsString,$$footer ;Write-Output $$content ;$$buffer = [System.Text.Encoding]::UTF8.GetBytes($$content) ;$$response.ContentLength64 = $$buffer.Length ;$$response.OutputStream.Write($$buffer, 0, $$buffer.Length) ;$$response.Close() ;$$responseStatus = $$response.StatusCode ;Write-Host('< {0}' -f $$responseStatus)  } ; "
         nodeSelector:
           beta.kubernetes.io/os: windows
   ```

3. Check the pods deployed on different Windows Server worker nodes with Inter-pod affinity and anti-affinity.

   ```bash
   $ kubectl get pod --namespace demo

   NAME                            READY   STATUS    RESTARTS   AGE
   win-webserver-8c5678c68-qggzh   1/1     Running   0          6m21s
   win-webserver-8c5678c68-v8p84   1/1     Running   0          6m21s

   # Check the detailed status of pods deployed
   $ kubectl describe pod win-webserver-8c5678c68-qggzh --namespace demo
   ```

4. Access the web service by node-to-pod communication across the network.

   From a kubectl client:

   ```bash
    $ kubectl get pods --namespace demo -o wide

    NAME                            READY   STATUS    RESTARTS   AGE   IP              NODE              NOMINATED NODE   READINESS GATES
    win-webserver-8c5678c68-qggzh   1/1     Running   0          16m   192.168.77.68   ryan-135716-win-1 <none>           <none>
    win-webserver-8c5678c68-v8p84   1/1     Running   0          16m   192.168.4.206   ryan-135716-win-0 <none>           <none>

    $ ssh -o ServerAliveInterval=15 root@<master-node>

    $ curl 10.96.29.12
    <html><body><H1>Windows Container Web Server</H1><p>IP 192.168.77.68 callerCount 1 </body></html>
    $
   ```

   Run the curl command a second time. You can see the second request load-balanced to a different pod.

   ```bash
   $ curl 10.96.29.12
   <html><body><H1>Windows Container Web Server</H1><p>IP 192.168.4.206 callerCount 1 </body></html>
   ```

5. Access the web service by pod-to-pod communication across the network.

   ```bash
   $ kubectl get service --namespace demo

   NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
   win-webserver   NodePort   10.96.29.12   <none>        80:35048/TCP   12m

   $ kubectl get pods --namespace demo -o wide

   NAME                            READY   STATUS    RESTARTS   AGE   IP              NODE              NOMINATED NODE   READINESS GATES
   win-webserver-8c5678c68-qggzh   1/1     Running   0          16m   192.168.77.68   ryan-135716-win-1 <none>           <none>
   win-webserver-8c5678c68-v8p84   1/1     Running   0          16m   192.168.4.206   ryan-135716-win-0 <none>           <none>

   $ kubectl exec -it win-webserver-8c5678c68-qggzh --namespace demo cmd

   Microsoft Windows [Version 10.0.17763.1098]
   (c) 2018 Microsoft Corporation. All rights reserved.

   C:\>curl 10.96.29.12
   <html><body><H1>Windows Container Web Server</H1><p>IP 192.168.77.68 callerCount 1 <p>IP 192.168.77.68 callerCount 1 </body></html>
   ```

# Exercise 3: Istio Ingress for Kubernetes

## Overview

Istio Ingress for Kubernetes provides abstractions over Kubernetes, facilitating the development of distributed microservice networks. This feature does not include Istio’s service mesh, but assists with the monitoring, routing, and managing of requests entering the microservice network from outside.

With this feature enabled, you can now expose apps and containers running inside your Kubernetes cluster to the outside world. You can route incoming requests using hostname, URL, header, and other characteristics to dictate which service receives the request.

> **Note** Disabled by default, the Instio Ingress feature can be toggled on and off repeatedly with UCP.

The main Istio Ingress concepts are Gateway, Virtual Service, Destination Rule, and Mixer.

| Component | Description|
| --- | --- |
| Gateway | Describes a load balancer operating at the edge of the mesh receiving incoming or outgoing HTTP/TCP connections. The specification describes a set of ports that should be exposed, the type of protocol to use, SNI configuration for the load balancer, etc. |
| Virtual Service  | Defines a set of traffic routing rules to apply when a host is addressed. Each routing rule defines matching criteria for traffic of a specific protocol. If the traffic is matched, then it is sent to a named destination service (or subset/version of it) defined in the registry. |
| Destination Rule | Defines policies that apply to traffic intended for a service after routing has occurred. These rules specify configuration for load balancing, connection pool size from the sidecar, and outlier detection settings to detect and evict unhealthy hosts from the load balancing pool. |
| Mixer | Responsible for providing policy controls and telemetry collection. Controlling the policy and telemetry features involves configuring three types of resources: handler, instance, and rule. |

## Setting up for this exercise

First follow the instructions in the section on [installing a Linux only UCP cluster](#installing-a-linux-only-ucp-cluster)
to install a **three-node Linux-only UCP cluster** using the Ubuntu 18.04 AMI.

Next follow the instructions in the section on [installing CLIs on your local system](#installing-clis-on-your-local-system)
to install and set up the `docker` and `kubectl` CLIs on your local system to access your UCP cluster.

## Usage

The Istio Ingress feature can be used by administrators and developers.

### Administrator usage

Traffic in Istio is categorized as data plane traffic and control plane traffic. Data plane traffic refers to the messages that the business logic of the workloads send and receive. Control plane traffic refers to configuration and control messages sent between Istio components to program the behavior of the mesh. Traffic management in Istio refers exclusively to data plane traffic.

> **Note** In order for development teams to use Istio Ingress, you must first configure the appropriate grants using the instructions provided at [Creating Role Grants](https://docs.docker.com/ee/ucp/admin/configure/configure-rbac-kube/#creating-role-grants).

1. Open the UCP web UI.
2. Select **Admin Settings**.
3. Select **Ingress**.
4. Under the **Kubernetes** tab:
   - Click the slider to enable Ingress for Kubernetes.
   - Configure the proxy to specify how external traffic is handled by the cluster:
   - Specify the node ports for the Istio Ingress Gateway Service. External traffic can enter the cluster via these ports. Ensure that these ports are open.
   - (Optional) Select **External IP** to create a Layer 7 load balancer in front of multiple nodes. You can then add a list of external IP addresses to the Ingress Gateway service.
   - Configure the replicas to specify how to scale load balancing.
   - Configure placement rules and load balancer configurations.
   - Click **Save**.
5. Create an Istio Gateway to expose your apps:

   - Select **Kubernetes**.
   - Select **Ingress**.
   - Under the **Gateways** tab, click **Create**.
   - Select the nodes on which this Gateway configuration will be applied.
   - Add the server details. These details describe the properties of the proxy on a given load balancer port.
   - Click **Generate YML** to create the configuration file.

   > **Note** You also have the option to create or upload your own YAML configuration file.

6. Click **Skip to YAML Editor**.

### Developer usage

This section assumes your developers have access to deploy workloads to the cluster.

### Use cases

#### Create an Istio Virtual Service

As an administrator, you need to create an Istio Virtual Service to route all requests matching `<domain>/{status,delay}` to an application. Virtual Services are namespace scoped and can only route to applications within the namespace they are created in.

To create a virtual service:

1. Select **Kubernetes**.
2. Select **Ingress**.
3. Under the **Virtual Services** tab, click **Create**.

   - Enter a name for the virtual service.
   - Add the destination hosts to which traffic is being sent.
   - Add the gateways.
   - Click **Generate YML** to create the configuration file.

   **Note** You also have the option to create or upload your own YAML configuration file.

   - Click **Skip to YAML Editor**.

Sample YAML configuration file:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  generation: 1
  name: httpbin-vs
  selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/httpbin-vs
spec:
  gateways:
  - application-gateway
  hosts:
  - httpbin
  http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
    route:
    - destination:
        host: httpbin
        port:
          number: 8000
```

#### Enable split testing

To enable split testing of an application, the administrator can add an Istio Destination Rule to route a small percentage of traffic to the new version of the application.

1. Add the Istio Destination Rule in the namespace where the application is deployed. This creates the service subsets needed for the Virtual Service’s matcher, based on Kubernetes pod labels.

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin-destination-rule
spec:
  host: httpbin
  subsets:
  - name: 'v1.0.0'
    labels:
      version: 'v1.0.0'
  - name: 'v1.1.0-featuretest'
    labels:
      version: 'v1.1.0-featuretest'
```

2. Edit the existing Virtual Service and add the needed routing policy, matching by destination rule.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin-vs
spec:
  gateways:
  - application-gateway
  hosts:
  - '*'
  http:
  - route:
    - destination:
        host: httpbin
        subset: 'v1.0.0'
      weight: 95
    - destination:
        host: httpbin
        subset: 'v1.1.0-featuretest'
      weight: 5
```

#### Configure sticky sessions

As a developer, you may need customers participating in split testing to consistently see a particular feature. For this, the administrator must add a sticky session to the initial application configuration, which will force Istio Ingress to route all follow-up requests from the same caller to the same pod.

The administrator creates a new Destination Rule, declaring a hashing-based load balancer for the application using the user cookie as the hash key:

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin-destination-rule
spec:
  host: httpbin.default.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      consistentHash:
        httpCookie:
          name: User
          ttl: 0s
```

#### Canary deployments

Canary deployments are a pattern for rolling out releases to a subset of users or servers. Canary deployments are useful if a developer wants to gradually deploy a new version of an application without any downtime.

To create a canary deployment, the administrator creates a Destination Rules for both versions of the application.

1. Add the Istio Destination Rule in the namespace where the application is deployed. This creates the service subsets needed for the Virtual Service’s matcher, based on Kubernetes pod labels.
2. Edit the existing Virtual Service and add the needed routing policy, matching by destination rule.
3. In the Virtual Service configuration, specify the weight of each version.
4. Gradually increase the weight of the new version, until it reaches 100%.
5. Additionally, use the [Kubernetes Autoscaler](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#cluster-autoscaling) to make sure that there are always available instances.

#### Blacklisting

To prevent DoS attacks, the administrator can blacklist the offending IP addresses to prevent them from degrading the application’s uptime. The blacklist can be created using Istio’s Mixer policies.

1. Create a handler, instance, and rule, which combined will filter out any requests from IP addresses specified in the Handler’s override spec:

   ```
   kubectl apply -f
   ```

2. Use the x-forwarded-for header that Istio automatically attaches.
3. Update the UCP config file, updating `cluster_config.service_mesh.ingress_preserve_client_ip` to `true`.

Sample configuration file:

```
apiVersion: "config.istio.io/v1alpha2"
kind: handler
metadata:
  name: blacklisthandler
  namespace: istio-system
spec:
  compiledAdapter: listchecker
  params:
    overrides:
    - 37.72.166.13
    - <IP/CIDR TO BE BLACKLISTED>
    blacklist: true
    entryType: IP_ADDRESSES
    refresh_interval: 1s
    ttl: 1s
    caching_interval: 1s
---
apiVersion: "config.istio.io/v1alpha2"
kind: instance
metadata:
  name: blacklistinstance
  namespace: istio-system
spec:
  compiledTemplate: listentry
  params:
    value: ip(request.headers["x-forwarded-for"]) || ip("0.0.0.0")
---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: blaclistcidrblock
  namespace: istio-system
spec:
  match: (source.labels["istio"] | "") == "ingressgateway"
  actions:
  - handler: blacklisthandler
    instances:
    - blacklistinstance
```

#### Related information

- [What is Istio?](https://istio.io/docs/concepts/what-is-istio/)
- [Kubernetes Cluster Ingress (Experimental)](https://docs.docker.com/ee/ucp/kubernetes/cluster-ingress/)

# Exercise 4: GPU support for Kubernetes workloads

Docker Enterprise now includes GPU support for Kubernetes workloads. This exercise walks you through setting up your system
to use underlying GPU support, and through deploying GPU-targeted workloads.

## Setting up for this exercise

First follow the instructions in the section on [installing a Linux only UCP cluster](#installing-a-linux-only-ucp-cluster)
to install a **two-node Linux-only UCP cluster** using one of the [GPU instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/accelerated-computing-instances.html).

Next follow the instructions in the section on [installing CLIs on your local system](#installing-clis-on-your-local-system)
to install and set up the 'docker' and 'kubectl' CLIs on your local system to access your UCP cluster.

There are two parts required in order to get set up for GPU support:

1. Installing the GPU drivers, which may be done either before or after UCP installation.
2. Deploying the device plugin, which is taken care of for you right out-of-the-box with UCP.

## Installing the GPU drivers

This procedure will install the NVIDIA driver via runfile on your Linux host. The latest NVIDIA driver can be found [here](https://www.nvidia.com/en-us/drivers/unix/). This procedure uses version 440.59, which is the latest available and verified version at the time of this writing.

**Note** This procedure describes how to manually install these drivers, but it is recommended that you use a pre-existing automation system to automate installation and patching of the drivers along with the kernel and other host software.

1. Ensure that your NVIDIA GPU is supported:
   `lspci | grep -i nvidia`

2. Verify that your GPU is present in the [list of supported NVIDIA GPU products](http://download.nvidia.com/XFree86/Linux-x86_64/440.59/README/supportedchips.html).
3. Install [dependencies](http://us.download.nvidia.com/XFree86/Linux-x86_64/440.59/README/minimumrequirements.html).
4. Make sure that your system is up-to-date, and you are running the latest kernel.
5. Install the following packages depending on your OS.

- Ubuntu:

  ```
  sudo apt-get install -y gcc make curl linux-headers-$(uname -r)
  ```

- RHEL:

  ```
  sudo yum install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r) gcc make curl elfutils-libelf-devel
  ```

6. Ensure that `i2c_core` and `ipmi_msghandler` kernel modules are loaded:

   ```
   sudo modprobe -a i2c_core ipmi_msghandler
   ```

   To persist the change across reboots:

   ```
   echo -e "i2c_core\nipmi_msghandler" | sudo tee /etc/modules-load.d/nvidia.conf
   ```

   All of the NVIDIA libraries are present under the specific directory on the host:

   ```
   NVIDIA_OPENGL_PREFIX=/opt/kubernetes/nvidia
   sudo mkdir -p $NVIDIA_OPENGL_PREFIX/lib
   echo "${NVIDIA_OPENGL_PREFIX}/lib" | sudo tee /etc/ld.so.conf.d/nvidia.conf
   sudo ldconfig
   ```

7. Run the installation:

   ```
   NVIDIA_DRIVER_VERSION=440.59
   curl -LSf https://us.download.nvidia.com/XFree86/Linux-x86_64/${NVIDIA_DRIVER_VERSION}/NVIDIA-Linux-x86_64-${NVIDIA_DRIVER_VERSION}.run -o nvidia.run
   ```

**Note** `--opengl-prefix` must be set to `/opt/kubernetes/nvidia`: `sudo sh nvidia.run --opengl-prefix="${NVIDIA_OPENGL_PREFIX}"`

8. Load the NVIDIA Unified Memory kernel module and create device files for the module on startup:

   ```
   sudo tee /etc/systemd/system/nvidia-modprobe.service << END
   [Unit]
   Description=NVIDIA modprobe

   [Service]
   Type=oneshot
   RemainAfterExit=yes
   ExecStart=/usr/bin/nvidia-modprobe -c0 -u

   [Install]
   WantedBy=multi-user.target
   END

   sudo systemctl enable nvidia-modprobe
   sudo systemctl start nvidia-modprobe
   ```

9. Enable the NVIDIA persistence daemon to initialize GPUs and keep them initialized:

   ```
   sudo tee /etc/systemd/system/nvidia-persistenced.service << END
   [Unit]
   Description=NVIDIA Persistence Daemon
   Wants=syslog.target

   [Service]
   Type=forking
   PIDFile=/var/run/nvidia-persistenced/nvidia-persistenced.pid
   Restart=always
   ExecStart=/usr/bin/nvidia-persistenced --verbose
   ExecStopPost=/bin/rm -rf /var/run/nvidia-persistenced

   [Install]
   WantedBy=multi-user.target
   END

   sudo systemctl enable nvidia-persistenced
   sudo systemctl start nvidia-persistenced
   ```

See [Driver Persistence](https://docs.nvidia.com/deploy/driver-persistence/index.html) for more information.

## Deploying the device plugin

UCP includes a GPU device plugin to instrument your GPUs. This section describes deploying `nvidia.com/gpu`.

```
kubectl describe node <node-name>

...
Capacity:
 cpu:                8
 ephemeral-storage:  40593612Ki
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             62872884Ki
 nvidia.com/gpu:     1
 pods:               110
Allocatable:
 cpu:                7750m
 ephemeral-storage:  36399308Ki
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             60775732Ki
 nvidia.com/gpu:     1
 pods:               110
 ...
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource        Requests    Limits
  --------        --------    ------
  cpu             500m (6%)   200m (2%)
  memory          150Mi (0%)  440Mi (0%)
  nvidia.com/gpu  0           0
```

## Scheduling GPU workloads

To consume GPUs from your container, request `nvidia.com/gpu` in the `limits` section. The following example shows how to deploy a simple workload that reports detected NVIDIA CUDA devices.

1. Create the example deployment:

   ```
   kubectl apply -f- <<EOF
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     creationTimestamp: null
     labels:
       run: gpu-test
     name: gpu-test
   spec:
     replicas: 1
     selector:
       matchLabels:
         run: gpu-test
     template:
       metadata:
         labels:
           run: gpu-test
       spec:
         containers:
         - command:
           - sh
           - -c
           - "deviceQuery && sleep infinity"
           image: kshatrix/gpu-example:cuda-10.2
           name: gpu-test
           resources:
             limits:
               nvidia.com/gpu: "1"
   EOF
   ```

2. If you have any available GPUs in your system, the pod will be scheduled on them. After some time, the Pod should be in the `Running` state:

   ```kubectl get po
   NAME                        READY   STATUS    RESTARTS   AGE
   gpu-test-747d746885-hpv74   1/1     Running   0          14m
   ```

3. Check the logs and look for `Result = PASS` to verify successful completion:

   ```
   kubectl logs <name of the pod>

   deviceQuery Starting...

   CUDA Device Query (Runtime API) version (CUDART static linking)

   Detected 1 CUDA Capable device(s)

   Device 0: "Tesla V100-SXM2-16GB"
   ...

   deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.2, CUDA Runtime Version = 10.2, NumDevs = 1
   Result = PASS
   ```

4. Determine the overall GPU capacity of your cluster by inspecting its nodes:

   ```
   echo $(kubectl get nodes -l com.docker.ucp.gpu.nvidia="true" -o jsonpath="0{range .items[*]}+{.status.allocatable['nvidia\.com/gpu']}{end}") | bc
   ```

5. Set the proper replica number to acquire all available GPUs:

   ```
   kubectl scale deployment/gpu-test --replicas N
   ```

6. Verify that all of the replicas are scheduled:
   ```
   kubectl get po
   NAME                        READY   STATUS    RESTARTS   AGE
   gpu-test-747d746885-hpv74   1/1     Running   0          12m
   gpu-test-747d746885-swrrx   1/1     Running   0          11m
   ```

If you attempt to add an additional replica, it should result in a `FailedScheduling` error with `Insufficient nvidia.com/gpu` message:

```
kubectl scale deployment/gpu-test --replicas N+1

kubectl get po
NAME                        READY   STATUS    RESTARTS   AGE
gpu-test-747d746885-hpv74   1/1     Running   0          14m
gpu-test-747d746885-swrrx   1/1     Running   0          13m
gpu-test-747d746885-zgwfh   0/1     Pending   0          3m26s
```

Run `kubectl describe po gpu-test-747d746885-zgwfh` to see the status of the failed deployment:

```
...
Events:
Type     Reason            Age        From               Message
----     ------            ----       ----               -------
Warning  FailedScheduling  <unknown>  default-scheduler  0/2 nodes are available: 2 Insufficient nvidia.com/gpu.
```

Remove the deployment and corresponding pods:

```
kubectl delete deployment gpu-test
```

## Related information

- [Schedule GPUs](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/)
- [Minimum Requirements](http://us.download.nvidia.com/XFree86/Linux-x86_64/440.59/README/minimumrequirements.html)
- [Supported NVIDIA GPU Products](http://download.nvidia.com/XFree86/Linux-x86_64/440.59/README/supportedchips.html)
