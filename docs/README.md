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

```bash
cd k8s-resources
kubectl apply -f namespace.yaml
dir_for_microservice_overlays="microservices/overlays"
kubectl kustomize "${dir_for_microservice_overlays}/microservice-a" | kubectl apply -f -
kubectl kustomize "${dir_for_microservice_overlays}/microservice-b" | kubectl apply -f -
kubectl kustomize "${dir_for_microservice_overlays}/microservice-c" | kubectl apply -f -

kubectl port-forward service/microservice-a 8081:8080

curl http://localhost:8081/endpoint?message=mmm
```


## Instructions to Run the Toy Project

### ...Without Istio:
```bash
...bla bla bla...
```

### ...in Sidecar Mode:
```bash
...bla bla bla...
```

### ...in Ambient Mode:
```bash
...bla bla bla...
```
