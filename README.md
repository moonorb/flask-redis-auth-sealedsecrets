# flask-redis-auth-sealedsecrets
Flask Application with Redis and Bitnami Sealed Secrets


#Download and Install kubesel
#Ref:https://github.com/bitnami-labs/sealed-secrets#linux

```
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.19.4/kubeseal-0.19.4-linux-amd64.tar.gz
tar -zxvf kubeseal-0.19.4-linux-amd64.tar.gz 
install -m 755 kubeseal /usr/local/bin/kubeseal
```

#Sealed Secrets Helm Chart
```
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets
```

Create namespace and encode mock password
```
kubectl create ns flask
ubuntu@testvm1:~/test$ echo -n 'qwerty123' | base64
cXdlcnR5MTIz
```

Create a yaml file secret. Since this is the file we are trying to protect, it must not be on Git hence it will be in gitignore. 
Name this file: ```secret.yaml```
```
apiVersion: v1
kind: Secret
metadata:
  name: credentials
  namespace: flask
type: Opaque
data:
  password: cXdlcnR5MTIz # Base64 encoded password
 ```

seal and apply the secret(encrypt the secret)
 ```
kubeseal --scope cluster-wide < api.secret.yaml -oyaml >sealed-secret.yaml
kubectl apply -f sealed-secret.yaml
 ```

This will create the regular secret with the same name which is now controlled by the sealed secret. 
 ```
ubuntu@testvm1:~/test$ kubectl get secret -nflask
NAME          TYPE     DATA   AGE
credentials   Opaque   1      20s

ubuntu@testvm1:~/test$ kubectl get sealedsecret -nflask
NAME          AGE
credentials   24s
 ```
 
 Apply 



