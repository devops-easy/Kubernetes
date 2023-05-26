## Ingress
* To understand concept of ingress [Refer Here](https://doc.traefik.io/traefik/getting-started/concepts/)
* In k8s we have 3 major objects which will help in ingress (layer 7 loadbalancing) 
     * ingress
     * ingressController: This is a third party implementation [Refer Here](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
     * ingressClass
* K8s doesnot have controller for ingress.
* Lets create four simple applicatons [Refer Here](https://github.com/devops-easy/Kubernetes/tree/master/k8s%20Files/Day15/microservices) for changes done
* Create docker image and push them to registry
* For this classroom purpose i will be using nginx-ingress-controller [Refer Here](https://www.nginx.com/products/nginx-ingress-controller/)
* Our implementation:

![Preview](./Images/k8s-ingress-diagram.png)

* lets install nginx-ingress controller using helm

```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm upgrade --install ingress-nginx ingress-nginx \
             --repo https://kubernetes.github.io/ingress-nginx \
             --namespace ingress-nginx --create-namespace
```
![Preview](./Images/helm.png)

* After last command we see output which better copy to some notepad
* Now execute the following command to watch for external ip to nginx ingress controller
```
 kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller

```

![Preview](./Images/helm0.png)

* Get ingress classes and there should be nginx ingress class from helm chart

![Preview](./Images/helm1.png)

* ets deploy application and services. [Refer Here](https://github.com/devops-easy/Kubernetes/tree/master/k8s%20Files/Day15/ingress) for the changes


![Preview](./Images/helm2.png)


![Preview](./Images/helm3.png)

*  [Refer Here](https://github.com/devops-easy/Kubernetes/tree/master/k8s%20Files/Day15/ingress) for the manifest file for ingress
* Now create ingress object

![Preview](./Images/helm4.png)

* Get external ip of ingress controller using ``` kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller ```

![Preview](./Images/helm5.png)

![Preview](./Images/helm6.png)

* [Refer Here](https://kubernetes.io/docs/concepts/services-networking/ingress/) for official docs of ingress
