# Git Ops: Example
Git Ops is a collection of software and practices that allow to achieve the duty of devOps via a git repository. 
The principles is that configuration is as important as the software that uses it to ensure success of a project 
as such the same practices apply. This is an example using various piece of software to ensure that with the 
right set of tool deploy on the cluster it is possible to ensure that the cluster is in sync with a git repository
which is the single source of thruth. 

This example leverages: 
- [Flux](https://github.com/fluxcd/flux-get-started)
- [Reloader](https://github.com/stakater/Reloader)
- [SealedSecret](https://github.com/bitnami-labs/sealed-secrets)

for the deployment of flux see: ../../flux it has been considered to monitor this repository but it was also configured
to refresh the kubernetes cluster every 3 minutes. 

## Scenario: 
### Manifest creation
Flux will use kubectl apply -f on each and every file it monitors
```
git add manifest.yaml
git commit -m "Handling of new manifest for any type of resource"
git push 
```
whithin N minutes (3 in our case) the resource appear.

### Manifest update 
Flux will do the same Kubernetes detect the change and act upon
```
git add manifest.yaml
git commit -m "Handling of updated manifest for any type of resource"
git push 
```

### Manifest deletion
Flux will only act if it is configured to garbage collect resources it created
```
git rm manifest.yaml
git commit -m "Deletion of a manifest"
git push
```

### How do I modify a config map or a secret in git and make sure the software uses those secret?
Kubernetes will automatically refresh the config map or secret if it is mounted as a volume. Which does not mean 
anything in general as your software within your container might not monitor for change and act upon it. Really what
you need is a redeploy, so spawn a new pod and delete the old one... But really you don't want to implement that for all
pod that needs that behaviour. Here comes ***Reloader***. 

```
kind: Deployment
metadata:
  annotations:
    configmap.reloader.stakater.com/reload: "foo-configmap"
```
Add the above to your deployment that uses foo-configmap 

```
git add manifest.yaml
git commit -m "Update of the deployment"
git push
git add foo-configmap.yaml
git commit -m "Update of the config map"
git push
```
Secret works in the same way. 

### Ah but how do I deal with my secret? I can't put them in GIT in the first place... 

True you can't put them in GIT but if they are encrypted in such a way that only the cluster can decrypt it then we are okay. 
That's exactly what private/public key are here for. So if I was able to encrypt my secret using a public key and 
the cluster hold the private key, then there is no problem to deliver it into GIT. That's what Sealed secret are for. 

Bitnami sealed secrets are encrypted using a public key. The cluster is the only holder of the private key. An operator 
in the cluster ensure that secret are decrepted when created and a related normal secret is created on the fly. Combine this 
with reloader and you can commit your secret to git but also you can use them without changing your application! 

Awesome ! 

Creation of a selead secret
```
echo -n bar | kubectl create secret generic mysecret --dry-run --from-file=foo=/dev/stdin -o yaml >mysecret.yaml
cat mysecret.yaml 
kubeseal -o yaml <mysecret.yaml >mysealedsecret.yaml 
git add mysealedsecret.yaml 
rm mysecret.yaml 
git push 
```

### What next? 
If we are missing a use case lets add it here with the solution. 
