# kimport

## **Prerequisites**
### **Install kubectl client**

```
Download Instructions: https://kubernetes.io/docs/tasks/tools/install-kubectl/
```

### **Install helm3 client**

```
Download Instructions: https://github.com/helm/helm/releases
```

### **Install kubeseal client**

```
Download Instructions: https://github.com/bitnami-labs/sealed-secrets/releases
```






## **Deploy**

```
./scripts/flux-init.sh https://github.com/username/reponame.git
```

## **Backup SealedSecrets Public/Private keypair**
```
kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml >master.key
```

## **To backup from Disaster**

Using the master.key file from the above backup step, replace the newly created secrets (from a freshly installed SealedSecrets deployment) and restart the controller

```
$ kubectl apply -f master.key
$ kubectl delete pod -n kube-system -l name=sealed-secrets-controller
```

## **Get Sealed Secret Cert**
```
kubeseal --fetch-cert \
--controller-name=sealed-secrets \
--controller-namespace=kube-system \
> pub-cert.pem
```

## **Create Sealed Secrets**
```
$ kubectl create secret generic test-secret -n mynamespace \
--from-literal=username='my-app' --from-literal=password='39528$vdg7Jb' \
--dry-run -oyaml | \
kubeseal --cert pub-cert.pem --format yaml \
> mysealedsecret.yaml
```

### **Create Sealed Secret with specific scope**
_In this example i use jq to add annotation to the initial manifest_

https://github.com/bitnami-labs/sealed-secrets#scopes

```
$ kubectl create secret generic test-secret -n default \
--from-literal=username='my-app' --from-literal=password='39528$vdg7Jb' \
--dry-run -ojson | \
jq '.metadata += {"annotations":{"sealedsecrets.bitnami.com/cluster-wide":"true"}}' | \
kubeseal --cert pub-cert.pem --format yaml \
> mysealedsecret.yaml
```

## To reconnect fluxcd to git repo

### If you need to print the public key again to load into GitHub
```
kubectl get secrets/fluxcd-git-deploy -n fluxcd -o jsonpath='{.data.identity}'  | base64 -d | ssh-keygen -f /dev/stdin -y
```