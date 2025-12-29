# Item 2: YAML-Based Nginx Deployment

Demonstrate manual resource creation with YAML manifests.

- Used non-root image for OpenShift compatibility
- Added health checks on port 8080
- Scaled replicas
- Exposed via Route






## Overview
In this tutorial, the user will:
1. Deploy sample Quarkus app via the console catalog.
2. Add readiness/liveness health checks to ensure stability.
3. Expose the app publicly with a Route.
4. Manually scaled replicas from 1 → 3 to demonstrate horizontal scaling and self-healing.
5. Trigger a new source-to-image build and verified rolling update across all pods.
6, Monitor via the Observe tab.

## Steps
1. In the Red Hat Developer Sandbox (https://sandbox.redhat.com/), select the **+** sign in the header at the top right of the page.
    - Select **Import from YAML**
    - Paste in the following code and click **Create**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-yaml-demo
  labels:
    app: nginx-yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-yaml
  template:
    metadata:
      labels:
        app: nginx-yaml
    spec:
      containers:
        - name: nginx
          image: nginxinc/nginx-unprivileged:1-alpine
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
          readinessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 30
          resources:
            requests:
              cpu: "50m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-yaml-service
  labels:
    app: nginx-yaml
spec:
  selector:
    app: nginx-yaml
  ports:
    - name: http
      port: 8080
      targetPort: 8080
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: nginx-yaml-route
  labels:
    app: nginx-yaml
spec:
  # Optional: you can omit 'host' and let OpenShift generate one
  # host: nginx-yaml-demo.apps.<your-sandbox-domain>
  to:
    kind: Service
    name: nginx-yaml-service
  port:
    targetPort: http
```

![1. Creating from YAML import](1-yaml-upload-demo-1.png)

2. The app, service, and route are created and a build is initiated.
    - Navigate to **Workloads > Topology** and view the new app
  
![2. View created app](1-yaml-upload-demo-2.png)

3. After a few minutes, click the **Route** link in the app properties window.
    - View the Nginx web server welcome page
    - Note: clicking the link too early will generate an error.
  
![3. Nginx Start](1-yaml-upload-demo-3.png)
