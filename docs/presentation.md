fundamental question: why use Istio (or a service mesh, in general)?
answer: without requiring application code changes, Istio can provide (any of):

why "sidecar": bcause of proxies as sidecar containers

why "ambient": Waypoint proxies, while heavier than the ztunnel overlay alone, still run as an ambient component of the infrastructure, requiring no modifications to application pods

waypoint proxies can be scaled on a namespace basis, but you can choose to have even a waypoint for each microservice (still scaled independently of the microservice's replicas)

ambient schemas:
- https://istio.io/latest/docs/ambient/architecture/data-plane/ztunnel-datapath-1.png
- https://istio.io/latest/docs/ambient/architecture/data-plane/ztunnel-waypoint-datapath.png

---

for the demo:
- call B and C directly -> 403
- modify A's envrionment to call C instead of B -> 403
- modify A's envrionment to call enternal website -> 403
- show containers in A's pod
    - in sidecar mode
    - in ambient mode
- show traffic is encrypted (how easy is it, though?)

---

funny:
- I know the temptation to sleep is strong...
- can you believe... one schema per concept?
    - I have some mental problem
- distributed traces... I challenge you to find an icon to represent distributed traces
- I wanted to show you similar dashboard live from my demo, but I wasted too much time making icons and schemas for this presentation, so I copied and pasted these pictures from Istio's docs
- Istio in a picture: I know you're coding... stop coding, because this is Istio in a picture (at least, for sidecar mode)
- colorful slide, so beaufitul, am I not an artist?
- still the demo?! what time is it? I can't take it anymore
