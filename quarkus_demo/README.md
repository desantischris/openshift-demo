# Deploying & Managing a Quarkus Application on Red Hat OpenShift

Hands-on experience using the **Red Hat Developer Sandbox** (OpenShift 4.20).

## Overview

This demo shows a realistic developer workflow: rapid deployment of a Quarkus application using the OpenShift **Developer Console Catalog**, followed by production hardening and management.

1. Developer Console → **+Add** → **From Catalog** → Selected the Quarkus sample template.
2. OpenShift instantly created and deployed the app with a Deployment, Service, and Route.
3. Added **readiness** and **liveness** health checks via the UI prompt (using Quarkus built-in `/q/health/ready` and `/q/health/live` endpoints on port 8080).
4. Manually scaled replicas to demonstrate horizontal scaling and self-healing.
5. Triggered a new source-to-image build and verified a rolling update across all pods.
6. **Exported the generated Deployment YAML** to show the underlying OpenShift resources and added health checks (see `/generated-yaml/...`).
7. Monitored the app via the built-in **Observe** tab (metrics, logs, events).

## Steps
1. In the Red Hat Developer Sandbox (https://sandbox.redhat.com/), select **Openshift**.
    - Select your project.
    - Under **Getting started resources**, select **View All Samples**.
    - Select **Basic Quarkus**.
    - Name the app and click **Create**

![1. Selecting the Quarkus Sample from Catalog](1-openshift-demo-1.png)

2. The app is created and a build is initiated.
    - In the side panel, select **Add Health Checks**

![2. Build in Progress and Health Checks Prompt](1-openshift-demo-2.png)

3. Add health checks per the following:
    - Readiness Probe
      HTTP GET on path /q/health/ready
      Port: 8081 (or your app's port)
      Initial delay: 10-30 seconds (Quarkus starts fast)
      Period: 10-30 seconds
      Timeout: 1-10 seconds
      Failure threshold: 3
    - Liveness Probe
      HTTP GET on path /q/health/live
      Port: 8081 (or your app's port)
      Initial delay: 10-30 seconds (Quarkus starts fast)
      Period: 10-30 seconds
      Timeout: 1-10 seconds
      Failure threshold: 3
    - Startup Probe
      HTTP GET on path /q/health/live
      Port: 8081 (or your app's port)
      Initial delay: 10-30 seconds (Quarkus starts fast)
      Period: 10-30 seconds
      Timeout: 1-10 seconds
      Failure threshold: 3

![3. Adding Readiness and Liveness Probes](1-openshift-demo-3.png)

4. Confirm the app runs without errors/issues

![4. Route Created and Exposed](1-openshift-demo-4.png)

5. Under **Workloads > Toplogy**, select the app
    - In the side panel, select Resources
    - Click the link for the Route at the bottom.
    - Verify the sample app page loads

![5. Scaling Replicas to 3 Pods](1-openshift-demo-5.png)

6. Navigate to **Workoads > Deployments**.
    - Select the app

![6. Multiple Pods Running](1-openshift-demo-6.png)

7. Increase the pods from 1 to 3.

![7. Starting a New Build](1-openshift-demo-7.png)

8. In the **Topology** view, observe the additional pods come online.

![8. New Build Complete](1-openshift-demo-8.png)

9. In the **Topology** view, start a new build
    - Observe the build run

![9. Rolling Update in Progress](1-openshift-demo-9.png)

10. Observe the pods rolling out with the new build
    - In Topology, there should be a rolling update where old pods are gradually replaced by new ones, all ending in Ready with the updated build.

![10. All Pods Updated with New Image](1-openshift-demo-10.png)
