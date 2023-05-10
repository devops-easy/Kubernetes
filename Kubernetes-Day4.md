### K8s Jobs
* [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/job/) for official docs
* K8s has two types of jobs
    * Job: Run an activity/script to completion
    * CronJob: Run an activity/script to completion at specific time period or intervals.
    * Job manifest file

    ```yml
    ---
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hellojob
    spec:
     backoffLimit: 5
     template:
       metadata:
         name: jobpod
       spec:
         restartPolicy: OnFailure
         containers:
           - name: alpine
             image: alpine
             command:
               - sleep
               - 10s
    ```
    * CronJob manifest file

    ```yml
    ---
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: periodicjob
    spec:
      schedule: '* * * * *' 
      jobTemplate:
        metadata:
          name: getlivedata
        spec:
          backoffLimit: 2
          template:
            metadata:
              name: livedatapod
            spec:
              restartPolicy: OnFailure
              containers:
                - name: alpine
                  image: alpine
                  command:
                    - sleep
                    - 3s
    ```
    * For jobs restartPolicy cannot be Always as job will never finish
    * Jobs have backoffLimit to limit number of restarts and activeDeadline seconds to limit timeperiod of execution.
    * Running job and waiting for completion
    * Cronjob manifest which we have written create a job every minute and waits for completion


## Replication Controller

* [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) for official docs
* There are many cases where we would want to run multiple instance of a application.
* In k8s we run application in Pod and to set mutlple instances we use replica sets or replication controllers.
* Lets try to run 5 nginx Pods in our cluster

```yml
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  minReadySeconds: 3
  replicas: 5
  selector:
    app: nginx
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
        ver: "1.23"
    spec:
      containers:
        - name: nginx
          image: nginx:1.23
          ports:
            - containerPort: 80
              protocol: TCP
```
* Replication Controller is the first gen replica workload for k8s objects.
* Replication Controller labels can be matched only on equality not set based [Refer Here](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-references-in-api-objects)

## Replica Set

* This is succesor to Replication Controller.
* Replica Sets are used by Deployments.
* Replica Sets changes can be tracked and that is what the deployment uses.
* [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) for the official docs
* Lets create a replicaset with 4 nginx Pods.

```yml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx
spec:
  minReadySeconds: 3
  replicas: 4
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - nginx
          - webserver
      - key: ver
        operator: Exists
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
        ver: "1.23"
    spec:
      containers:
        - name: nginx
          image: nginx:1.23
          ports:
            - containerPort: 80
              protocol: TCP
```
* k8s replica set will always try to maintain the desired count. Of the desired is not matching with actual state, a new Pods can be created or existing pods can be deleted to maintain the desired state
* Scaling number of replicas
    * imperative way:

    ```
    kubectl scale rs nginx-rs --replicas=2
    kubectl scale rs nginx-rs --replicas=5
    ```
    * declartive way: Change the spec and apply the spec again.

* Exercise: Create a replica set with Pod specification with jenkins Pod and ping -c 4 google.com in alpine as init container with restart policy Never. 

```yml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: cicd
  labels:
    app: jenkins
    purpose: cicd
spec:
  replicas: 4
  selector:
    matchLabels:
      purpose: cicd
  template:
    metadata:
      labels:
        purpose: cicd
    spec:
      containers:
       - name: cicd
         image: jenkins/jenkins:lts-jdk11
         ports:
           - containerPort: 8080
             protocol: TCP
      initContainers:
       - name: ping
         image: alpine:3.17.0      
         command: ["ping" ,"-c","4","google.com"]
```    