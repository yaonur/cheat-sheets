[[kubernetes]]
[offical link](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-config-file/#edit-secret)
```yaml
apiVersion: v1  
king: Secret  
metadata:  
  name: mongo-secret  
type: Opaque  
data:  
  mongo-user: cm9vdA==  
  mongo-password: MTIz
```
```bash
echo -n uername |base64

```

```bash
kubectl apply -f secret.yaml

kubectl create secret generic my-secret --from-literal=SOME_TOKEN=somepass123

kubectl create secret generic my-secret --from-file=cert=/path/to/cert/file
```
