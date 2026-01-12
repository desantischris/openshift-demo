# Item 4: OpenShift Pipelines (Tekton CI/CD) Demo

This project demonstrates deploying a s2i golang builder app

## Lessons Learned
- Knative serverless scales to zero when idle – access the Route to trigger pods.
- PipelineRuns are instances of the pipeline definition – new runs pull the latest code.
- Sandbox limits auto-triggers so manual PipelineRun start required.

## Generated Resources
Exported YAML in `generated-yaml/`:
- [pipeline.yaml](generated-yaml/pipeline.yaml)
- [pipelinerun.yaml](generated-yaml/pipelinerun.yaml)
- [knative-service.yaml](generated-yaml/knative-service.yaml)

1. In the Red Hat Developer Sandbox (https://sandbox.redhat.com/), click **+ Add** in the left navigation.
   - Select **Import from Git**.
   - **(Optional but recommended)** Fork the repo [[https://github.com/sclorg/golang-ex](https://github.com/sclorg/golang-ex)](https://github.com/sclorg/golang-ex) to your GitHub account.
   - Select **Go** for Builder Image.
   - Select **Build Using Pipelines** for Build Option. Ensure **s2i-go-knative** is selected for Pipeline.
   - Select **Serverless Deployment** for Resource Type.
   - Ensure **Create a route to the application** is checked.
   - Click **Create**.

![1. Import from Git](4-openshift-pipeline-01.png)

2. Topolgy view loads. Verify Resource was added and PipelineRun was initiated.

![2. Resource loaded](4-openshift-pipeline-02.png)

3. Observe the PipelineRun until it completes.
  - There will be two builds: one for the image, and one for the deployment

![3. PipelineRun completed](4-openshift-pipeline-03.png)

4. In the side panel for the golang resource, navigate to the **Route** section. Click the Route.
  - The route opens in a new window with a short welcome message of "Hello Openshift!"

![4. Route opened](4-openshift-pipeline-04.png)

5. Close the app page. Observe the resource until the pod terminates automatically once the route is inactive for a short period of time.

![5. Pod terminates](4-openshift-pipeline-05.png)

6. View the service.
  - No pods are found
  - Services are listed as autoscaling to zero.

![6. Pod terminates](4-openshift-pipeline-06.png)

![7. Revisions will autoscale](4-openshift-pipeline-07.png)

7. Click the **Route** again.
  - The route opens in a new window with the same message.
  - A pod is loaded.

![6. Route opens](4-openshift-pipeline-08.png)

![7. Pod runs](4-openshift-pipeline-09.png)

8. Modify the `hello_openshift.go` `response` varaible with a new message in the forked repo.

![8. Response](4-openshift-pipeline-10.png)

9. Navigate to the Pipeline.
- Click the three dots icon and elect **Start**.
- Verify the following parameters and click **Start**.

![9. Response](4-openshift-pipeline-11.png)
![9. Response](4-openshift-pipeline-12.png)

10. Navigate to PipelineRuns.
  - Observe the PipelineRun execute.

![10. PipelineRun](4-openshift-pipeline-13.png)

11. Once the PipelineRun has completed, open the Route.

  - Notice the message **has not** updated.

![11. Text not updated](4-openshift-pipeline-14.png)

  - Reason: Knative sees no spec change in the Service, so it keeps using the existing revision which still points at the old image/digest. No new revision = no new pods with the updated text.
  - Any change under spec.template forces Knative to create a new revision.
  - Run the following patch to create a new revision:
    `
oc patch ksvc golang-ex-demo \
  --type='merge' \
  -p '{
    "spec": {
      "template": {
        "metadata": {
          "annotations": {
            "demo-revision": "text-update-1"
          }
        }
      }
    }
  }'
`

12. Repeat **Step 9** to start another pipeline run.
  - Observe the PipelineRun.
  - When it has completed, open the route again. The text is now updated.

![12. Text not updated](4-openshift-pipeline-15.png)
