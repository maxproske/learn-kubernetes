# learn-kubernetes

```sh
kubectl apply -f mongo-config.yaml
# configmap/mongo-config created

kubectl apply -f mongo-secret.yaml
# secret/mongo-secret created

kubectl apply -f mongo.yaml
# deployment.apps/mongo-deployment created
# service/mongo-service created

kubectl apply -f webapp.yaml
# deployment.apps/webapp-deployment created
# service/webapp-service created

kubectl get all
# Doesn't list configmaps or secrets
kubectl get configmap
kubectl get secret

# Get logs
kubectl logs

# Get IP
minikube ip
kubectl get node -o wide
```

## Known Issues

If you can't access the NodePort service webapp with MinikubeIP:NodePort, execute the following command:

```
minikube service webapp-service --url
```