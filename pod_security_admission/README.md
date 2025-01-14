# Demo steps

1. Create a namespace enforces the baseline Pod Security Standards (PSS)

```sh
kubectl create ns baseline
kubectl label ns baseline pod-security.kubernetes.io/enforce-version=v1.23 pod-security.kubernetes.io/enforce=baseline
```

2. Verify that the privileged pod is blocked by the baseline Pod Security Admission (PSa) check
```sh
kubectl run privileged-pod --image=busybox --privileged --dry-run=server -n baseline
```

```
Error from server (Forbidden): pods "privileged-pod" is forbidden: violates PodSecurity "baseline:v1.23": privileged (container "privileged-pod" must not set securityContext.privileged=true)
```

3. Create the policy that exempts `runAsNonRoot` control
```sh
kubectl apply -f exempt-run-as-non-root.yaml
```

4. Create pods and verify the exemption

The pod `root-pod-exempted` creation will be allowed:
```sh
kubectl apply -f root-pod-exempted.yaml -n baseline --dry-run=server
```

The pod `root-pod-forbidden` creation will be blocked as the policy does not exempt the `nginx` container image:
```sh
kubectl apply -f root-pod-forbidden.yaml -n baseline --dry-run=server
```

5. (optional) Verify the restricted PSa check

Create a namespace that enforces the restricted PSS control:
```sh
kubectl create ns restricted
kubectl label ns restricted pod-security.kubernetes.io/enforce-version=v1.23 pod-security.kubernetes.io/enforce=restricted
```

Create the same pod `root-pod-exempted` that is allowed in step 4, and it fails the restricted PSa check:

```sh
kubectl apply -f root-pod-exempted.yaml -n restricted --dry-run=server
```

```
Error from server (Forbidden): error when creating "root-pod-exempted.yaml": pods "root-pod-exempted" is forbidden: violates PodSecurity "restricted:v1.23": runAsNonRoot != true (container "nginx" must not set securityContext.runAsNonRoot=false)
```


# Cleanup
```sh
kubectl delete ns baseline restricted
kubectl delete -f exempt-run-as-non-root.yaml
```
