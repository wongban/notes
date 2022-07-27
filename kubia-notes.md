```yaml
# kubia-manual.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual
  labels: # 两个标签被附加
    creation_method: manual
    env: prod
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```
创建pod
```
kubectl create -f kubia-manual.yaml
```
查看pod完整描述文件
```
kubectl get po kubia-manual -o yaml[json]
```
列出pod状态
```
kubectl get pods
kubectl get po --show-labels
kubectl get po -L creation_method,env
```
获取容器日志
```
docker logs <container id>
kubectl logs kubia-manual
多容器pod时指定容器名
kubectl logs kubia-manual -c kubia
```
向pod发送请求
```
kubectl port-forward kubia-manual 8888:8080
```
