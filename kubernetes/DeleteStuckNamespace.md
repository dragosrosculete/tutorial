## Delete Stuck Namespace
It may happen at some point that the namespace is stuck in terminating state. Follow the below procedure to force it.

> **Use only if there is no other way to terminate it and have already checked that there is no api service missing that is causing this issue**

### For example: 
kubectl get namespaces
```
test Terminating   1d
```

1. Dump the descriptor as JSON to a file

kubectl get namespace test -o json > test.json

```
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "creationTimestamp": "2020-10-26T06:49:40Z",
        "deletionTimestamp": "2020-11-02T07:15:10Z",
        "name": "test",
        "resourceVersion": "4139701",
        "selfLink": "/api/v1/namespaces/test",
        "uid": "78b0c371-9102-4a1d-ad73-1f5c99452aaf"
    },
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
    "status": {
        "conditions": [
            {
                "lastTransitionTime": "2020-11-02T07:15:15Z",
                "message": "Discovery failed for some groups, 1 failing: unable to retrieve the complete list of server APIs: metrics.k8s.io/v1: the server is currently unable to handle the request",
                "reason": "DiscoveryFailed",
                "status": "True",
                "type": "NamespaceDeletionDiscoveryFailure"
            },
            {
                "lastTransitionTime": "2020-11-02T07:15:20Z",
                "message": "All legacy kube types successfully parsed",
                "reason": "ParsedGroupVersions",
                "status": "False",
                "type": "NamespaceDeletionGroupVersionParsingFailure"
            },
            {
                "lastTransitionTime": "2020-11-02T07:15:20Z",
                "message": "All content successfully deleted, may be waiting on finalization",
                "reason": "ContentDeleted",
                "status": "False",
                "type": "NamespaceDeletionContentFailure"
            },
            {
                "lastTransitionTime": "2020-11-02T07:15:44Z",
                "message": "All content successfully removed",
                "reason": "ContentRemoved",
                "status": "False",
                "type": "NamespaceContentRemaining"
            },
            {
                "lastTransitionTime": "2020-11-02T07:15:20Z",
                "message": "All content-preserving finalizers finished",
                "reason": "ContentHasNoFinalizers",
                "status": "False",
                "type": "NamespaceFinalizersRemaining"
            }
        ],
        "phase": "Terminating"
    }
}
```

2. Edit the file we've just saved ( test.json ) and remove "kubernetes" line from finalizers 
```
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
```

3. Apply changes

kubectl replace --raw "/api/v1/namespaces/test/finalize" -f ./test.json