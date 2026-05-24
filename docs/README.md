$$ \Huge \color{#516baa} Istio: \space Sidecar \space vs \space Ambient $$
<p align="center">
    <strong>A Comparison of <a href="https://istio.io/latest/docs">Istio</a>'s Data Plane Modes: Sidecar and Ambient</strong>
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

#### ...Without Istio:
```bash
MODE="no-mesh"
```
#### ...in Sidecar Mode:
```bash
MODE="sidecar-mode"
```
#### ...in Ambient Mode:
```bash
MODE="ambient-mode"
```

### Then:

#### Set Up:
```bash
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.28.2 TARGET_ARCH=x86_64 sh -
mv ./istio-1.28.2/bin/istioctl .
rm -r ./istio-1.28.2

sudo snap install --classic concierge
cat >> concierge.yaml <<EOF
juju:
    enable: false
providers:
  k8s:
    enable: true
    bootstrap: false
    channel: 1.32-classic/stable
    features:
      local-storage:
      load-balancer:
        enabled: true
        l2-mode: true
        cidrs: 10.64.140.43/32
    bootstrap-constraints:
      root-disk: 2G
  lxd:
    enable: false
    bootstrap: false
EOF
sudo concierge prepare --trace
rm concierge.yaml

kubectl apply --force-conflicts --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/experimental-install.yaml

kubectl -n kube-system patch configmap cilium-config --type merge --patch '{"data":{"bpf-lb-sock-hostns-only":"true"}}'
kubectl -n kube-system patch configmap cilium-config --type merge --patch '{"data":{"cni-exclusive":"false"}}'
kubectl -n kube-system rollout restart daemonset cilium
```

#### Deploy:
```bash
manifest_folder="./manifests"
manifest_subfolder="${manifest_folder}/${MODE}"
subfolder_for_microservice_overlays="${manifest_subfolder}/microservice-overlays"

./istioctl install -y -f "${manifest_folder}/istio-configurations/${MODE}.yaml"

kubectl apply -f "${manifest_subfolder}/namespace.yaml"
kubectl apply -f "${manifest_subfolder}/observability"
kubectl apply -f "${manifest_subfolder}"
for microservice_overlay_suffix in a b c
do
    kubectl kustomize "${subfolder_for_microservice_overlays}/microservice-${microservice_overlay_suffix}" | kubectl apply -f -
done
```

#### Experiment:
```bash
application_namespace_name=$(grep -o 'name: .*' "${manifest_subfolder}/namespace.yaml" | cut -d ' ' -f 2)
if [ ${MODE} == "sidecar-mode" ]
then
    ingress_gateway_namespace=istio-experiments-${MODE}-istio-system
    ingress_gateway_service_name=istio-ingressgateway
else
    ingress_gateway_namespace=${application_namespace_name}
    ingress_gateway_service_name=microservices-ingress-gateway-istio
fi
ingress_gateway_ip_address=$(kubectl get services ${ingress_gateway_service_name} -n ${ingress_gateway_namespace} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

user_serviceaccount_name="super-handsome-user"
kubectl create serviceaccount -n ${ingress_gateway_namespace} ${user_serviceaccount_name}
jwt=$(kubectl create token ${user_serviceaccount_name} -n ${ingress_gateway_namespace} --duration 10m --audience istio-experiments)

# in general:
# ✅
curl -w '\n' -H "User-Agent: a-very-handsome-client" -H "Host: completely.made.up.host.com" http://${ingress_gateway_ip_address}/a?message=welcome -H "Authorization: Bearer ${jwt}"
# ⛔ forbidden user-agent header:
curl -w '\n' -H "User-Agent: an-ugly-client" -H "Host: completely.made.up.host.com" http://${ingress_gateway_ip_address}/a?message=welcome -H "Authorization: Bearer ${jwt}"
# ⛔ forbidden path routing:
curl -w '\n' -H "User-Agent: a-very-handsome-client" -H "Host: completely.made.up.host.com" http://${ingress_gateway_ip_address}/b?message=welcome -H "Authorization: Bearer ${jwt}"

# in ambient mode only:
# ⛔ no JWT:
curl -w '\n' -H "User-Agent: a-very-handsome-client" -H "Host: completely.made.up.host.com" http://${ingress_gateway_ip_address}/a?message=welcome
# ⛔ compromised JWT:
curl -w '\n' -H "User-Agent: a-very-handsome-client" -H "Host: completely.made.up.host.com" http://${ingress_gateway_ip_address}/a?message=welcome -H "Authorization: Bearer an-invalid-token"

# ⛔ after some time, token expired:
curl -w '\n' -H "User-Agent: a-very-handsome-client" -H "Host: completely.made.up.host.com" http://${ingress_gateway_ip_address}/a?message=welcome -H "Authorization: Bearer ${jwt}"

# observability:
for i in {1..100}; do curl -w '\n' -H "User-Agent: a-very-handsome-client" -H "Host: completely.made.up.host.com" http://${ingress_gateway_ip_address}/a?message=welcome -H "Authorization: Bearer ${jwt}"; done
./istioctl dashboard kiali --istioNamespace istio-experiments-${MODE}-istio-system -n ${application_namespace_name}
```

#### Tear Down:
```bash
kubectl delete -f "${manifest_subfolder}/namespace.yaml"
./istioctl uninstall -y --purge
kubectl delete namespace "istio-experiments-${MODE}-istio-system"

kubectl delete -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/experimental-install.yaml
```


## ToDos
- document
    - `kubectl get gatewayclasses`
    - show containers in A's pod: `kubectl get -n  ${application_namespace_name} -o=custom-columns="POD_NAME:.metadata.name,CONTAINERS:.spec.containers[*].name,INIT-CONTAINERS:.spec.initContainers[*].name" pods/microservice-a-7d7ddb67dc-k47p9`
        - in sidecar mode -> 1 init + 1 sidecar per replica
            - `kubectl logs -n  ${application_namespace_name} -c istio-init pods/...`
            - `kubectl logs -n  ${application_namespace_name} -c istio-proxy pods/...`
        - in ambient mode -> 1 init + 0 sidecar per replica
            - `kubectl logs -n  ${application_namespace_name} -c istio-init pods/...`
    - call B and C directly (from the gateway though `HTTPRoute`) -> 403
    - change microservice env vars to see how API calls are denied when against policies
        - modify A's environment to call C directly instead of B -> 403
        - modify A's envrionment to call enternal domains -> 403
            - all egress blocked because no external domain registered as `ServiceEntry` and `outboundTrafficPolicymode: REGISTRY_ONLY` - FIXME, it's not exactly like that, read:
                - https://blog.howardjohn.info/posts/zero-to-value/
    - intercepted API calls within the cluster to verify traffic is encrypted
        - https://www.redhat.com/en/blog/capture-packets-kubernetes-ksniff
    - distributed tracing (request tracking across microservices)
        - Kiali dashboard
        - display number of API calls from A to B (outgoing) and from B to A (incoming) and from outside the cluster to A
