# Deploying Applications into Kubernetes Cluster

## Common Kubernetes objects
- Create an NGINX pod manifest
  ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        ports:
        - containerPort: 80
            protocol: TCP
  ```
- Apply the manifest
  ```
  kubectl apply -f pod.yaml
  ```
<img width="972" alt="Screenshot 2022-06-27 at 13 27 06" src="https://user-images.githubusercontent.com/33035619/175942139-95294973-08ae-4019-819c-14476d18fe8e.png">

Fields:
  - apiVersion: Which version of the Kubernetes API you're using to create this object
  - kind: Kind of object to create
  - metadata: Data that helps uniquely identify the object
  - spec: Contains further information about the Pod. Where to find the image to run the container – (this defaults to Docker Hub), the port and protocol

## Accessing the app from the browser
- Access the pod through its IP from within the K8s cluster
  ```
  kubectl run curl --image=dareyregistry/curl -i --tty
  ```
  ```
  curl -v 172.50.202.214:80
  ```
<img width="975" alt="Screenshot 2022-06-27 at 13 27 15" src="https://user-images.githubusercontent.com/33035619/175942324-37704c53-4ed8-40ed-a0e6-92b64bf55087.png">

- Create a service to access to Nginx pod
  ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
        app: nginx-pod 
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
  ```
  ```
  kubectl apply -f nginx-service.yaml
  ```
- Update the pod manifest to include the `labels` metadata
  ```yaml
  apiVersion: v1
    kind: Pod
    metadata:
        name: nginx-pod
        labels:
            app: nginx-pod  
    spec:
        containers:
        - image: nginx:latest
          name: nginx-pod
          ports:
            - containerPort: 80
              protocol: TCP
  ```
  `kubectl apply -f nginx-pod.yaml`
- Use port forwarding to attach port 8089 on the node to port 80 on the service
  ```
  kubectl  port-forward svc/nginx-service 8089:80
  ```
 <img width="970" alt="Screenshot 2022-06-27 at 13 27 23" src="https://user-images.githubusercontent.com/33035619/175942520-349ff493-3aee-41f5-b733-8e96130a0388.png">


## Self Side Task
<img width="794" alt="Screenshot 2022-06-27 at 13 27 35" src="https://user-images.githubusercontent.com/33035619/175942593-b9dc57c8-1e4a-4013-bb01-b6cb77ab1388.png">

<img width="1006" alt="Screenshot 2022-06-27 at 13 27 41" src="https://user-images.githubusercontent.com/33035619/175942665-3c0062d9-b638-4889-b8c9-4f73c23ce612.png">

- Build the Tooling app Dockerfile and push it to Dockerhub registry
<img width="945" alt="Screenshot 2022-06-27 at 13 27 50" src="https://user-images.githubusercontent.com/33035619/175942717-93f87b27-97f0-4524-91b4-71e35819e79d.png">

- Write a Pod and a Service manifests, ensure that you can access the Tooling app’s frontend using port-forwarding feature.
  - [tooling manifest](tooling.yaml)
<img width="941" alt="Screenshot 2022-06-27 at 13 28 10" src="https://user-images.githubusercontent.com/33035619/175942776-78cad448-3965-4df6-b793-3dafe55998ec.png">

    ![](imgs/tooling.png)
<img width="1008" alt="Screenshot 2022-06-27 at 13 28 25" src="https://user-images.githubusercontent.com/33035619/175943155-0e5ff72d-f6aa-4c5a-9c4d-ab757a33ed76.png">



## Expose services using NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30080
```
<img width="548" alt="Screenshot 2022-06-27 at 13 28 34" src="https://user-images.githubusercontent.com/33035619/175943219-08eec7a6-0b6b-4661-b41b-12f0c8f35b08.png">
## Create a ReplicaSet
- Create rs.yaml
  ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: nginx-rs
    spec:
      replicas: 3
      selector:
          app: nginx-pod
      template:
          metadata:
              name: nginx-pod
              labels:
              app: nginx-pod
          spec:
          containers:
          - image: nginx:latest
              name: nginx-pod
              ports:
              - containerPort: 80
                protocol: TCP
  ```
  `kubectl apply -f rs.yaml`
- Scale the replicaset
  ```
  kubectl scale rs nginx-rs --replicas=5
  ```
<img width="976" alt="Screenshot 2022-06-27 at 13 28 41" src="https://user-images.githubusercontent.com/33035619/175943229-870e1717-8913-4167-9d58-91638994ed39.png">

## Using Deployments
- Create deployment.yaml file
  ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        tier: frontend
    spec:
      replicas: 3
      selector:
        matchLabels:
        tier: frontend
      template:
        metadata:
          labels:
            tier: frontend
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
  ```
- Scale the deployment to 15 pods

<img width="973" alt="Screenshot 2022-06-27 at 13 28 49" src="https://user-images.githubusercontent.com/33035619/175943274-e73a88a1-76ac-4482-b2a8-2aa86c625b14.png">
- List the nginx files in one of the pods
  ```
  kubectl exec -it <pod-name> bash
  ```
  ```
  ls -ltr /etc/nginx/
  ```
  ```
  cat  /etc/nginx/conf.d/default.conf 
  ```


## Persisting Data for Pods
- Scale down nginx deployment to 1 replica
- Exec into the container and install vim
- Update the content of the file and add the code below /usr/share/nginx/html/index.html
  ```html
  <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to DAREY.IO!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to DAREY.IO!</h1>
    <p>I love experiencing Kubernetes</p>

    <p>Learning by doing is absolutely the best strategy at 
    <a href="https://darey.io/">www.darey.io</a>.<br/>
    for skills acquisition
    <a href="https://darey.io/">www.darey.io</a>.</p>

    <p><em>Thank you for learning from DAREY.IO</em></p>
    </body>
    </html>
  ```
- Check the browser
<img width="975" alt="Screenshot 2022-06-27 at 13 28 59" src="https://user-images.githubusercontent.com/33035619/175943285-5527b9b3-d2c7-417b-95ec-b268146ec66c.png">
