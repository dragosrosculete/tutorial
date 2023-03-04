## Kubernetes Secrets

* Using secrets. Can be a a simple key and value or something more complex like the one below, where we pass an entire config. In this example it is not encrypted, just base64 encoded and saved into Kube secrets. The user still requires access to Kube secrets in order to retrieve it .

* Add a secret as a stringData
```
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
stringData:
  alertmanager.yaml: |-
    "global":
      "resolve_timeout": "5m"
    "inhibit_rules":
    - "equal":
      - "namespace"
      - "alertname"
      "source_match":
        "severity": "critical"
      "target_match_re":
        "severity": "warning|info"
    - "equal":
      - "namespace"
      - "alertname"
      "source_match":
        "severity": "warning"
      "target_match_re":
        "severity": "info"
    "receivers":
    - "name": "Default"
    - "name": "Watchdog"
    - "name": "Critical"
    "route":
      "group_by":
      - "namespace"
      "group_interval": "5m"
      "group_wait": "30s"
      "receiver": "Default"
      "repeat_interval": "12h"
      "routes":
      - "match":
          "alertname": "Watchdog"
        "receiver": "Watchdog"
      - "match":
          "severity": "critical"
        "receiver": "Critical"
type: Opaque
```

* Retrieve a secret and decode it
```
kubectl get secret -n monitoring alertmanager-main -o yaml -o jsonpath="{.data['alertmanager\.yaml']}" | base64 --decode
```

* Add a SSH KEY into Kubernetes secrets using Ansible There are 2 ways of doing this:

## Option 1 . 
* Create a vault secret env file and put the value for ssh_key2 like this:
```
ssh_key1: |
  -----BEGIN RSA PRIVATE KEY-----
  MIIEowIBAAKCAQEAlFNshaX+Agrmu1iz6mT+3luKnkOu+FweEapQMQvPgpqmjq5Pp0sIJm941tKA
  tD3gTKBGKc1rmlQ9errlkigEga6mDnz/wNE2C139hVjPV7vU1LqqRYui0090f2wEm8VnuK6bJ7Oq
  YkfftU+vrYRtRhRVIs5479vjcDaW6n8kHpzg6pt58GymjQY+5joUf6wJXKko8uguJ62RrR3d58OV
  uakfLxTi9GndC5HJvMz+eDVXrlN1le0qDp8a3WF0/TIfmGcSacZ9p14ggLMs62JY5rmwPFUDCL3H
  -----END RSA PRIVATE KEY-----
```

* Define an Ansible template this like, in our case, named secret1.yaml.j2 
```
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-test1
  namespace: default
type: Opaque
data:
  id_rsa: "{{ssh_key1 | b64encode}}"
```

* Create an Ansible task like this:
```
- name: Deploy secret
  k8s:
    state: present
    definition: "{{ lookup('template', 'secret1.yaml.j2') }}"
    validate:
      fail_on_error: yes
      strict: yes
```

## Option 2 . 
* Encode your ssh key, using base64. Example: cat id_rsa.pem | base64 . Add the encoded key into Ansible vault variable
```
ssh_key2: "LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQ0KTUlJRW93SUJBQUtDQVFFQWxGTnNoYVgrQWdybXUxaXo2bVQrM2x1S25rT3UrRndlRWFwUU1RdlBncHFtanE1UHAwc0lKbTk0MXRLQQ0KdEQzZ1RLQkdLYzFybWxROWVycmxraWdFZ2E2bURuei93TkUyQzEzOWhWalBWN
3ZVMUxxcVJZdWkwMDkwZjJ3RW04Vm51SzZiSjdPcQ0KWWtmZnRVK3ZyWVJ0UmhSVklzNTQ3OXZqY0RhVzZuOGtIcHpnNnB0NThHeW1qUVkrNWpvVWY2d0pYS2tvOHVndUo2MlJyUjNkNThPVg0KdWFrZkx4VGk5R25kQzVISnZNeitlRFZYcmxOMWxlMHFEcDhhM1dGMC9USWZtR2NTYWNaOXA"
```

* Define an Ansible template this like, in our case, named secret2.yaml.j2 
```
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-test2
  namespace: default
type: Opaque
data:
  id_rsa: "{{ssh_key2}}"
```

* Create an Ansible task like this:
```
- name: Deploy secret
  k8s:
    state: present
    definition: "{{ lookup('template', 'secret2.yaml.j2') }}"
    validate:
      fail_on_error: yes
      strict: yes
```

* Example of a deployment that uses both variants. The keys will be /etc/ssh_key1/id_rsa and /etc/ssh_key2/id_rsa
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: ssh-key1
      mountPath: "/etc/ssh_key1"
    - name: ssh-key2
      mountPath: "/etc/ssh_key2"
  volumes:
  - name: ssh-key1
    secret:
      secretName: secret-test1
      defaultMode: 0400
  - name: ssh-key2
    secret:
      secretName: secret-test2
      defaultMode: 0400
```