# Task 3 (15 pts)

## 🚀 Overview

Kubernetes is not just about running applications; it's about keeping them healthy and adapting to traffic automatically. In this final task, you will configure an "Auto-Pilot" for your Tetris app using two powerful features:
1. **Liveness Probes (Self-Healing):** Kubernetes will constantly check if the app is frozen or broken. If it is, Kubernetes will kill and recreate it.
2. **Horizontal Pod Autoscaler (HPA):** Kubernetes will monitor the CPU usage. If a lot of players suddenly open your game, it will automatically create more Pods to handle the load.

### Part 1 - Deploy with Probes and Resource Limits

To use Auto-Scaling, Kubernetes needs to know how much CPU your app is allowed to use. To use Self-Healing, it needs a health check.

Create a new `tetris-autopilot.yaml` file with the code below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tetris-prod
spec:
  replicas: 1
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
        resources:
          requests:
            cpu: "10m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

Apply the updated deployment:

```bash
kubectl apply -f tetris-autopilot.yaml
```

Wait a few seconds and check if the pod is running:
```bash
kubectl get pods
```

### Part 2: Testing Self-Healing (The Zombie Attack)

Let's simulate a critical application failure. The container is still running, but the game is broken. We will do this by entering the Pod and deleting the main `index.html` file.

First, copy your Tetris pod name from the previous command.
Now, execute the "attack" (replace `<pod-name>`):

```bash
kubectl exec <pod-name> -- rm /usr/share/nginx/html/index.html
```

Now, quickly run this command to watch the pod status:
```bash
kubectl get pods -w
```

What happened?
Because you deleted the index file, the internal Nginx server started returning a `404 Not Found` error. The `livenessProbe` detected this error. Within 10-15 seconds, you will see Kubernetes automatically `RESTART` your Pod to fix the application! Press `Ctrl+C` to exit the watch mode.

### Part 3: Testing Auto-Scaling (The Traffic Spike)

Now let's configure the Autoscaler. We will tell Kubernetes to keep the CPU usage around 50%. If it goes higher, it can scale up to 10 pods.

Create the HPA (Horizontal Pod Autoscaler):
```bash
kubectl autoscale deployment tetris-prod --cpu-percent=50 --min=1 --max=10
```

Now, let's simulate a massive traffic spike (like a DDoS attack) from a temporary "load-generator" pod. Run this command:

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://tetris-service; done"
```

*Note: Your terminal will get stuck in a loop. This is normal! Leave this terminal running.*

Open a **SECOND terminal window** in Killercoda (click the `+` button at the top of the terminal area) and watch the Autoscaler react in real-time:

```bash
kubectl get hpa -w
```

Wait 1-2 minutes. You will see the CPU load jump from 0% to over 100%. Shortly after, the `REPLICAS` count will start increasing from 1 to 2, 4, or even 10! Kubernetes is automatically adding more servers to save your game.

### Part 4: Show us it works! (Proof of completion)

To get your final 15 points, we need to see that your Auto-Pilot actually reacted to the traffic spike.

**How to get your points:**
1. Wait until your `kubectl get hpa -w` command shows that the `REPLICAS` column has increased to more than 1 (e.g., 3, 4, or 5).
2. **Take a screenshot** of this second terminal window showing the `get hpa` output with the high CPU percentage and the scaled-up replicas.

*(Once you have your screenshot, you can go back to the first terminal and press `Ctrl+C` to stop the load generator. The HPA will automatically scale your pods back down to 1 after a few minutes).*