fundamental question: why use Istio (or a service mesh, in general)?
answer: without requiring application code changes, Istio can provide (any of):
- traffic management:
    - service discovery (on underneath K8s')
    - routing rules
    - ingress
    - egress
    - advanced, layer-7 load balancing (unlike K8s Services')
    - traffic splitting (A/B testing, canary deployment)
    - timeouts
    - retries
    - rate limiting
    - circuit breaking
    - failovers
- security:
    - mTLS (zero-trust network: both client-service and server-service identity verification and encryption)
    - identity & certificate management
    - authentication (service-to-service, end-user)
    - authorization
- telemetry for observability:
    - metrics (latency, traffic, errors and saturation, both data-plane and control-plane)
    - distributed traces (call flows, service dependencies)
    - access logs (request details)

scalability challenges of sidecar mode -> ambient mode more efficient and cost-effective:
- opting out L7 processing when not needed
- still scaling L7 processing independent of application pods when needed
    - they could be scaled on a namespace basis, but you can choose to have even a waypoint for each microservice (still scaled independently of the microservice's replicas)
    - [ ztunnel (in rust) -> destination waypoiny (envoy) -> ztunnel (in rust) ]
      still faster than
      [ sidecar (envoy) -> sidecar (envoy) ]

why "sidecar": bcause of proxies as sidecar containers
why "ambient": Waypoint proxies, while heavier than the ztunnel overlay alone, still run as an ambient component of the infrastructure, requiring no modifications to application pods
why "ztunnel": zero-trust tunnel
why "HBONE": HTTP-Based Overlay Network Environment
why "SPIFFE": Secure Production Identity Framework For Everyone

sidecar schemas:
- https://istio.io/latest/docs/ops/deployment/architecture/arch.svg
- https://istio.io/latest/docs/concepts/security/arch-sec.svg

ambient schemas:
- https://istio.io/latest/docs/ambient/architecture/data-plane/ztunnel-datapath-1.png
- https://istio.io/latest/docs/ambient/architecture/data-plane/ztunnel-waypoint-datapath.png

traffic schemas: https://m.youtube.com/watch?v=IVADUaLqJbE&pp=ygUSaXN0aW8gYW1iaWVudCBtb2Rl
- sidecar mode
- ambient mode
    - layer-4 processing sufficient
        - same node
        - different nodes
    - layer-7 processing necessary
        - same node
        - different nodes

idea: show how istiod (control place) watches changes to custom resources (the .yaml files you apply, that define Istio's behavior) and configures proxies (data plane) accordingly
- e.g., AuthorizationPolicy: https://istio.io/latest/docs/concepts/security/authz.svg

for the demo:
- call B and C directly -> 403
- modify A's envrionment to call C directly -> 403
- show containers in A's pod(s)
- show traffic is encrypted (how easy is it, though?)

TODO: ingresses in schemas

---

funny:
- I know the temptation to sleep is strong...
- can you believe... one schema per concept?
    - I have some mental problem
- distributed traces... I challenge you to find an icon to represent distributed traces
- I know you're coding... stop coding, because this is Istio in a picture (at least, for sidecar mode)
- colorful slide, so beaufitul, am I not an artist?
