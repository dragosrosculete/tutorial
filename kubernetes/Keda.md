## Scale pods using KEDA

Kubernetes Event-driven Autoscaling 
Used to scale application based on external events. For example AWS SQS message \
https://keda.sh/docs/2.7/concepts/scaling-deployments/ \
https://keda.sh/docs/2.7/scalers/aws-sqs/ \
https://github.com/kedacore/charts/tree/main/keda 

* Example. It is important to set the name of the object, the namespace, the name of the deployment. \
queueURL , queueLength ( this represents how many message a single pod can handle before scaling )

```
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: test
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: test
  pollingInterval: 30
  cooldownPeriod:  150
  idleReplicaCount: 0
  minReplicaCount: 0
  maxReplicaCount: 5
  fallback:
    failureThreshold: 3
    replicas: 3
  advanced:
    restoreToOriginalReplicaCount: true
    horizontalPodAutoscalerConfig:
      behavior:
        scaleDown:
          stabilizationWindowSeconds: 300
          policies:
          - type: Percent
            value: 100
            periodSeconds: 15
  triggers:
  - type: aws-sqs-queue
    metadata:
      identityOwner: operator
      queueURL: "https://sqs.eu-west-1.amazonaws.com/00000000/test"
      queueLength: "5"
      awsRegion: "eu-west-1"
```