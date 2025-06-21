k8s集群 v1.33.1，nginx ingress v1.12.2 安装测试
```
wget -O nginx-ingress-deploy.yaml https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.2/deploy/static/provider/cloud/deploy.yaml
```

修改yaml适配

- 默认是云上LB，替换为NodePort

```
grep LoadBalancer nginx-ingress-deploy.yaml
sed -i 's/LoadBalancer/NodePort/g' nginx-ingress-deploy.yaml
```

- 在dnsPolicy行下，加hostNetwork，同级，使ingress-nginx-controller在所在节点，监听80，443端口

```
grep dnsPolicy nginx-ingress-deploy.yaml
hostNetwork: true

最后部署完，检查节点端口监听
# netstat -anlp |grep -w -E '80|443'
# ps -ef |grep nginx
```

- 添加ingres deployment副本，保证每个节点都有一个，不然有的节点没有ingres controller不会监听

```
找到Deployment下的spec, 添加replicas: 3
grep -n Deployment deploy.yaml.1
```

```
kubectl apply -f nginx-ingress-deploy.yaml
kubectl get pods -n ingress-nginx -o wide
```

```
结果:
NAME                                        READY   STATUS      RESTARTS   AGE   IP                NODE   NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-t7s4d        0/1     Completed   0          86s   10.240.247.195    k02    <none>           <none>
ingress-nginx-admission-patch-p782f         0/1     Completed   0          86s   10.240.247.196    k02    <none>           <none>
ingress-nginx-controller-7c85984c55-7j8vq   1/1     Running     0          86s   192.168.180.212   k02    <none>           <none>
ingress-nginx-controller-7c85984c55-qtvfq   1/1     Running     0          86s   192.168.180.213   k03    <none>           <none>
ingress-nginx-controller-7c85984c55-xxgjn   1/1     Running     0          86s   192.168.180.211   k01    <none>           <none>

# kubectl -n ingress-nginx get pod -o yaml
```

部署 nginx,tocat服务，不同域名访问显示不同，ingress分流

直接复制下面全部到 yaml

vi Deployment.yaml

```
# cat Deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat-pod
  template:
    metadata:
      labels:
        app: tomcat-pod
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5-jre10-slim
        ports:
        - containerPort: 8080
---
# cat Service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: test
spec:
  ports:
    - port: 80
      name: nginx
  clusterIP: None
  selector:
    app: nginx-pod
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  namespace: test
spec:
  selector:
    app: tomcat-pod
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
---
# cat Ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-http
  namespace: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
#    kubernetes.io/ingress.class: nginx
#    nginx.ingress.kubernetes.io/backend-protocol: HTTPS
#    nginx.ingress.kubernetes.io/ssl-redirect: 'true'
#    nginx.ingress.kubernetes.io/use-regex: 'true'
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
  - host: tomcat.test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tomcat-service
            port:
              number: 80
```

```
kubectl apply -f Deployment.yaml --dry-run=client

结果：
deployment.apps/nginx-deployment configured (dry run)
deployment.apps/tomcat-deployment configured (dry run)
service/nginx-service configured (dry run)
service/tomcat-service configured (dry run)
ingress.networking.k8s.io/ingress-http configured (dry run)

如果不一致，缺少对象， 检查yaml格式
https://www.yamllint.com/
```

```
kubectl create namespace test
kubectl apply -f Deployment.yaml
kubectl get all -n test -o wide

结果：
NAME                                    READY   STATUS    RESTARTS   AGE    IP               NODE   NOMINATED NODE   READINESS GATES
pod/nginx-deployment-866cb5c64-bcqfv    1/1     Running   0          95m    10.240.247.205   k02    <none>           <none>
pod/tomcat-deployment-875d7797b-nnc8h   1/1     Running   0          102m   10.240.88.135    k03    <none>           <none>

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/nginx-service    ClusterIP   None            <none>        80/TCP    95m   app=nginx-pod
service/tomcat-service   ClusterIP   10.96.201.128   <none>        80/TCP    95m   app=tomcat-pod

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES                  SELECTOR
deployment.apps/nginx-deployment    1/1     1            1           95m    nginx        nginx:1.17.1            app=nginx-pod
deployment.apps/tomcat-deployment   1/1     1            1           102m   tomcat       tomcat:8.5-jre10-slim   app=tomcat-pod

NAME                                          DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES                  SELECTOR
replicaset.apps/nginx-deployment-866cb5c64    1         1         1       95m    nginx        nginx:1.17.1            app=nginx-pod,pod-template-hash=866cb5c64
replicaset.apps/tomcat-deployment-875d7797b   1         1         1       102m   tomcat       tomcat:8.5-jre10-slim   app=tomcat-pod,pod-template-hash=875d7797b


集群内节点直接curl测试访问

nginx pod ip
curl -kv 10.240.247.205

tomcat pod ip
curl -kv 10.240.88.135:8080
```

ingress controller日志，查看访问原因和报错
```
kubectl logs -f svc/ingress-nginx-controller -n ingress-nginx
```

在集群节点，直接访问ingress controller svc
```
kubectl get svc -n ingress-nginx -o wide

结果：
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE   SELECTOR
ingress-nginx-controller             NodePort    10.96.65.193   <none>        80:32591/TCP,443:31284/TCP   22m   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
ingress-nginx-controller-admission   ClusterIP   10.96.42.55    <none>        443/TCP                      22m   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
```

ingress 域名分流测试
```
直接在集群节点访问 ingres svc ip
curl http://nginx.test.com/ --resolve 'nginx.test.com:80:10.96.65.193'
curl http://tomcat.test.com/ --resolve 'tomcat.test.com:80:10.96.65.193'

外部访问节点端口
curl http://tomcat.test.com/ --resolve 'tomcat.test.com:80:192.168.180.211'
```