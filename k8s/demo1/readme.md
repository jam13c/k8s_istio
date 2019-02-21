1. deploy a pod: ```kubectl apply -f 01_pod.yaml```
    - define:
        - `apiVersion` - controls the version of the k8s api used
        - `kind` - type of resource being managed
        - `metadata` - information about the resource that the YAML is defining
            - `label` otherk8s constructs can do matches on these
            - `annotation` provideinformation to users/tools. not used to identify the pod
        - `spec` desired state of the resource.
    - List the pod `kubectl get pod nginx` 
    - exec a bash terminal `kubectl exec -it nginx -- bash` 
    - Delete `kubectl delete pod nginx`

2. Deploy a replicaset ```kubectl apply -f 02_replicaset.yaml```
    - Manages a group of pod replicas across a cluster
    - k8s will monitor pods and try to ensure the number of copis running matches
    - define:
        - `kind` is ReplicaSet
        - `spec` section has a `selector` which provides labels that will be used to identify pods managed
        - `template` describes the pods this replicaset will manage. Has to specify everything a pod can, so has its own  `metadata` section, and its own `spec` for the pods themself.
    - get the rs `kubectl get rs nginx` list the pods `kubectl get pods`
    - delete one pod `kubectl delete pod nginx-****` and show new one recreated
    - delete rs `kubectl delete rs nginx`

3. Deploy a deployment `kubectl apply -f 03_deployment.yaml`

