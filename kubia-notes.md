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
  nodeSelector: # 节点选择器
    gpu: "true"
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```
从YAML或JSON文件创建任何资源
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
```
标签添加和修改
```
kubectl label po kubia-manual creation_method=manual
kubectl label po kubia-manual-v2 env=debug --overwrite
```
标签选择器
```
kubectl get po --show-labels
kubectl get po -L creation_method,env
kubectl get po -l creation_method=manual
kubectl get po -l env
kubectl get po -l '!env'
creation_method!=manual
env in (prod,devel)
env notin (prod,devel)
```
添加和修改注解
```
kubectl annotete pod kubia-manual mycompany.com/someannotation="foo bar"
```
获取容器日志
```
docker logs <container id>
kubectl logs kubia-manual
多容器pod时指定容器名
kubectl logs kubia-manual -c kubia
```
将本地网络端口转发到pod中的端口
```
kubectl port-forward kubia-manual 8888:8080
curl localhost:8888
```
命名空间
```yaml
# custom-namespace.yaml
apiVerison: v1
kind: Namespace
metadata:
  name: custom-namespace
```
```
kubectl create -f custom-namespace.yaml
kubectl create namespace custom-namespace
```
在命名空间中创建资源了，可以选择在matadata字段中添加一个namespace: custom-namespace属性，也可以在使用kubectl create命令创建时指定：
```
kubectl create -f kubia-manual.yaml -n custom-namespace
```
停止和移除pod
```
kubectl delete po pod1 pod2
kubectl delete po -l creation_method=manual
kubectl delete ns custom-namespace
kubectl delete po --all
kubectl delete all --all
```
