## APP Deployment Guidance

* Using pod topology. This is used to tell Kube scheduler to try to spread the replicas among the nodes https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/ 

Adding the following under template, under spec, will try if possible to provision the replicas evenly on nodes and zones. \
Is important to use it to minimize the risk of downtime if a node goes down.
```
    spec:
      topologySpreadConstraints:
      - labelSelector:
          matchLabels:
            app: whoami
        maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
      - labelSelector:
          matchLabels:
            app: whoami
        maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
      containers:
      - name: whoami
        image: containous/whoami
```


* Define the resource requests and limits. These are defined under container. \
Below is an example to request 1CPU and 1GB memory and have a limit of 2 CPU and 2GB memory.\
Requests means the initial CPU and Memory it requires to be available on the node in order to schedule the application. \
Limits means the maximum it can use. For CPU it will begin to throttle if it reach it. For Memory, there is no throttling so the application will run out of memory (OOM).
```
    spec:   
      containers:
      - name: whoami
        image: containous/whoami
        resources:
          limits:
            cpu: 2000m
            memory: 2000Mi
          requests:
            cpu: 1000m
            memory: 1000Mi

```

* Define the readiness and liveness checks. This makes sure the application comes up and stays up. \
It is very important to have a healthy application and Kubernetes helps to achieve this.
Readiness and liveness - Each doing its own part to ensure the pods come up and serve traffic only when ready and healthy.
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

In the following example we will tell it to check port 8080 and path /actuator/health for both Readiness and Liveness check.
For Readiness it is only executed when a new pod comes up. In our example we give it an initial 10 seconds before it can start the check and it will execute the check every 10 seconds until the endpoint will returns 200 .
Until it passes the health check the pod in considered NotReady and no traffic is sent to it. This is very important because we want the pod to be healthy and ready to go before any traffic is sent to it.
```
    spec:   
      containers:
      - name: whoami
        image: containous/whoami
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 140
          periodSeconds: 5
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
```
Liveness can also check on an endpoint. This happens throughout the life of the pod. As long as the pod is considered healthy it will be in Running state. If it becomes unhealthy then it will be killed and a new pod will take it place. \
In our example we give it an initial 140 seconds before it can do any checks, this is required depending on the application as it can take some time to be ready. The failureThreshold determines how many checks it has to fail before it is considered unhealthy. 

As you can see , both checks are important and must be used together in order to achieve the best HA possible. It is also recommended to use Replicas if supported by the application to further increase HA.

> Amazing information on lifecycle of a pod. https://learnk8s.io/graceful-shutdown \
Additional info: https://youtu.be/mxEvAPQRwhw \
How to configure a Java application to be smart and make sense of the exit code: https://medium.com/trendyol-tech/graceful-shutdown-of-spring-boot-applications-in-kubernetes-f80e0b3a30b0



* Add a lifecyle preStop if the application cannot make sense of TERM signal
> Typically, the container runtime sends a TERM signal to the main process in each container. Many container runtimes respect the STOPSIGNAL value defined in the container image and send this instead of TERM. Once the grace period has expired, the KILL signal is sent to any remaining processes, and the Pod is then deleted from the API Server

When using a lifecycle, depending on the waiting time we need to adjust the **terminationGracePeriodSeconds** to a higher values
```
    spec:   
      terminationGracePeriodSeconds: 60
      containers:
      - name: whoami
        image: containous/whoami
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - sleep 30
```

* Example of a deployment containing all the above
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
  namespace: test
spec:
  selector:
    matchLabels:
      app: whoami
  replicas: 3
  template:
    metadata:
      labels:
        app: whoami
    spec:
      terminationGracePeriodSeconds: 60
      topologySpreadConstraints:
      - labelSelector:
          matchLabels:
            app: whoami
        maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
      - labelSelector:
          matchLabels:
            app: whoami
        maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
      containers:
      - name: whoami
        image: containous/whoami
        resources:
          limits:
            cpu: 2000m
            memory: 2000Mi
          requests:
            cpu: 1000m
            memory: 1000Mi
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - sleep 30
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 140
          periodSeconds: 5
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1  
```