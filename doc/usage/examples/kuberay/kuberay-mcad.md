### KubeRay-MCAD integration

This integration will help in queuing on [kuberay](https://github.com/ray-project/kuberay) clusters in Multi-cluster-app-dispatcher (MCAD) until aggregated resources are available in the cluster.

#### System Requirement

Minimum hardware requirement: 4 CPUs and 8GB memory

#### Prerequisites
- [Podman](https://podman.io/get-started) is an opensource container engine which allows creating a KinD cluster with specified CPU/Memory resources on a Podman Machine(VM).
- Kubernetes(see [KinD](https://kind.sigs.k8s.io/)) or Openshift cluster(see [OpenShift Local](https://developers.redhat.com/products/openshift-local/overview))
- Kubernetes client tools such as [kubectl](https://kubernetes.io/docs/tasks/tools/) or [OpenShift CLI](https://docs.openshift.com/container-platform/4.13/cli_reference/openshift_cli/getting-started-cli.html)
- [Helm](https://helm.sh/docs/intro/install/)
- Install MCAD and KubeRay operators:
    - KinD cluster:

        Install the stable release of MCAD operator from local charts
        ```bash
        git clone https://github.com/project-codeflare/multi-cluster-app-dispatcher
        cd multi-cluster-app-dispatcher
        helm install mcad --set image.repository=quay.io/project-codeflare/mcad-controller --set image.tag=stable deployment/mcad-controller --set resources.requests.cpu=1000m --set resources.requests.memory=1048Mi --set resources.limits.cpu=1000m --set resources.limits.memory=1048Mi
        ```

        Make sure MCAD has ClusterRole to create ray resources, please patch using [xqueuejob-controller.yaml](doc/usage/examples/kuberay/config/xqueuejob-controller.yaml). For example:
        ```
        kubectl apply -f doc/usage/examples/kuberay/config/xqueuejob-controller.yaml
        ```

        See [deployment.md](../../../../doc/deploy/deployment.md) for more options.

        Install KubeRay operator using the [instructions](https://github.com/ray-project/kuberay#quick-start). For example, install KuberRay v0.6.0 from remote helm repo:
        ```
        helm repo add kuberay https://ray-project.github.io/kuberay-helm/
        helm install kuberay-operator kuberay/kuberay-operator --version 0.6.0
        ```

    - OpenShift cluster:

        On OpenShift, MCAD and KubeRay are already part of the Open Data Hub Distributed Workload Stack. The stack provides a simple, user-friendly abstraction for scaling, queuing and resource management of distributed AI/ML and Python workloads. Please follow the Quick Start in the [Distributed Workloads](https://github.com/opendatahub-io/distributed-workloads) for installation.


#### Steps

> Note that the RayCluster created here is to accommodate a resource-constrained local Kubernetes environment. It is not recommended for real workloads or production.
- Submit the RayCluster custom resource to MCAD as AppWrapper using the [aw-raycluster.yaml](doc/usage/examples/kuberay/config/aw-raycluster.yaml) example:
    ```bash
    kubectl create -f doc/usage/examples/kuberay/config/aw-raycluster.yaml
    ```
- Check the status of the AppWrapper custom resource using command
    ```bash
    kubectl describe appwrapper raycluster-complete -n default
    ```
- Check the RayCluster status is ready using command
    ```bash
    kubectl get raycluster -n default
    ```
    Expect:
    ``````
    NAME                  DESIRED WORKERS   AVAILABLE WORKERS   STATUS   AGE
    raycluster-complete   1                 1                   ready    6m45s
    ```
