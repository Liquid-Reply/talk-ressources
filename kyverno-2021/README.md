# Kyverno Demo
## Install


Check the current status of the kyverno pod
```
kubectl describe pod <kyverno-pod-name> -n kyverno

kubectl logs <kyverno-pod-name> -n kyverno
```
Within the logs you see that some Cluster Policies are applied, let's have look
`k get ClusterPolicy -A`.

You cann also see the K8s attached webhooks via [kubectl view webhook](https://github.com/Trendyol/kubectl-view-webhook)
```
kubectl view-webhook
```

What does this basic rules do?
```
kubectl create deployment nginx --image=nginx
```
They audit, but they don't prevent!

## Validation
### Single Line Validation

Apply a validation policy 
```
kubectl apply -f ./validate/require-label.yaml
```
Delete and redepoy the nginx service.

Now you see that the pod can't be created. 

Apply the nginx container with a label 
```
kubectl run nginx --image nginx --labels app.kubernetes.io/name=hey-there
pod/nginx created
```
It works!

Cleanup
```
kubectl delete po nginx
kubectl delete -f ./validate/require-label.yaml
```

### Complex Validation
A restrictive policy will prevent that any container can run as root, priviliged, hosting file system and drop capabilities.

```
kustomize build https://github.com/kyverno/policies/pod-security/restricted | kubectl apply -f -
```

If you try to deploy the nginx container you will get an error that the container doesn't apply the non root policy.

Apply now the yaml from the folder ./validate/secure-nginx/nginx.yaml

The container could be deployed now, but it fails because it can't start as root. Thats a common issue with public deployments and their poor security configuration.

## Mutation
As we are all often lazy and don't want to adjust the charts by hand, we will apply the mutating policy from ./mutate/add-default-security-context.yaml to fullfill the requirements of the security context.

```
kubectl apply -f ./mutate/add-default-security-context.yaml
```

Now apply again a nginx, it will still fail because it requires root permissions, but at least Kyverno doesn't forbid us to deploy it.

Another great use case especially for corporate situations is to replace the container registry where you are pulling images from.

## Generation
Have a look at your namespaces:
```
kubectl describe ns default

kubectl create ns no-limits

kubectl describe ns no-limits
```
The output shows directly that there are no quotas set.

Let's apply the policy from ./generate
```
kubectl apply -f ./generate/resource-q.yaml
```

Check the namespaces again, any difference?

Ok, than create a new one `kubectl create ns limits` and make a describe on your new namespace. What now?

This is also very helpful for example to create network policies for any new namespace, because it directly applys an additional layer of security.

