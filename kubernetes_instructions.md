#Kubernetes

- Open source platform that coordinates a highly available cluster of computers
 that are connected to work as a single unit. It is backed by Google and RedHat.
 - Applications need to be containerized.
 - Kubernetes automates the distribution and scheduling of application containers
 across a cluster in a fairly efficient way.
 - A cluster can be deployed on either physical or virtual machines.
 
 ##Kubernetes Clusters
 
 - Kubernetes coordinates a highly available cluster of computers that are connected
 to work as a single unit.
 - The abstractions in Kubernetes allow you to deploy containerized applications to a
  cluster without tying them specifically to individual machines.
 - Kubernetes automates the distribution and scheduling of application
  containers across a cluster in a more efficient way.

 ## Minikube
 
 - Minikube is a lightweight Kubernetes implementation that creates a VM on your local machine and deploys a
  simple cluster containing only one node.
 - The Minikube CLI provides basic bootstrapping operations for working with your cluster, including start, stop,
  status, and delete.
  
 ###instructions for starting a cluster and deployment of application
 - Minikube to create clusters: 
    - ```minikube version``` to see it's there.
    - ```minikube start``` to start cluster of one node.
    - To interact with Kubernetes, kubectl command line interface is used.
    ```kubectl version``` to check if it's configured. client version is for kubctl
    and server version is for kubernetes. also see details about the build.
    - ```kubectl cluster-info``` for cluster details.
    - ```kubectl get nodes``` to view the nodes. 
 - Using kubectl to create a Deployment:
    - To deploy a containerized applications on top of the cluster, you create a
    kubernetes Deployment configuration. 
    - The master then schedules the application
    instances included in that deployment to run on individual nodes in the cluster.
    - After creating instances. If the Node hosting an instance goes down, the 
    deployment controller replaces the instance with an instance on another node.
    - deploy from docker hub using ```Kubectl create deployment <name> --image <repo:image> ```
    - to list deployments ```kubectl get deployment```, it is a docker container that is running
    - The pods run on a private, isolated network. When using **kubectl** we're using
    an API endpoint to communicate with the application.
    - ```kubectl proxy``` will forward the communications inside the pod into the cluster-wide,
    private network. The proxy can be terminated using control-C and it won't show any output
    while it's running. the output would be ```starting to serve on 127.0.0.1:8001```
    - ```curl http://localhost:8001/version``` would query the version directly through API
    - The API server will automatically create an endpoint for each pod, based on the pod name,
     that is also accessible through the proxy.
    - To get the pod name ```export POD_NAME=$(kubectl get pods -o go-template --template
     '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')``` and then ```echo Name of the Pod: $POD_NAME```
     will get the pod name and store it in the environment variable POD_NAME:, it will prong it
    - In order for the new deployment to be accessible without a proxy, a Service is
    required.
   
 ## Pods
 - A pod is an kubernetes abstraction that represents a group of one or more 
 application containers, Those resources include:
    - Shared storage, as Volumes
    - Networking, as a unique cluster IP address
    - Information about how to run each container, such as the container image version
    or specific ports to use.
    
 - A pod models an application-specific "logical host" and contain different application
 containers which are relatively tightly coupled. The containers share an IP address and port
 space, are always co-located and co-scheduled, and run in a shared context on the same Node.
 - Pods are the atomic unit on the kubernetes platform. If a node fail, identical pods are
 scheduled on other available Nodes in the cluster. 
 ## Nodes
 - a pod always run on a node. Each node(working manager) is managed by the master.
 - A node can have multiple pods, the master scheduling of the pods across the Nodes
 in the cluster, Considering the resources.
 - Each Node runs at least:
    - Kubelet, a process responsible for communication between the kubernetes Master
    and the Nodes. it manages the Pods and the containers running on the machine.
    - A container runtime responsible for pulling container image from a registry,
    unpacking the container, and running the application.
 ##Troubleshooting with Kubectl
 commands:
 - ```kubectl get``` - list resources
 - ```kubectl describe``` - show detailed information about a resource
 - ```kubectl logs``` - print the logs from a container in a pod.
 - ```kubectl exec``` - execute a command on a container in a pod.
 ##Exploring the app
 - ```kubectl get pods``` - list pods
 - ```kubectl describe pods``` - to view what containers are inside
  that Pod and what images are used to build those containers
 - ```echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; kubectl proxy```
 to start a proxy to access the pod
 - ```export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}') echo Name of the Pod: $POD_NAME```
 to export the name of the pod
 - ```curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/```
 to see the output of the application
 - get the STDOUT by ```kubectl logs $POD_NAME```
 - to list environment variables we need to execute a command in the container so
 ```kubectl exec $POD_NAME env```
 - start a bash session ```kubectl exec -ti $POD_NAME bash```
 - The source code of the NodeJS app is in the server.js file 
 - to check if the app is running ```curl localhost:8080```
 - to close connection ```exit```
 
 ##Using a service to expose Apps
 - A service is an abstraction which defines a logical set of Pods and a policy 
 by which to access them.
 - Services enable a loose coupling between dependent pods.
 - A service is defined using YAML or JSON, like all kubernetes object.
 - The set of Pods targeted by a Service is usually determined by a LabelSelector(might not 
 want to include a selector in the spec of the service)
 - The IP addresses of the pods are not exposed without a service. Services allow
 the applications to receive traffic.
 - Services can be exposed in different ways by specifying a type in the ServiceSpec:
    - ClusterIP(default) - Exposes the service on an internal IP in the cluster.
    This type makes the Service only reachable from within the cluster.
    - NodePort - Exposes the Service on the same port of each selected Node in the cluster
    using NAT. Makes a Service accessible from outside the cluster using ```<NodeIP:NodePort```
    - LoadBalancer - Creates an external load balancer in the current cloud (if supported)
    and assigns a fixed, external IP to the Service. Superset of NodePort.
    - ExternalName - Exposes the Services using an arbitrary name (specified by externalName)
    in the spec) by returning a CNAME record with tha name. Proxy is used. This type requires
    v1.7 or higher of ```kube-dns```.
    - Additionally, note that there are some use cases with Services that involve not defining selector in the spec.
     A Service created without selector will also not create the corresponding Endpoints object. This allows users to
     manually map a Service to specific endpoints. Another possibility why there may be no selector is you are strictly
     using type: ExternalName.
 - A service routes traffic across a set of Pods.
 - Services match a set of Pods using labels and selectors, a grouping primitive that allows
 operations on objects in kubernetes. Labels are key/value pairs attached to objects and
 can be used in any number of ways:
    - Designate objects for development, test, and production
    - Embed version tags
    - Classify an object using tags
    
 ###Create a new service
 - ```kubectl get services``` to list the current services
 - ```kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080```
 to create a new service and expose it with NodePort as parameter.
 - The service received a unique cluster-IP, an internal port and an external-IP(of the node)
 - ```kubectl describe services/kubernetes-bootcamp``` to find what port was opened externally 
 ( by the NodePort option)
 - ```export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') echo NODE_PORT=$NODE_PORT```    
 to export the node_port as environment variable.
 - ```curl $(minikube ip):$NODE_PORT``` to get a response from the server.

###Using labels
- The deployment created automatically a label for the pod.
```kubectl describe deployment```
- to query pods using the label ```kubectl get pods -l run=kubernetes-bootcamp```
- ```export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}') echo Name of the Pod: $POD_NAME```
for exporting pod name
- ```kubectl label pod $POD_NAME app=v1``` to apply new label for the pod
- ```kubectl describe pods $POD_NAME```
- ```kubectl get pods -l app=v1``` to query using the new label
###Delete Services
- ```kubectl delete service -l run=kubernetes-bootcamp``` to delete using labels
- ```kubectl get services``` to confirm
- to confirm that the route is not exposed ```curl $(minikube ip):$NODE_PORT```
- All of this just delete the service but not the app. to confirm that it is still
running ```kubectl exec -ti $POD_NAME curl localhost:8080```
## Scaling an App
- Services have an integrated load-balancer that will distribute network traffic to all Pods of an exposed Deployment.
 Services will monitor continuously the running Pods using endpoints, to ensure the traffic is sent only 
 to available Pods.
- ```kubectl get deployments``` to list deployments.
- to see the replica set created by the deployment run ```kubectl get rs```
- to scale the deployment to 4 replicas. ```kubectl scale deployments/kubernetes-bootcamp --replicas=4```
- ```kubectl get pods -o wide``` to check the change in the number of pods. note,
the deployments number also changed.
- ```kubectl describe deployments/kubernetes-bootcamp``` to check the deployment log.
- ```kubectl describe services/kubernetes-bootcamp``` to the find the service exposed ports
- ```export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') echo NODE_PORT=$NODE_PORT```
to export node port.
- ```curl $(minikube ip):$NODE_PORT``` the app and load balancing are working.
- ```kubectl scale deployments/kubernetes-bootcamp --replicas=2``` it will scale down

## Performing a rolling update
Allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.
it allows the following actions:
    - promote an application from one environment to another (via container image updates)
    - Rollback to previous versions
    - Continuous Integration and Continuous Delivery of applications with zero downtime.
- To update the image of the application to version 2 ```kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2```
- to have the image ```kubectl describe pods``` 
- ```kubectl get pods``` you'll see pods are terminating
- ```kubectl describe services/kubernetes-bootcamp``` to check exposed IP and Port
- ```export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}') echo NODE_PORT=$NODE_PORT```
to export node port. this command is flawed. check node port manually.
- ```kubectl rollout status deployments/kubernetes-bootcamp``` to confirm the update
- to rollback an update:
    - ```kubectl rollout status deployments/kubernetes-bootcamp``` update image
    - ```kubectl get deployments``` for status of deployments
    - ```kubectl get pods``` in both commands it seems that there is something wrong
    - ```Errimage pull``` means it could not find this image through ```kubectl describe pods```
    - ```kubectl rollout undo deployments/kubernetes-bootcamp``` to rollback to the previous version
    - ```kubectl get pods``` problem solved.
    - back to v2. see through this command ```kubectl describe pods```
    
##Installing kubernetes on ubuntu
###Single node cluster
- ```sudo snap install microk8s --classic``` to install k8s
- ```sudo usermod -a -G microk8s ubuntu``` ```sudo chown -f -R ubuntu ~/.kube``` to change permissions of user and group
- ```microk8s.status``` to verify installation
- ```microk8s.status --wait-ready``` wait until is ready
- ```sudo snap alias microk8s.kubectl kubectl``` to use kubectl
- ```microk8s.kubectl config view --raw > $HOME/.kube/config``` and .kubeconfig

###Multinode cluster
-   
- on all nodes
    - install docker
    - install https support ```sudo apt-get install -y apt-transport-https```
    - install curl ```sudo apt-get install curl```
    - add kubernetes GPG key ``` curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -```
    - add kubernetes repository ```sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"```
    - Install kubeadm,kubelet and kubectl ```sudo apt-get install -y kubelet kubeadm kubectl```
- on the master node
    - initialize master node using kubeadm ``` sudo kubeadm init --pod-network-cidr=192.168.10.0/24```
    - copy the token from the previous output
    - make a directory ```.kube``` and copy the admin.conf to that dir. ```$ sudo mkdir -p $HOME/.kube```
```$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config```
```$ sudo chown $(id -u):$(id -g) $HOME/.kube/config```
    - to check ```kubeadm version```
    - to check master node status ```kubectl get nodes```
    - ```kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"``` 
    - Configure Pod network and verify Pod namespaces. Install Weave network plugin to communicate master and worker nodes.
    run this command ```kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"```
- on the worker nodes
    - the line below is part of the weave network command in the master node
    - ```sudo kubeadm join 172.31.2.135:6443 --token snzv8q.7r0nwln1h5kranl2 --discovery-token-ca-cert-hash sha256:149e7f2c3fd75e785d8832449c31028b4c9a76c2b96f9d279677cab7ff62f8d0```
    - ```sudo kubectl get pods --all-namespaces``` to verify pods namespaces
    - ```sudo nano fosstechnix-web-pod.yaml``` to create a pod
    - ```kubectl apply -f fosstechnix-web-pod.yaml```
    - ```kubectl get pods``` 
    - ```kubectl describe pods```
    - ```kubectl get pods -o wide``` To check pods IP address and its states
    - ```kubectl delete pod fosstechnix``` or ```fosstechnix-web-pod.yml``` to delete pod.
    
#ALERT 
Follow this link for aws loadbalancer instructions [https://itnext.io/kubernetes-part-2-a-cluster-set-up-on-aws-with-aws-cloud-provider-and-aws-loadbalancer-f02c3509f2c2]
    
##FaaS
FaaS is kubernetes native
- install Faas
- Deploy using kubectl:
    - ```git clone https://github.com/openfaas/faas-netes``` clone the repository
    - deploy the whole stack ```kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml```
        - namespace for two commands:
            - ```openfaas``` for OpenFaas services
            - ```openfaas-fn``` for functions
    - Create password for the gateway:
        - generate random password ```PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)```
        - create FaaS secret in kubectl ```kubectl -n openfaas create secret generic basic-auth --from-literal=basic-auth-user=admin --from-literal=basic-auth-password="$PASSWORD"```
    - Now deploy openFaaS ```cd faas-netes && kubectl apply -f ./yaml```   