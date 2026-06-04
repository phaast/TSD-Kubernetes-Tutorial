# Task 2 (10 pts)

## 🚀 Overview

Real-world applications are rarely just one container. They are split into multiple microservices (e.g., a Frontend talking to a Backend). Because Pods are constantly created and destroyed in Kubernetes, their IP addresses change frequently. To solve this, Kubernetes uses Service Discovery (an internal DNS). A Pod can simply call the name of a Service, and Kubernetes will automatically route the traffic to the correct, alive Pod.

### Part 1: Deploy the Backend API

We already have our Tetris Frontend running. Now, let's deploy a simple Backend (using Nginx as a mock API) and expose it using a Service.

Create a new `backend.yaml` file with code listed below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: score-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: score-api
  template:
    metadata:
      labels:
        app: score-api
    spec:
      containers:
      - name: api
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: score-service
spec:
  selector:
    app: score-api
  ports:
    - port: 80
```

Apply the deployment using:

```bash
kubectl apply -f backend.yaml
```

### Part 2: The Ping Test (Pod-to-Pod Communication)

Now we will prove that the Tetris Frontend can talk to the Backend. We will run a command inside the Tetris container to send a network request to the backend using ONLY its Service name (`score-service`).

First, find your Tetris pod name:

```bash
kubectl get pods
```

Execute a `curl` command from inside the Tetris pod targeting the Backend Service (replace `<tetris-pod-name>`):

```bash
kubectl exec <tetris-pod-name> -- curl -s http://score-service | head -n 4
```

If you see HTML code starting with `<!DOCTYPE html>`, it means your Tetris app successfully found and communicated with the Backend over the internal network.

### Part 3: Show us it works! (Proof of completion)

We saw the HTML output in the Tetris pod, but did the request actually reach our Nginx backend? Let's check the logs of our backend containers to find the exact network request.

Run this command to check the logs of all pods labeled with `app=score-api`:

```bash
kubectl logs -l app=score-api
```

What happened?

In the output, you should see a log entry looking something like this: 
`"GET / HTTP/1.1" 200 615 "-" "curl/7.88.1"`

**How to get your points:**
To prove that your microservices are communicating correctly, **take a screenshot** of your terminal showing the output of the `kubectl logs` command with the visible `GET` request from `curl`. 

This log is the ultimate proof that your network request traveled from the Tetris frontend, was routed by the `score-service` via internal DNS, and successfully landed on your backend pod! ```