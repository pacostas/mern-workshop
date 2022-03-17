# MERN example: backend repository with plenty of details
Heavily modified version of the [IBM mern-app example](https://github.com/IBM-Cloud/MERN-app).

Optimised to work in local environments for demonstrative and education purposes.

### Docker

Install Docker on your Mac.

```
docker build -t backend:v1.0.0 .
docker run -d -p 27017:27017 --name mern-mongo mongo
export MONGO_URL=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mern-mongo)
docker run -p 30555:30555 -e MONGO_URL=$MONGO_URL backend:v1.0.0
```

### Kubernetes

Use the Docker system tray (click the whale in the top right corner) then go to Preferences -> Advanced -> Enable Kubernetes, choose Kubernetes as the orchestrator and to show system containers. Apply and restart Docker, waiting for Kubernetes to be reported as available.

Check `values.yaml` points to the image you've built, e.g. `backend:v1.0.0`.

Deploy MongoDB into your cluster with `helm install --name mongo --set usePassword=false,replicaSet.enabled=true,service.type=LoadBalancer,replicaSet.replicas.secondary=3 stable/mongodb`

MongoDB has a variety of [configuration options](https://github.com/helm/charts/tree/HEAD/stable/mongodb) you can provide with `--set`.

Deploy this Node.js application into your cluster with  `helm install --name backend chart/mernexample`. 

This Node.js application connects to the MongoDB service and will retry on connection loss (so the MongoDB pod can be killed).

Data from MongoDB is persisted at `$HOME/.docker/Volumes/mongo-mongodb`.

Tested on Mac with Kubernetes (docker-for-desktop).

### Deploying the frontend
See the frontend folder in this repository. It will discover this backend Kubernetes service and connect to it. The frontend listens on port 3001, the backend listens on port 30555.

### Deploying the backend
`helm install --name backend chart/backend`

### Using Postman

Assuming you've deployed to Kubernetes (notice the bigger port number here: this is the specified NodePort which will be used when you Helm install its chart):

POST to `localhost:30555/api/todos` with, for example:
```
{
	"task": "A new task"
}
```

Data is stored in MongoDB (both in the container and in the persistent volume on your local disk).

View data with the application at `localhost:30555/api/todos` (this is doing a GET through your web browser).

### Like to script things too?
`./create-todos.sh x` where x is how many todos you would like to create. A POST request is sent to the to create x amount of todos by sending POST requests to the `api/todos endpoint`.

### How do I know my data is really persisting?
- `./create-todos.sh 10`
- `curl localhost:30555/api/todos`
- Kill the MongoDB pod, for example with `kubectl delete pod -l app=mongodb`: don't worry, a new pod will take its place
- `curl localhost:30555/api/todos` should now show no data
- Wait for a new MongoDB pod to start and run successfully
- `curl localhost:30555/api/todos` should show the same data that we stored previously

## Updating todo entries
- Create a todo entry first and take a note of the generated ID
- Send a PUT request to localhost:30555/api/todos with the todo ID as a path parameter, e.g. PUT to `http://localhost:30555/api/todos/5bd6f0404a943d0018e46874`. - Send the new `task` in the body

### Cleaning up
- `curl -X DELETE localhost:30555/api/todos` will delete all data from the todos table
- `helm delete --purge mongo` uninstalls the MongoDB Helm chart
- `helm delete --purge backend` uninstalls the Node.js backend application
- `helm delete --purge frontend` uninstalls the frontend

