$$ \Huge \color{#516baa} Istio: \space Sidecar \space vs \space Ambient $$
<p align="center">
    <strong>A Comparison of Istio's Data Plane Modes: Sidecar and Ambient</strong>
</p>
<img src="./istio-icon.svg" style="vertical-align: middle;">

<!-- the following alternative is prettier but does not work with GutHub-flavored Markdown: -->
<!-- <div style="text-align: center;">
    <a style="margin: 0 10px 0 10px;">
        <img src="./istio-icon.svg" style="vertical-align: middle;">
    </a>
    <a style="margin: 0 10px 0 10px;">
        <strong>A Comparison of Istio's Data Plane Models: Sidecar and Ambient</strong>
    </a>
    <a style="margin: 0 10px 0 10px;">
        <img src="./istio-icon.svg" style="vertical-align: middle;">
    </a>
</div> -->


## Comparison Summary

...bla bla bla...


## Instructions to Run the Toy Project


### First...
- #### ...Without Istio:
    ```bash
    MANIFEST_SUBFOLDER="no-mesh"
    ```
- #### ...in Sidecar Mode:
    ```bash
    MANIFEST_SUBFOLDER="sidecar-mode"
    ```
- #### ...in Ambient Mode:
    ```bash
    MANIFEST_SUBFOLDER="ambient-mode"
    ```

### Then:
1. #### Set Up:
    ```bash
    alias kubectl='microk8s kubectl'

    curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.28.2 TARGET_ARCH=x86_64 sh -
    mv ./istio-1.28.2/bin/istioctl .
    rm -r ./istio-1.28.2

    sudo snap install microk8s --classic --channel=1.32
    sudo usermod -a -G microk8s $USER
    sudo chown -R $USER ~/.kube
    newgrp microk8s
    microk8s status --wait-ready
    microk8s enable community
    microk8s enable dns
    sudo microk8s enable hostpath-storage
    microk8s enable istio
    ```
1. #### Deploy:
    ```bash
    manifest_folder="./manifests"
    manifest_subfolder="${manifest_folder}/${MANIFEST_SUBFOLDER}"
    subfolder_for_microservice_overlays="${manifest_folder}/microservices/overlays"

    ./istioctl install -f "${manifest_folder}/istio-configurations.yaml"

    kubectl apply -f "${manifest_subfolder}"
    kubectl kustomize "${subfolder_for_microservice_overlays}/microservice-a" | kubectl apply -f -
    kubectl kustomize "${subfolder_for_microservice_overlays}/microservice-b" | kubectl apply -f -
    kubectl kustomize "${subfolder_for_microservice_overlays}/microservice-c" | kubectl apply -f -
    ```
1. #### Experiment:
    ```bash
    namespace_name=$(grep -o 'name: .*' "${manifest_subfolder}/namespace.yaml" | cut -d ' ' -f 2)

    kubectl port-forward -n ${namespace_name} service/microservice-a 8081:80
    curl -w '\n' http://localhost:8081/endpoint?message=welcome
    ```
1. #### Tear Down:
    ```
    kubectl delete -f "${manifest_subfolder}/namespace.yaml"
    ./istioctl uninstall --purge
    kubectl delete namespace istio-system
    ```


## Useful Information

### Docs
#### Introductory:
- [What is Istio?](https://istio.io/latest/docs/overview/what-is-istio/)
- [Why choose Istio?](https://istio.io/latest/docs/overview/why-choose-istio/)
- [Sidecar or ambient?](https://istio.io/latest/docs/overview/dataplane-modes/)
#### Both for Sidecar and Ambient Modes:
- [Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
- [Authorization Policy](https://istio.io/latest/docs/reference/config/security/authorization-policy/)
- [Observability](https://istio.io/latest/docs/concepts/observability/)
- [Security](https://istio.io/latest/docs/concepts/security/)
- [Install the Istio CNI node agent](https://istio.io/latest/docs/setup/additional-setup/cni/)
#### For Sidecar Mode:
- [Architecture](https://istio.io/latest/docs/ops/deployment/architecture/)
#### For Ambient Mode:
- [Overview](https://istio.io/latest/docs/ambient/overview/)
- [Architecture](https://istio.io/latest/docs/ambient/architecture/)


## ToDos
- K8s Gateway with no mesh
- install Istio
    - for disabling all egress traffic, set the `outboundTrafficPolicy.mode` to `REGISTRY_ONLY` when installing Istio with the required settings via `istioctl`
- test
    - number of sidecars (1 per replica)
    - access control
        - internal
        - egress
- security (mTLS)
- some observability (traffic metrics)
- distributed tracing (request tracking across microservices)
