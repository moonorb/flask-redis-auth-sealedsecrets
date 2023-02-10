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

#Create namespace and encode mock password(which we are trying to hide in the first place by sealing). 
```
kubectl create ns flask
ubuntu@testvm1:~/test$ echo -n 'qwerty123' | base64
cXdlcnR5MTIz
```

#Create a yaml file secret. Since this is the file we are trying to protect, it must not be on Git hence it will be in gitignore. 
#Name this file: ```secret.yaml```
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

#seal and apply the secret(encrypt the secret). 
#We are not applying secret.yaml which must be in your .gitignore file so its not commited to git. 
 ```
kubeseal --scope cluster-wide < api.secret.yaml -oyaml >sealed-secret.yaml
kubectl apply -f sealed-secret.yaml
 ```

#This will create the regular secret with the same name which is now controlled by the sealed secret. 
 ```
ubuntu@testvm1:~/test$ kubectl get secret -nflask
NAME          TYPE     DATA   AGE
credentials   Opaque   1      20s

ubuntu@testvm1:~/test$ kubectl get sealedsecret -nflask
NAME          AGE
credentials   24s
 ```
 
#Apply all the manifests 
```
kubectl apply -f .
-rw-rw-r-- 1 john john   90 Feb 10 07:45 api.config.yaml
-rw-rw-r-- 1 john john  739 Feb 10 07:45 api.deployment.yaml
-rw-rw-r-- 1 john john  248 Feb 10 07:45 api.service.yaml
-rw-rw-r-- 1 john john  421 Feb 10 07:45 flask-ingress.yaml
-rw-rw-r-- 1 john john  347 Feb 10 07:45 redis.deployment.yaml
-rw-rw-r-- 1 john john  158 Feb 10 07:45 redis.service.yaml
-rw-rw-r-- 1 john john 1146 Feb 10 07:45 sealed-secret.yaml
```


