## General Guidance

* Show the kube clusters: **kubectl config get-contexts** . The one with * represents the current one you are own.
```
	CURRENT   NAME                     CLUSTER                 AUTHINFO                NAMESPACE
	          cluster1.example.com     cluster1.example.com    cluster1.example.com     
	*         cluster2.example.com     cluster2.example.com    cluster2.example.com
```

* To switch to another cluster **kubectl config use-context** cluster1.example.com

* List namespaces
```
kubectl get namespaces
test                    Active    63d
cert-manager            Active   119d
robots-prod             Active   170d	
```

* List services from namespace test. Services are an abstract way to expose an application running on a set of Pods.
```
kubectl get svc -n robots-prod
•	NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
•	small-bot              ClusterIP   100.68.139.223   <none>        8080/TCP                    110d
•	big-bot                ClusterIP   100.66.10.84     <none>        8080/TCP                    110d
```

To get more info about resources , can use one of the following commands:
```
kubectl describe svc -n robots-prod (svc-name)
kubectl describe deployment -n robots-prod (deployment-name)
kubectl describe pod -n robots-prod (pod-name)
```

## Debugging

* How to identify a problematic pod
```
kubectl get pods -n test -o wide

NAME                                      READY   STATUS    RESTARTS   AGE     IP               NODE                                           NOMINATED NODE   READINESS GATES
test1-98f96467b-hk2f2   1/1     Running   0          6d23h   172.20.168.122   ip-172-20-171-79.eu-west-1.compute.internal    <none>           <none>
test2-98f96467b-sxmpw   0/1     Running   3          3d23h   172.20.216.37    ip-172-20-203-9.eu-west-1.compute.internal     <none>           <none>

```

If the pod is not 1/1 Running and has a different state for a longer period of time or it has “RESTARTS“ gt 0, it means that there is a problem there and should be investigated.

* How to check current application logs. You can also append **--previous** to the command to check logs of the previous pod 
```
kubectl logs -f test1-98f96467b-hk2f2 -n test
2021/09/10 13:34:26 Starting NGINX Prometheus Exporter version=0.9.0 commit=5f88afbd906baae02edfbab4f5715e06d88538a0 date=2021-03-22T20:16:09Z
2021/09/10 13:34:26 Listening on :9113
2021/09/10 13:34:26 NGINX Prometheus Exporter has successfully started
```

* Check why is pod crashing
```
kubectl describe pod -n test test2-98f96467b-sxmpw
```
Scroll through the results and look for STATE. The following will tell you the pod run out of memory.
```
    State:          Terminating
      Reason:       OOM
    Ready:          False
    Restart Count:  3
```

You can also get a message like this if it cannot get the image
```
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
```

* Restart(replace) pods of a deployment
```
kubectl rollout restart deployment test -n test
```