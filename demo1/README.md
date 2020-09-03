# deployment test

```
创建: 
kubectl create namespace dev
kubectl apply -f nginx_deplment.yaml --namespace dev

查看:
kubectl get pods -n dev -o wide
kubectl describe pod nginx -n dev

扩容:
kubectl scale rs nginx -replicas=5 -n dev
kubectl get pods -n dev -o wide

del:
kubectl delete -f nginx_deplment.yaml -n dev
```