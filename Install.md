# Minikube Tutorial

Docker, Minikube, and Kubectl are required.

Reference1:
Minikube installation: https://minikube.sigs.k8s.io/docs/start/
Reference1:
Interavtive minikube quick start https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/

## Create a Cluster with a Pod to host the application instance
A Pod is a Kubernetes abstraction that represents a group of one or more application containers, and some shared resources for those containers, include: Shared storage(Volumes), Networking(cluster IP address), Information abou(container image version/specific ports). One can also say a Pod is the basic execution unit of a Kubernetes application. Each Pod represents a part of a workload that is running on your cluster.
```
# Start the cluster, by running the minikube start command:
$ minikube start

# Get cluster info
$ kubectl cluster-info

# Shows all nodes that can be used to host our applications.
$ kubectl get nodes
```
## Deploying an App to a Pod

```
#Deploy our app on Kubernetes, ubectl create <deployment name> --image=<docker image name>
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

# Check deployments
$ kubectl get deployments

# Get Podname
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

# View tha app by doing a curl request
$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/
```

## Accessing an App on a Pod

A Pod always runs on a Node. A Node is a worker machine in Kubernetes and may be either a virtual or a physical machine, depending on the cluster. Each Node is managed by the control plane. A Node can have multiple pods, and the Kubernetes control plane automatically handles scheduling the pods across the Nodes in the cluster. The control plane's automatic scheduling takes into account the available resources on each Node.

Pods that are running inside Kubernetes are running on a private, isolated network. By default they are visible from other pods and services within the same kubernetes cluster, but not outside that network.
```
# A further look in to the info of a pod
$ kubectl describe pods

# Set proxy access to interact with the app (Pods are running in an isolated, private network)
Run proxy in termianl 2
$ kubectl proxy

# Get podname and see output
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

# View tha app by doing a curl request
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/

# Anything that the application would normally send to STDOUT becomes logs for the container within the Pod. We can retrieve these logs using the kubectl logs command.
$ kubectl logs $POD_NAME

# We can execute commands directly on the container once the Pod is up and running. This is done by using the `exec` command and use the name of the Pod as a parameter. For example, listing the environment variables.

$ kubectl exec $POD_NAME -- env

# We can also start a bash session in the Pod’s container
$ kubectl exec -ti $POD_NAME -- bash

# Exit the container
$ exit
```

## Using a Service to Expose the App - Expose the application outside the kubernetes cluster

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. It routes traffic across a set of Pods and allows pods to die and replicate in Kubernetes without impacting your application.  

A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The set of Pods targeted by a Service is usually determined by a LabelSelector. Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service. Services allow your applications to receive traffic. Services can be exposed in different ways by including:

- ClusterIP (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
- NodePort - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
- LoadBalancer - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
- ExternalName - Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up. This type requires v1.7 or higher of kube-dns, or CoreDNS version 0.0.8 or higher.

```
# Create a new service and expose it to external traffic we’ll use the expose command with NodePort as parameter (minikube does not support the LoadBalancer option yet).
$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080

# Test that the app is exposed outside of the cluster using curl, with IP of the Node and the externally exposed port (These are shown with kubectl get services)
$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ curl $(minikube ip):$NODE_PORT

# Get list of pods and existing services
$ kubectl describe deployment
$ kubectl get pods -l app=kubernetes-bootcamp
$ kubectl get services -l app=kubernetes-bootcamp

# Set labels for pods (These are shown with kubectl get pods)
$ export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

# Apply new lable to the pods
$ kubectl label pods $POD_NAME version=v1

# Delete Services with delete service command, and confirm its deleted.
$ kubectl delete service -l app=kubernetes-bootcamp

# Confirm that the service is gone
$ kubectl get services

# Confirm that route is not exposed anymore you can curl the previously exposed IP and port
$ curl $(minikube ip):$NODE_PORT

# Deleting the service does not delete the running app, this is show by:
$ kubectl exec -ti $POD_NAME -- curl localhost:8080
```

## Scale you app: Running multiple instances of an application
Scaling out a Deployment will ensure new Pods are created and scheduled to Nodes with available resources. Scaling will increase the number of Pods to the new desired state. Kubernetes also supports autoscaling of Pods. The is mainly done by the load-balancer Services have an integrated load-balancer that will distribute network traffic to all Pods of an exposed Deployment.

```
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           64s

# - NAME lists the names of the Deployments in the cluster.
# - READY shows the ratio of CURRENT/DESIRED replicas
# - UP-TO-DATE displays the number of replicas that have been updated to achieve the desired state.
# - AVAILABLE displays how many replicas of the application are available to your users.
# - AGE displays the amount of time that the application has been running.


# see the ReplicaSet
$ kubectl get rs
NAME                            DESIRED   CURRENT   READY   AGE
kubernetes-bootcamp-fb5c67579   1         1         1       117s

# Notice that the name of the ReplicaSet is always formatted as [DEPLOYMENT-NAME]-[RANDOM-STRING]
# - DESIRED displays the desired number of replicas of the application, which you define when you create the Deployment. This is the desired state.
# - CURRENT displays how many replicas are currently running.


# Now scale up replicas
$ kubectl scale deployments/kubernetes-bootcamp --replicas=4

# checking the following to see if its scaled up 
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           5m8s

$ kubectl get pods -o wide
NAME                                  READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
kubernetes-bootcamp-fb5c67579-bh9wb   1/1     Running   0          15s    172.18.0.9   minikube   <none>           <none>
kubernetes-bootcamp-fb5c67579-l59pc   1/1     Running   0          15s    172.18.0.8   minikube   <none>           <none>
kubernetes-bootcamp-fb5c67579-rjh24   1/1     Running   0          15s    172.18.0.7   minikube   <none>           <none>
kubernetes-bootcamp-fb5c67579-sb4wd   1/1     Running   0          5m4s   172.18.0.2   minikube   <none>           <none>

$ kubectl describe deployments/kubernetes-bootcamp
...
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m44s  deployment-controller  Scaled up replica set kubernetes-bootcamp-fb5c67579 to 1
  Normal  ScalingReplicaSet  114s   deployment-controller  Scaled up replica set kubernetes-bootcamp-fb5c67579 to 4
```


We futher take a look at how the load balencing is working
```
# Checking the service
$ kubectl describe services/kubernetes-bootcamp
...
Endpoints:                172.18.0.2:8080,172.18.0.7:8080,172.18.0.8:8080 + 1 more...
...

# Do multiple  curl to the exposed IP and port to see that we hit a different Pod with every request
$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-bh9wb | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-sb4wd | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-rjh24 | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-l59pc | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-l59pc | v=1
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-fb5c67579-bh9wb | v=1
```

Scale down the Service by 2, check if the number of replicas decreased to 2.
```
$ kubectl scale deployments/kubernetes-bootcamp --replicas=2

# List the Deployments to check if the change was applied with the get deployments command:
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   2/2     2            2           24m

$ kubectl get pods -o wide
NAME                                  READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
kubernetes-bootcamp-fb5c67579-bh9wb   1/1     Running   0          19m   172.18.0.9   minikube   <none>           <none>
kubernetes-bootcamp-fb5c67579-sb4wd   1/1     Running   0          24m   172.18.0.2   minikube   <none>           <none>
```



## Performing a Rolling Update: A good way of updating an application

Users expect applications to be available all the time and developers are expected to deploy new versions of them several times a day. In Kubernetes this is done with rolling updates. Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones. The new Pods will be scheduled on Nodes with available resources.

Scaling our application to run multiple instances a requirement for performing updates without affecting application availability. By default, the maximum number of Pods that can be unavailable during the update and the maximum number of new Pods that can be created, is one.

Rolling updates allow the following actions:

- Promote an application from one environment to another (via container image updates)
- Rollback to previous versions
- Continuous Integration and Continuous Delivery of applications with zero downtime

Update the image of the application to version 2, use the set image command, followed by the deployment name and the new image version(remember the previous image is `gcr.io/google-samples/kubernetes-bootcamp:v1`)
```
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2

# Verify update(not that nodePort can be found in `kubectl describe services/kubernetes-bootcamp` which shows `NodePort: <unset>  31426/TCP`)
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')

# Do multiple curls to see if every time a different pod is hit and with the latest version(v2)
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7d44784b7c-jbp2z | v=2
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7d44784b7c-qnwpx | v=2
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7d44784b7c-qnwpx | v=2
$ curl $(minikube ip):$NODE_PORT
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-7d44784b7c-xmrmb | v=2

# Confirm the update by running the rollout status
$ kubectl rollout status deployments/kubernetes-bootcamp
# Confirm the the lates image is run oin the pods(check the `Image` field of the output)
$ kubectl describe pods
```

Lets do it again with an update on an image
```
$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
deployment.apps/kubernetes-bootcamp image updated
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   3/4     2            3           23m
$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
kubernetes-bootcamp-59b7598c77-5px7q   0/1     ImagePullBackOff   0          2m24s
kubernetes-bootcamp-59b7598c77-fs6zq   0/1     ImagePullBackOff   0          2m24s
kubernetes-bootcamp-7d44784b7c-jbp2z   1/1     Running            0          19m
kubernetes-bootcamp-7d44784b7c-v5x8h   1/1     Running            0          19m
kubernetes-bootcamp-7d44784b7c-xmrmb   1/1     Running            0          18m
```
Something went wrong
```
$ kubectl describe pods
...
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  5m16s                  default-scheduler  Successfully assigned default/kubernetes-bootcamp-59b7598c77-fs6zq to minikube
  Normal   Pulling    3m50s (x4 over 5m15s)  kubelet            Pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed     3m49s (x4 over 5m15s)  kubelet            Failed to pull image "gcr.io/google-samples/kubernetes-bootcamp:v10": rpc error: code = Unknown desc = Error response from daemon: manifest for gcr.io/google-samples/kubernetes-bootcamp:v10 not found: manifest unknown: Failed to fetch "v10" from request "/v2/google-samples/kubernetes-bootcamp/manifests/v10".
  Warning  Failed     3m49s (x4 over 5m15s)  kubelet            Error: ErrImagePull
  Normal   BackOff    3m34s (x6 over 5m14s)  kubelet            Back-off pulling image "gcr.io/google-samples/kubernetes-bootcamp:v10"
  Warning  Failed     8s (x21 over 5m14s)    kubelet            Error: ImagePullBackOff
...
```
In the Events section of the output for the affected Pods, notice that the v10 image version did not exist in the repository. To roll back the deployment to your last working version, use the rollout undo command:
```
kubectl rollout undo deployments/kubernetes-bootcamp
```
The rollout undo command reverts the deployment to the previous known state (v2 of the image). Updates are versioned and you can revert to any previously known state of a deployment.

```
$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-59b7598c77-5px7q   0/1     Terminating   0          7m26s
kubernetes-bootcamp-59b7598c77-fs6zq   0/1     Terminating   0          7m26s
kubernetes-bootcamp-7d44784b7c-jbp2z   1/1     Running       0          24m
kubernetes-bootcamp-7d44784b7c-qw2d9   1/1     Running       0          4s
kubernetes-bootcamp-7d44784b7c-v5x8h   1/1     Running       0          24m
kubernetes-bootcamp-7d44784b7c-xmrmb   1/1     Running       0          24m
$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-7d44784b7c-jbp2z   1/1     Running   0          24m
kubernetes-bootcamp-7d44784b7c-qw2d9   1/1     Running   0          40s
kubernetes-bootcamp-7d44784b7c-v5x8h   1/1     Running   0          24m
kubernetes-bootcamp-7d44784b7c-xmrmb   1/1     Running   0          24m
```