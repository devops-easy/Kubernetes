### Kubernetes Storage
* To persist the data in the Read/Write Layer, docker has volumes.
* K8s supports volumes to persist the data. The types of volumes which are supported by k8s are
    * Volume:
        * This gives volume with the help of mnt namespace to the container.
            Volume’s lifecyle is equivalent to lifecycle of Pod
    * Ephemeral Volume:
        * This is also temporary volume used for containers where they require any persistent storage across Pod restarts/creations.
    * Persistent Volume:
        * This stores volumes and has no relation with life time of Pod.
            This uses Storage Classes which help for dynamic provisioing of storage (i.e. create azure managed disk or ebs volume or azure fileshare or aws elastic file system automatically) or admins manually provisioning storage and providing it as storage class.



### Persistent Volumes
* These volumes will have a lifetime different than Pod i.e. they exist even after pod is dead.
* Types of PVC [Refer Here](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
* Persistent Volumes in Azure [Refer Here](https://learn.microsoft.com/en-us/azure/aks/concepts-storage)
* Storage classes in Azure [Refer Here](https://learn.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes)
* Storage classes in AWS [Refer Here](https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html)

## Lets create a mysql-pod with dynamic volume provisioning
* Lets create a Persistent Volume Claim and a mysql pod claiming pvc

* Now lets modify the spec to add mysql related environment variables