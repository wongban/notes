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
    livenessProbe:
      httpGet: # 一个HTTP GET存活探针
        path: /
        port: 8080
      initialDelaySeconds: 15 # 第一次探测前等待15秒
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
前一容器的日志
kubectl logs mypod --previous
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
Kubernetes可以通过存活探针（liveness probe）检查容器是否还在
运⾏。可以为pod中的每个容器单独指定存活探针。如果探测失败，
Kubernetes将定期执⾏探针并重新启动容器。

三种探测容器的机制：
1. HTTP GET探针对容器的IP地址执⾏HTTP GET请求。如果探测器收到响应，并且响应状态码不代表错误，则认为探测成功。
2. TCP套接字探针尝试与容器指定端⼜建⽴TCP连接。如果连接成功
建⽴，则探测成功。
3. Exec探针在容器内执⾏任意命令，并检查命令的退出状态码。

ReplicationController
```yml
# kubia-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3      # pod实例的目标数目
  selector:
    app: kubia     # pod选择器决定了RC的操作对象
  template:        # 创建新pod所用的pod模板
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
        ports:
        - containerPort: 8080
```
