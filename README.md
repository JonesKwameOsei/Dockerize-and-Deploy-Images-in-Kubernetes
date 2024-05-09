# Deploy-Images-in-Kubernetes using Ghost Image I have Dockerized in my Docker Hub Repo

## Dockerize Ghost Image in Docker Hub
1. Clone this repo
```
git clone <repourl>
```
2. If it is the only folder, open with:
```
code -a .
```
3. Run Docker Compose up to build and start the containers:
```
docker-compose -f docker-compose.yml up -d
```
4. Check the built images:
```
docker images
```
5. Tag and push the images to Docker Hub:
```
docker tag mysql:8.0 kwameds/mysql:latest    # replace my dockerhub username with yours in both 
docker tag ghost:latest kwameds/ghost:latest
```
6. Logon to your `Docker Hub` account.
```
docker login
```
7. Push to your `Docker Hub Repo`
```
docker push kwameds/mysql:latest    # replace my dockerhub username with yours in both 
docker push kwameds/ghost:latest
```
![docker images](https://github.com/JonesKwameOsei/Dockerize-and-Deploy-Images-in-Kubernetes/assets/81886509/1f536c33-f41e-48b5-aacb-3126b9f37f8f)<p>

![dockerhub](https://github.com/JonesKwameOsei/Dockerize-and-Deploy-Images-in-Kubernetes/assets/81886509/54b9ab7c-4f9c-43ba-9a1c-8a72633a149e)


## Deploy Dockerized Ghost in Kubernetes
1. Create a `Namespace`:
```
apiVersion: v1
kind: Namespace
metadata:
  name: assignment-1
```
2. Apply the this deployment to create the namespace 
```
kubectl apply -f namespace.yaml
```
3.  Create a MySQL Deployment:Define a Kubernetes Deployment resource for MySQL.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: assignment-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: kwameds/mysql:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: example
        ports:
        - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: assignment-1
spec:
  selector:
    app: mysql
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
```
This will ensure that MySQL runs as a separate service in your Kubernetes cluster.<p>
4. Apply the deployment
```
kubectl apply -f mysql-deployment.yaml
```
5. Reference `MySQL` in `Ghost Deployment`:Update the Ghost Deployment configuration to reference the MySQL service by its `DNS name (which is the service name defined in the YAML)`.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ghost
  namespace: assignment-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ghost
  template:
    metadata:
      labels:
        app: ghost
    spec:
      containers:
      - name: ghost
        image: kwameds/ghost:latest
        ports:
        - containerPort: 2368
        env:
        - name: database__client
          value: mysql
        - name: database__connection__host
          value: mysql-service
        - name: database__connection__user
          value: root
        - name: database__connection__password
          value: example
        - name: database__connection__database
          value: ghost
```
6. Apply deployment
```
kubectl apply -f ghost-deployment.yaml
```
7. Check the deployment status, pods.
```
kubectl get deploy
kubectl get pods
```
8. If deployment or pods creation delays, check status here:
```
Kubectl describe deploy all 
```
or
```
kubectl describe deploy all | grep ghost   # or grep mysql
```
![Ghost](https://github.com/JonesKwameOsei/Dockerize-and-Deploy-Images-in-Kubernetes/assets/81886509/8d6d33b4-8809-4c5e-9790-e7db7dfae002)

## Conclusion
Please be patient if deployment delays. It might be that you have a slow network.





