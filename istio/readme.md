1. Install CRDs: ```kubectl apply -f 01_crds.yaml```

2. Install ISTIO with default Manual TLS Authentication: ```kubectl apply -f 02_istio-demo-auth.yaml```

   -  fresh installation only
   - newly deployed workloads guaranteed to have istio sidecars installed


3. Verfy installation
   - ```kubectl get svc -n istio-system```
   - ```kubectl get pods -n istio-system```

4. Test with simple httpbin api. http://httpbin.org

   - ```kubectl apply -f 03_httpbin.yaml```
   - ```kubectl apply -f 04_httpbin-gateway.yaml```
   - test routes http://localhost:81/ip and http://localhost:81/headers

4. Deploy bookinfo application https://istio.io/docs/examples/bookinfo/noistio.svg

   - cluster has automatic sidecar injection, so label `default` namespace with istio-injection=enabled: ```kubectl label namespace default istio-injection=enabled```
   - install application ```kubectl apply -f 05_bookinfo.yaml```
   - confirm all serviced are running: ```kubectl get services```
   - confirm all pods are running: ```kubectl get pods```

5. Make the application accessible from outside the cluster
   
   - Define the ingress gateway: ```kubectl apply -f 06_bookinfo-gateway.yaml```
   - Confirm the gateway has been created: ```kubectl get gateway```
   - Get the ingress host: ```kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'```
   - Get the ingress port: ```kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name==''http2'')].port}'```
   - Get the ingress secure port: ```kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name==''https'')].port}'```

6. Check the application is available

   - http://localhost:81/productpage

7. Traffic management: 

    - Setup destination rules: ```kubectl apply -f 07_destination-rule-all-mtls.yaml```
    - Apply virtual services v1 only: ```kubectl apply -f 08_virtual-service-all-v1.yaml```
    - Show different content to specific user: ```kubectl apply -f .\09_virtual-service-reviews-test-v2.yaml```
    - Inject an abort fault ```kubectl apply -f 10_virtual-service-reviews-test-v2-abort-fault```
         - send selected traffic (by user) to `reviews` v2
         - but inject a 500 error
    - Inject a delay fault ```kubectl apply -f 10_virtual-service-ratings-test-delay.yaml```: 
         - Removes the above abort fault from `reviews` v2 - but keeps user specific redirect
         - Injects a delay fault for specified user when calling `ratings`
         - There is a bug - hardcoded timeouts have caused the `reviews` service to fail
         - when productpage calls reviews service there is a timeout of 3s (with 2 retries for total 6s)
         - https://github.com/istio/istio/blob/master/samples/bookinfo/src/productpage/productpage.py
         - Timeout between `reviews` and `ratings` is hardcoded at 10s 
         - Because of the delay we introduced, the `/productpage` times out prematurely and throws the error.

