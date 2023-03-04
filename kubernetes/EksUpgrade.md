## EKS Upgrade

Consists of Control Plane upgrade ( master nodes) and worker nodes
Can only upgrade from one minor version to another, for example from 1.21 to 1.22 and then from 1.22 to 1.23

Example . upgrade from 1.21 to 1.22 . Always check for breaking changes
Breaking changes https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.22.md \
https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html#kubernetes-1.22

* Check that the sub-systems installed also work on the new version: \
aws ingress controller \
kube2iam \
istio \
cluster autoscaler \
etc

* Check that the addons installed also work on the new version: \
vpc cni \
kube-proxy \
coredns

* Check version
```
kubectl version --short
Client Version: v1.21.0
Server Version: v1.21.14-eks-ffeb93d
```