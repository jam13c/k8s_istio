1. deploy a pod: `kubectl apply -f 01_pod.yaml`
    - define:
        - `apiVersion` - controls the version of the k8s api used
        - `kind` - type of resource being managed
        - `metadata` - information about the resource that the YAML is defining
            - `label` otherk8s constructs can do matches on these
            - `annotation` provideinformation to users/tools. not used to identify the pod
        - `spec` desired state of the resource.
    - List the pod `kubectl get pod echoweb` 
    - exec a bash terminal `kubectl exec -it echoweb -- bash` 
    - Delete `kubectl delete pod echoweb`

2. Deploy a replicaset `kubectl apply -f 02_replicaset.yaml`
    - Manages a group of pod replicas across a cluster
    - k8s will monitor pods and try to ensure the number of copis running matches
    - define:
        - `kind` is ReplicaSet
        - `spec` section has a `selector` which provides labels that will be used to identify pods managed
        - `template` describes the pods this replicaset will manage. Has to specify everything a pod can, so has its own  `metadata` section, and its own `spec` for the pods themself.
    - get the rs `kubectl get rs echoweb` list the pods `kubectl get pods`
    - delete one pod `kubectl delete pod echoweb-****` and show new one recreated
    - delete rs `kubectl delete rs echoweb`

3. Deploy a deployment `kubectl apply -f 03_deployment.yaml`
    - Higher level manages replixasets
    - Describes desired state
    - list the pods `kubectl get pods` and the deployment `kubectl get deployment echoweb`

4. Deploy the upgrade: `kubectl apply -f 04_deployment-upgrade.yaml`
    - quickly list the deployment again to watch it upgrade

5. Deploy a service: `kubectl apply -f 05_service.yaml`
    - Passed in the pod id on env vars
    - Includes a `NodePort` service to expose it to host on port 32003
    - Browse to `http://localhost:32003`

6. Deploy the upgrade: `kubectl apply -f 06_service-upgrade.yaml`
    - Upgrades deployment to v2
    - No change to service
    - Watch upgrade to  new version of app on `http://localhost:32003`

7. Deploy daemonset `kubectl apply -f 07_daemonset.yaml`
    - list daemonset `kubectl get ds` it exists
    - list pods `kubectl get pods` no pods created
    - point out nodeSelector and label node `kubectl label nodes docker-for-desktop needsdaemon=true`
    - reapply daemonset, pod exists

8. Deploy stateful set `kubectl apply -f 08_statefulset.yaml`
    - list statefulset `kubectl get statefulset`
    - list pods `kubectl get pods` show ordered list
    - apply update `kubectl apply -f 09_statefulset-update.yaml` 
    - list pods `kubectl get pods` show downsized in meaningful way

9. Deploy pod with volume mount `kubectl apply -f 10_volume-share.yaml`
    - bash in command window `kubectl exec -it nginxdeb -c nginx-container -- bash`
    - Show mount created with `ls` and `cd usr/share/data`
    - show content of file `cat hello.txt`
    - delete all pods `kubectl delete pods --all`
    - Redeploy without write `kubectl apply -f 11_volume-share-nowrite.yaml`
    - bash in command window `kubectl exec -it nginxdeb -c nginx-container -- bash`
    - Data has gone

10. Deploy pv, pvc, and pod with persistent storage `kubectl apply -f 12_persistent-volume.yaml`
    - show pv created `kubectl get pv` and claim `kubectl get pvc`
    - Pod has storage. `kubectl exec -it nginx -- bash`
    - cd to dir `cd usr/shared/data/`
    - write a file `echo Hello World > hi.txt`
    - delete the pod `delete pod nginx`

11. Deploy a stateful set mounted to same storage `kubectl apply -f 13_persistent-volume-statefulset.yaml`
    - Show the pods created `kubectl get pods`
    - bash to one. `kubectl exec -it nginx-1 -- bash`
    - cd to dir `cd usr/shared/data/`
    - data has persisted `ls` and `cat hi.txt`
    - delete stateful set `kubectl delete -f 13_persistent-volume-statefulset.yaml`
    - delete pv & pvc `kubectl delete pvc -all` and `kubectl delete pv -all`

12. Deploy a configmap `kubectl apply -f 14_configmap.yaml`
    - Pod has run and completed. See logs `kubectl logs configmap-example`
    - delete it `kubectl delete -f 14_configmap.yaml`
    - Deploy web app with configmap as env `kubectl apply -f 15_configmap-env.yaml`
    - port forward `kubectl port-forward echoweb 81:80`
    - Browse to http://localhost:81 to see config var passed in to `POD_ID`
    - delete it `kubectl delete -f 15_configmap-env.yaml`

    







