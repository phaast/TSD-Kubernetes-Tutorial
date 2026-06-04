# Task 1 (5 pts)

## 🚀 Overview

In Kubernetes, port-forwarding creates a direct, fragile tunnel to a single Pod. If that Pod is deleted, the tunnel breaks. In production, we use Services (NodePort). A Service acts as a stable gateway that automatically balances traffic across all available Pods, ensuring high availability even if individual Pods crash or restart.

### Part 1 - Deploy the application

We launch 3 copies of the Tetris game. The Deployment ensures that 3 replicas are always running. Create a new `tetris.yaml` file with code listed below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tetris-prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tetris
  template:
    metadata:
      labels:
        app: tetris
    spec:
      containers:
      - name: tetris-app
        image: bsord/tetris
        ports:
        - containerPort: 80
```

Apply the deployment using:

```bash
kubectl apply -f tetris.yaml
```

Now find your pod name:

```bash
kubectl get pods
```

Create the tunnel (replace `<pod-name>`):

```bash
kubectl port-forward pod/<pod-name> 8080:80 --address 0.0.0.0 &
```

It is now possible to access Tetris! In Killercoda:

- In the top-right corner, expand the `≡` button, go to `Traffic / Ports` and click on the `8080` port button.
- You can try to play a few games 😎.

Once you're finished - delete the Pod that the tunnel is connected to:

```bash
kubectl delete pod <pod-name>
```

What happened?

Run:

```bash
kubectl get pods
```

You will see that the cluster automatically created a new Pod (the Deployment worked!). However, your game in the browser stopped working (connection error). Why? Because the port-forward tunnel was "holding onto" the IP address of a Pod that no longer exists. The tunnel died along with the Pod.

### Part 2: Fix it with a Service (NodePort)

Instead of tunneling to a Pod, we create a "Reception" Service. The Reception redirects traffic to all Pods that are currently alive.

Create a new file named `tetris-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tetris-service
spec:
  type: NodePort
  selector:
    app: tetris
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
```

Apply this Service:

```bash
kubectl apply -f tetris-service.yaml
```


Open the game via port 30008 (use `Traffic / Ports` in Killercoda). 
Now, delete any Pod:

```Bash
kubectl delete pod <pod-name>
```

Why does this work?

The Service `tetris-service` does not connect to a specific Pod IP. It connects to the label (`app: tetris`). When a Pod dies, the Service simply removes it from the list of active targets and sends traffic to the other two Pods. The game works without interruption because you are connecting to a "gateway" instead of a specific "room."

This is the foundation of High Availability in Kubernetes!