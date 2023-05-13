### Deployment Strategies
* Recreate: This has a down time. Stop the application, deploy the new version and start all of the applciation instances to bring back to the working state.
* Rolling update:
    * We will define the percentage of application instances that can be down
    * What will be percentage of application instances that can be used more for new versions
* Red/Blue or Red/Green
* A/B
* [Refer Here](https://www.plutora.com/blog/deployment-strategies-6-explained-in-depth) for detailed info

### Kubernetes Deployment
* [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) for official docs
* For Deployment Strategies [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#deploymentstrategy-v1-apps)

* Lets try to deploy the two versions.
* Check below manifest for the first version of the deployment spec.

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deploy
spec:
  minReadySeconds: 10
  replicas: 4
  selector:
    matchLabels:
      app: jenkins
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      name: jenkins-pod
      labels:
        app: jenkins
        version: "1.651.2"
    spec:
      containers:
        - name: jenkins
          image: jenkins:1.651.2
          ports:
            - containerPort: 8080
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /
              port: 8080
          livenessProbe:
            tcpSocket:
              port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-svc-lb
spec:
  type: LoadBalancer
  selector:
    app: jenkins
  ports:
    - name: webport
      port: 35000
      targetPort: 8080

```

* Lets create the deployment

```
kubectl apply -f deploy.yml
kubectl get deploy
kubectl get rs
kubectl get pods
```
* Now let us do changes deploy as version 2.

```yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deploy
spec:
  minReadySeconds: 10
  replicas: 4
  selector:
    matchLabels:
      app: jenkins
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      name: jenkins-pod
      labels:
        app: jenkins
        version: "2.60.3"
    spec:
      containers:
        - name: jenkins
          image: jenkins:2.60.3
          ports:
            - containerPort: 8080
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /
              port: 8080
          livenessProbe:
            tcpSocket:
              port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-svc-lb
spec:
  type: LoadBalancer
  selector:
    app: jenkins
  ports:
    - name: webport
      port: 35000
      targetPort: 8080
```

* Execute the below commands

```
kubectl rollout history deployments/jenkins-deploy
```
* Undo deployment

```
kubectl rollout undo deployments/jenkins-deploy --to-revision=1
kubectl rollout status deployments/jenkins-deploy
```