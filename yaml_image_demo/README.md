# Item 2: YAML-Based Nginx Deployment

This project demonstrates creating a complete application stack — Deployment, Service, and Route — entirely from raw YAML  in the OpenShift console without using the catalog wizard.

# Key Features & Learnings
- Manual resource creation: All components defined in a single YAML file for full control.
- OpenShift compatibility: Used the non-root image nginxinc/nginx-unprivileged:1-alpine to satisfy Security Context Constraints (SCCs). OpenShift blocks root containers for security purposes causing the standard Nginx image to fail. Using a non-root version makes the deployment work smoothly.
- Reliability: Configured readiness and liveness probes on port 8080 (path /) to ensure pods are marked Ready and traffic is routed correctly.
- Public exposure: Created a Route to make the application accessible externally and verified the standard Nginx welcome page.

This approach is common for custom or advanced deployments in enterprise environments.

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

2. The app, service, and route are created.
    - Navigate to **Workloads > Topology** and view the new app
  
![2. View created app](1-yaml-upload-demo-2.png)

3. After a few minutes, click the **Route** link in the app properties window.
    - View the Nginx web server welcome page
    - Note: Clicking the Route link too early may show an error as the backend pods are not yet ready and Service endpoints don't exist yet. The page may cache and not load even when the pods are online and service endpoints exist. A hard refresh may clear the issue. The command ```curl http://nginx-yaml-service:8080/``` can be run inside the pod terminal to determine if the Nginx route is available.
  
![3. Nginx Start](1-yaml-upload-demo-3.png)
