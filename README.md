# Template for deploying machine learning model to K8s

The meaning of this repo is not to develope a sophisticated model but 


## Training
```
$ cd ml-dev

# Set up docker for model dev and training
$ docker build -t "placementapp" .
$ docker run -it --rm --name placementapp -p 9696:9696 -v $(pwd):/app placementapp

# Model training
$ python train.py

# Save model weighs for deployment
$ cp model/project_one_model.pkl ../app/.
```
## Deployment(Local)

### Test flask application

Terminal 1:
```
$ cd app

# Start flask app
$ docker run -it --rm --name placementapp -p 9696:9696 placementapp
```
Terminal 2:
```
$ cd app

# Activate an environment with requests lib
$ conda activate <your-env>
$ python predict-test.py
```

### Test K8s deployment
Terminal 1:

Push docker image to docker hub
```
$ cd app

# Build deplyment docker image aand push to docker registry
$ docker build -t "placementapp" .
$ docker tag placementapp lovemormus/placementapp:1.0
$ docker push lovemormus/placementapp:1.0
```
Start cluster
```
$ minikube start
$ kubectl get all     
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   104s
```
Deploy app to pod and Expose app IP address with a load balancer service. A LoadBalancer service is the standard way to expose a service to the internet. With this method, each service gets its own IP address.
```
$ kubectl create --filename deployment.yaml
$ kubectl create --filename service.yaml
```

Get the minikube IP(external IP) once the service is exposed.
```
$ minikube service placement-app --url  
```

Terminal 2:

Change the url path in `pretict-test.py` based on the url return in terminal 1.
```
$ cd app

$ python predict-test.py

The Model Prediction for placement : {'Placement': True, 'Placement_Probability': 0.884765625}
```

(OPTIONAL) If update docker image
```
$ kubectl set image deployment.apps/placement-app placement-app=lovemormus/placementapp:1.1 

# show that the updated image is pulled
$ kubectl describe pod
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  41s   default-scheduler  Successfully assigned default/placement-app-66b77f7f48-krbh5 to minikube
  Normal  Pulling    41s   kubelet            Pulling image "lovemormus/placementapp:1.1"
  Normal  Pulled     40s   kubelet            Successfully pulled image "lovemormus/placementapp:1.1" in 951.3015ms (951.304751ms including waiting)
  Normal  Created    40s   kubelet            Created container placement-app
  Normal  Started    40s   kubelet            Started container placement-app
```

Delete local Kubernetes cluster
```
$ minikube delete
```


Reference: 
- https://www.analyticsvidhya.com/blog/2022/01/deploying-ml-models-using-kubernetes/
- https://github.com/HSubbu/AV-k8s-placement-app