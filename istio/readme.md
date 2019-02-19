1. Install CRDs: ```kubectl apply -f 01_crds.yaml```

2. Install ISTIO with default Manual TLS Authentication: ```kubectl apply -f 02_istio-demo-auth.yaml```

   -  fresh installation only
   - newly deployed workloads guaranteed to have istio sidecars installed


3. Verfy installation
   - ```kubectl get svc -n istio-system```
   - ```kubectl get pods -n istio-system```

4. Deploy application

![](https://istio.io/docs/examples/bookinfo/noistio.svg)

   - cluster has automatic sidecar injection, so label `default` namespace with istio-injection=enabled: ```kubectl label namespace default istio-injection=enabled```
   - install application ```kubectl apply -f 03_bookinfo.yaml```
   - confirm all serviced are running: ```kubectl get services```
   - confirm all pods are running: ```kubectl get pods```

5. Make the application accessible from outside the cluster
   
   - Define the ingress gateway: ```kubectl apply -f 04_bookinfo-gateway.yaml```
   - Confirm the gateway has been created: ```kubectl get gateway```
   - Get the ingress host: ```kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'```
   - Get the ingress port: ```kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name==''http2'')].port}'```
   - Get the ingress secure port: ```kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name==''https'')].port}'```

6. Check the application is available

   - http://localhost/productpage