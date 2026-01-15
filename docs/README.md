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

### ...Without Istio:
```bash
cd manifests/no-mesh

kubectl apply -f namespace.yaml
dir_for_microservice_overlays="microservices/overlays"
kubectl kustomize "${dir_for_microservice_overlays}/microservice-a" | kubectl apply -f -
kubectl kustomize "${dir_for_microservice_overlays}/microservice-b" | kubectl apply -f -
kubectl kustomize "${dir_for_microservice_overlays}/microservice-c" | kubectl apply -f -

kubectl port-forward -n istio-experiments service/microservice-a 8081:80
curl -w '\n' http://localhost:8081/endpoint?message=welcome

kubectl delete namespace istio-experiments
```

### ...in Sidecar Mode:
```bash
...bla bla bla...
```

### ...in Ambient Mode:
```bash
...bla bla bla...
```


## Useful Information

### Docs
#### Introductory:
- [What is Istio?](https://istio.io/latest/docs/overview/what-is-istio/)
- [Why choose Istio?](https://istio.io/latest/docs/overview/why-choose-istio/)
- [Sidecar or ambient?](https://istio.io/latest/docs/overview/dataplane-modes/)
#### Both for Sidecar and Ambient Modes:
- [Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
- [Observability](https://istio.io/latest/docs/concepts/observability/)
- [Security](https://istio.io/latest/docs/concepts/security/)
- [Install the Istio CNI node agent](https://istio.io/latest/docs/setup/additional-setup/cni/)
#### For Sidecar Mode:
- [Architecture](https://istio.io/latest/docs/ops/deployment/architecture/)
#### For Ambient Mode:
- [Overview](https://istio.io/latest/docs/ambient/overview/)
- [Architecture](https://istio.io/latest/docs/ambient/architecture/)


## ToDos
- traffic (routing rules + gateway)
- security (mTLS)
- some observability (traffic metrics)
- distributed tracing (request tracking across microservices)
- retries
