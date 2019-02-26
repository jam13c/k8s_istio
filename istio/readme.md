1. Install CRDs: ```kubectl apply -f 01_crds.yaml```

2. Install ISTIO with default Manual TLS Authentication: ```kubectl apply -f 02_istio-demo-auth.yaml```

   - fresh installation only
   - newly deployed workloads guaranteed to have istio sidecars installed


3. Verfy installation
   - ```kubectl get svc -n istio-system```
   - ```kubectl get pods -n istio-system```
   - cluster has automatic sidecar injection, so label `default` namespace with istio-injection=enabled: ```kubectl label namespace default istio-injection=enabled```


4. Deploy v1 of echoweb: `kubectl apply -f 03_echoweb.yaml`
     - Nothing istio specific so far
     - Deploy the istio parts `kubectl apply -f 04_echoweb-istio.yaml`
     - Now has gateway and virtual service which points to the echoweb service
     - browse to the app http://localhost:81
     
5. Upgrade the app: `kubectl apply -f 05_echoweb-upgrade.yaml`
     - New deployment exists with pods `kubectl get pods`
     - Traffic still going to v1 due to destination rule
     - Deploy a new destination rule tocanary the release `kubectl apply -f 06_echoweb-canary.yaml`
     - traffic now 25%to new app http://localhost:81


6. Deploy bookinfo: `kubectl apply -f 07_bookinfo.yaml`
     - Deploys service & deployment for all components
     - confirm services & pods all up and running `kubectl get svc` & `kubectl get pods`
     - Deploy istio gateway & virtual service: `kubectl apply -f 08_bookinfo-gateway.yaml`
     - Create derfault destination rules for all services to use mTLS: `kubectl apply -f 09_destination-rule-all-mtls.yaml`
     - browse to the app http://localhost:81/productpage
     - refresh a few times, requests are directed evenly between versions
     - Deploy Virtual services for all v1: `kubectl apply -f 10_virtual-service-all-v1.yaml`

7. Test v2 of reviews for a specific user: `kubectl apply -f 11_virtual-service-reviews-test-v2.yaml`
     - refresh page. Still v1
     - login as `jamie` and get v2
     - logout before next step

8. Inject a fault calling reviews: `kubectl apply -f 12_virtual-service-reviews-test-v2-abort-fault.yaml`
     - login as `jamie`
     - see the error
     - logout before next step

9. Remove the fault calling reviews but add a delay calling ratings: `kubectl apply -f 13_virtual-service-ratings-test-delay.yaml`
     - normal working
     - login as `jamie`
     - expect to get review with rating but with a 7s delay
     - page actually renders after 6 seconds with error
     - we found a bug! hardcoded timeouts have caused review service to fail
          - when productpage calls reviews service there is a timeout of 3s (with 2 retries for total 6s)
         - https://github.com/istio/istio/blob/master/samples/bookinfo/src/productpage/productpage.py#L309
     - revert to v1 for all `kubectl apply -f 10_virtual-service-all-v1.yaml`

10. Deploy weighted routing `kubectl apply -f 14_virtual-service-reviews-50-v3.yaml`
     - not just on ingress. Can be applied between services

11. Port forward to Jaeger: `kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686`
     - browse to http://localhost:16886

12. Port forward to prometheus: `kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090 `
     - browse to http://localhost:9090/graph?g0.range_input=1h&g0.expr=istio_request_duration_seconds_bucket&g0.tab=0

13. Port forward to Grafana: `kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000`
     - browse to http://localhost:3000/d/1/istio-mesh-dashboard?refresh=5s&orgId=1

14. Port forward to service graph: `kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}') 8088:8088`
     - browse to http://localhost:8088/force/forcegraph.html?time_horizon=60s&filter_empty=true
     - http://localhost:8088/dotviz
     - http://localhost:8088/graph

     




