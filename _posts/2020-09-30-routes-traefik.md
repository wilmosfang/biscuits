---
layout: post
title: "Traefik routes"
date: 2020-09-30 21:00:00
image: '/assets/img/'
description: 'traefik 路由'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - traefik
 - code
categories:
 - code
twitter_text: "Traefik routes"
introduction: "Traefik routes"
---

# 前言

开启一个 **[CODE][code]** 系列，用来收集各种好用的小片段

这一篇用来展示如何通过 Traefik routes 规则进行应用层路由

> **Tip:** 此次使用的 traefic 版本为 **Traefik v2.2**

---

# 操作


## OS 环境

~~~
➜  /tmp sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.4
BuildVersion:	19E287
➜  /tmp kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.5", GitCommit:"e6503f8d8f769ace2f338794c914a96fc335df0f", GitTreeState:"clean", BuildDate:"2020-06-26T03:47:41Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"16+", GitVersion:"v1.16.6-beta.0", GitCommit:"e7f962ba86f4ce7033828210ca3556393c377bcc", GitTreeState:"clean", BuildDate:"2020-01-15T08:18:29Z", GoVersion:"go1.13.5", Compiler:"gc", Platform:"linux/amd64"}
➜  /tmp
~~~


## 依赖 

~~~
➜  /tmp docker --version
Docker version 19.03.12, build 48a66213fe
➜  /tmp kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.5", GitCommit:"e6503f8d8f769ace2f338794c914a96fc335df0f", GitTreeState:"clean", BuildDate:"2020-06-26T03:47:41Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"16+", GitVersion:"v1.16.6-beta.0", GitCommit:"e7f962ba86f4ce7033828210ca3556393c377bcc", GitTreeState:"clean", BuildDate:"2020-01-15T08:18:29Z", GoVersion:"go1.13.5", Compiler:"gc", Platform:"linux/amd64"}
➜  /tmp
~~~


## 代码

~~~
➜  spike tree
.
├── nginx.yml
├── rbac.yml
├── resource_definition.yml
├── traefik.yml
└── whoami.yml

0 directories, 5 files
➜  spike cat rbac.yml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller

rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - traefik.containo.us
    resources:
      - middlewares
      - ingressroutes
      - traefikservices
      - ingressroutetcps
      - ingressrouteudps
      - tlsoptions
      - tlsstores
    verbs:
      - get
      - list
      - watch
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
  - kind: ServiceAccount
    name: traefik-ingress-controller
    namespace: kube-system


➜  spike cat resource_definition.yml
# All resources definition must be declared
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressroutes.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRoute
    plural: ingressroutes
    singular: ingressroute
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: middlewares.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: Middleware
    plural: middlewares
    singular: middleware
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressroutetcps.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRouteTCP
    plural: ingressroutetcps
    singular: ingressroutetcp
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressrouteudps.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRouteUDP
    plural: ingressrouteudps
    singular: ingressrouteudp
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tlsoptions.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TLSOption
    plural: tlsoptions
    singular: tlsoption
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tlsstores.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TLSStore
    plural: tlsstores
    singular: tlsstore
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: traefikservices.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TraefikService
    plural: traefikservices
    singular: traefikservice
  scope: Namespaced

➜  spike cat traefik.yml
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: traefik
  namespace: kube-system
  labels:
    app: traefik
spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik
  template:
    metadata:
      labels:
        app: traefik
    spec:
      serviceAccountName: traefik-ingress-controller
      containers:
        - name: traefik
          image: traefik:v2.3
          args:
            - --entryPoints.web.address=:80
            - --entryPoints.admin.address=:8000
            - --providers.kubernetescrd
            - --accesslog
            - --log.filePath=/tmp/traefik.log
            - --log.level=DEBUG
            - --api=true
            - --api.insecure=true
          ports:
            - name: web
              containerPort: 80
            - name: admin
              containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: traefik
  namespace: kube-system
spec:
  selector:
    app: traefik
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8000
      targetPort: 8080
      name: admin
  type: NodePort
➜  spike cat nginx.yml
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
  namespace: dev
  labels:
    app: nginx

spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: dev
  labels:
    app: nginx
spec:
  ports:
    - name: http
      port: 80
      targetPort: 80
  selector:
    app: nginx
  type: ClusterIP
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: nginx-ingress
  namespace: dev
spec:
  entryPoints:
    - web
  routes:
  - match: PathPrefix(`/123`) || Host(`a.b.c`)
    kind: Rule
    services:
    - name: nginx
      port: 80
➜  spike cat whoami.yml
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: whoami
  namespace: dev
  labels:
    app: whoami

spec:
  replicas: 3
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: containous/whoami
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: whoami
  namespace: dev
  labels:
    app: whoami
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 80
  selector:
    app: whoami
  type: ClusterIP
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: spring-cloud-k8s
rules:
- apiGroups: [""]
  resources: ["secrets", "services", "pods", "configmaps", "endpoints"]
  verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: default:spring-cloud-k8s
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: spring-cloud-k8s
subjects:
- kind: ServiceAccount
  name: default
  namespace: dev
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: whoami-ingress
  namespace: dev
spec:
  entryPoints:
    - web
  routes:
  - match:  Host(`e.d.f`) || PathPrefix(`/456`)
    kind: Rule
    services:
    - name: whoami
      port: 8080
➜  spike
➜  spike tail -n 3 /etc/hosts
# To allow the same kube context to work on the host and the container:
127.0.0.1 kubernetes.docker.internal   a.b.c  e.d.f
# End of section
➜  spike
~~~


## 执行


~~~
➜  spike ping a.b.c
PING kubernetes.docker.internal (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.035 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.106 ms
^C
--- kubernetes.docker.internal ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.035/0.071/0.106/0.035 ms
➜  spike ping e.d.f
PING kubernetes.docker.internal (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.042 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.098 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.095 ms
^C
--- kubernetes.docker.internal ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.042/0.078/0.098/0.026 ms
➜  spike
➜  spike kubectl create namespace dev
namespace/dev created
➜  spike
➜  spike kubectl apply -f rbac.yml
clusterrole.rbac.authorization.k8s.io/traefik-ingress-controller created
serviceaccount/traefik-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/traefik-ingress-controller created
➜  spike kubectl apply -f resource_definition.yml
customresourcedefinition.apiextensions.k8s.io/ingressroutes.traefik.containo.us created
customresourcedefinition.apiextensions.k8s.io/middlewares.traefik.containo.us created
customresourcedefinition.apiextensions.k8s.io/ingressroutetcps.traefik.containo.us created
customresourcedefinition.apiextensions.k8s.io/ingressrouteudps.traefik.containo.us created
customresourcedefinition.apiextensions.k8s.io/tlsoptions.traefik.containo.us created
customresourcedefinition.apiextensions.k8s.io/tlsstores.traefik.containo.us created
customresourcedefinition.apiextensions.k8s.io/traefikservices.traefik.containo.us created
➜  spike kubectl apply -f traefik.yml
deployment.apps/traefik created
service/traefik created
➜  spike kubectl apply -f whoami.yml
deployment.apps/whoami created
service/whoami created
clusterrole.rbac.authorization.k8s.io/spring-cloud-k8s created
rolebinding.rbac.authorization.k8s.io/default:spring-cloud-k8s created
ingressroute.traefik.containo.us/whoami-ingress created
➜  spike kubectl apply -f nginx.yml
deployment.apps/nginx created
service/nginx created
ingressroute.traefik.containo.us/nginx-ingress created
➜  spike
➜  /tmp sudo kubectl port-forward svc/traefik 80:80  8000:8000 -n kube-system
Password:
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
Forwarding from 127.0.0.1:8000 -> 8080
Forwarding from [::1]:8000 -> 8080
...
...
...
~~~


进行访问测试

当使用 **`a.b.c`** 作为主机名时，访问到了 nginx

当使用 **`e.d.f`** 作为主机名时，访问到了 whoami

对于路径 **`/123 和 /456`** 也是一样的道理


~~~
➜  /tmp curl http://a.b.c/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
➜  /tmp curl http://e.d.f/
Hostname: whoami-5b4bb9c787-xfzwn
IP: 127.0.0.1
IP: 10.1.0.24
RemoteAddr: 10.1.0.23:53278
GET / HTTP/1.1
Host: e.d.f
User-Agent: curl/7.68.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: e.d.f
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-7888c4cdf4-pt8zk
X-Real-Ip: 127.0.0.1

➜  /tmp curl http://127.0.0.1/123
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.19.3</center>
</body>
</html>
➜  /tmp curl http://127.0.0.1/456
Hostname: whoami-5b4bb9c787-bclhn
IP: 127.0.0.1
IP: 10.1.0.25
RemoteAddr: 10.1.0.23:36618
GET /456 HTTP/1.1
Host: 127.0.0.1
User-Agent: curl/7.68.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-7888c4cdf4-pt8zk
X-Real-Ip: 127.0.0.1

➜  /tmp
~~~

多访问几次 whoami 服务

~~~
➜  spike curl http://e.d.f/
Hostname: whoami-5b4bb9c787-7v9fh
IP: 127.0.0.1
IP: 10.1.0.26
RemoteAddr: 10.1.0.23:50078
GET / HTTP/1.1
Host: e.d.f
User-Agent: curl/7.68.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: e.d.f
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-7888c4cdf4-pt8zk
X-Real-Ip: 127.0.0.1

➜  spike curl http://e.d.f/
Hostname: whoami-5b4bb9c787-xfzwn
IP: 127.0.0.1
IP: 10.1.0.24
RemoteAddr: 10.1.0.23:54426
GET / HTTP/1.1
Host: e.d.f
User-Agent: curl/7.68.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: e.d.f
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-7888c4cdf4-pt8zk
X-Real-Ip: 127.0.0.1

➜  spike curl http://e.d.f/
Hostname: whoami-5b4bb9c787-bclhn
IP: 127.0.0.1
IP: 10.1.0.25
RemoteAddr: 10.1.0.23:37662
GET / HTTP/1.1
Host: e.d.f
User-Agent: curl/7.68.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: e.d.f
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-7888c4cdf4-pt8zk
X-Real-Ip: 127.0.0.1

➜  spike curl http://e.d.f/
Hostname: whoami-5b4bb9c787-7v9fh
IP: 127.0.0.1
IP: 10.1.0.26
RemoteAddr: 10.1.0.23:50078
GET / HTTP/1.1
Host: e.d.f
User-Agent: curl/7.68.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 127.0.0.1
X-Forwarded-Host: e.d.f
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-7888c4cdf4-pt8zk
X-Real-Ip: 127.0.0.1

➜  spike
~~~
 
可以看到轮询到后面的服务器中


---

# 总结

通过 traefic 的 routes 匹配规则，可以基于请求的四层特征进行路由，从而达到 IP 复用的作用


* TOC
{:toc}

---

[code]:http://www.atomicgain.com/category/code/
