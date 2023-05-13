### Deployment
* Deployment is a k8s object which can help in rolling out and rolling back updates
* Deployment controls replica set and replica set controls pods

![Preview](./Images/k8s-deployment.PNG)

* Lets create a manifest with some application deployment
* Check below for the manifests used to create revision 1

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  minReadySeconds: 1
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        ver: "1.23"
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
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
    - name: nginx-svc
      port: 80
      targetPort: 80
      protocol: TCP
  type: LoadBalancer

```
* apply deployment and service. Access the application

```
kubectl apply -f deploy.yml
kubectl get deploy
kubectl get svc
```
* Lets get deployment information

```
kubectl get deploy
kubectl get rs
kubectl get pods
```
* lets explore rollout command

```
kubectl rollout history deployment/nginx-deploy
```

* Lets update the specs to change image from nginx to httpd check below manifest

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  minReadySeconds: 5
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        ver: "1.23"
    spec:
      containers:
        - name: nginx
          image: httpd
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
    - name: nginx-svc
      port: 80
      targetPort: 80
      protocol: TCP
  type: LoadBalancer


```
* Now to rollback to previous versions and update multiple versions ``` kubectl rollout undo ```

```
kubectl rollout history deployment nginx-deploy
kubectl rollout status deployment nginx-deploy
```
* The change-cause is showning as none which is not good. What can be done to have a valid change cause.

### Annotations
* [Refer Here](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/) for official docs
* Check below for the manifest with change cause annotation

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  annotations:
    kubernetes.io/change-cause: "update to phpmyadmin"
spec:
  minReadySeconds: 1
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        ver: "1.23"
    spec:
      containers:
        - name: nginx
          image: phpmyadmin
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
    - name: nginx-svc
      port: 80
      targetPort: 80
      protocol: TCP
  type: LoadBalancer

```

* [Refer Here](https://azure.github.io/application-gateway-kubernetes-ingress/annotations/) for some annotations specific to azure aks ingress
* [Refer Here](https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/guide/ingress/annotations.md) for some annotations specific to aws eks ingress

### Resource Limits
* [Refer Here](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for official docs
* Lets create a new replicaset with jenkins application with following
    * lower limit => requests
        * Memory => 256 Mi
    * upper limit => limits
        * Memory => 512 Mi
* Check below for the manifest file and apply the manifest

```yml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: jenkins-rs
  labels:
    app: jenkins
spec:
  minReadySeconds: 5
  replicas: 2
  selector:
    matchLabels:
      app: jenkins
      env: qa
  template:
    metadata:
      name: jenkins-pod
      labels:
        app: jenkins
        env: qa
        jdk: "11"
        ver: lts
    spec:
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts-jdk11
          ports:
            - containerPort: 8080
              protocol: TCP
          resources:
            requests:
              memory: "256Mi"
            limits:
              memory: "512Mi"
```
* Check the resourse limit

```
kubectl top pods
```
### Daemonset
* This workload creates a Pod on every node in the k8s cluster
* This is generally used for running agents i.e. log agents, backup agents, heart beat/monitoring agents.

![Preview](./Images/daemonset.PNG)

* Lets try to write a manifest for a alpine with sleep 1d

```yml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: myagent-ds
spec:
  minReadySeconds: 5
  selector:
    matchLabels:
      app: agent
  template:
    metadata:
      name: agent-pod
      labels:
        app: agent
        version: v1.0
    spec:
      containers:
        - name: agent-pod
          image: alpine:3
          command:
            - sleep
            - 1d
```
* Another manifest with log agent

```yml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-ds
  annotations:
    kubernetes.io/change-cause: "update to latest"
spec:
  minReadySeconds: 5
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      name: fluentd
      labels:
        app: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluentd:latest
```

### Kubernetes Probes
* Types of Probes: [Refer Here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) 
   * Liveness Probe
   * Readiness Probe
   * Startup Probe
* K8s supports 3 kinds of checks
    * liveness probe: if this check fails kuberenetes will restart the container.
    * readiness probe: if this check fails the pod will be removed from service (pod will not get requests from service)
    * startup probe: This checks for startup and until startup is ok, the other checks will be paused.
* Probes or checks can be performed by 
    *  exec: run any linux/windows command which returns status/exit code.
    * http: we send http request to the application. based on status codes we can decide [Refer Here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
    * grpc: This communicates over grpc
    * tcp: send tcp request

