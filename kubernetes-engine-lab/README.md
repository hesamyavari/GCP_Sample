Configure the config.yaml file.

1. Click the pencil icon at the top of Cloud Shell to launch Cloud Shell Editor.
2. At the top, click to expand the content-gcpro-developer folder, then the kubernetes-engine-lab folder, and then click the config.yaml file.
3. On line 32, replace [PROJECT_ID] with your project ID, which you'll see in the Cloud Shell.
4. In the File menu, select Save.

Build the containerized Docker image.

1. In the Cloud Shell, execute the following command:
   docker build -t la-container-image . 

Push the containerized app into the Container Registry.

1. In the Cloud Shell, configure Docker to use the gcloud command:
   gcloud auth configure-docker 
   When the warning pops up, enter Y.
2. Tag the image with the registry name (replacing <PROJECT_ID> with your project ID):
   docker tag la-container-image gcr.io/<PROJECT_ID>/la-container-image:v1 
3. Push the image to the Container Registry:
   docker push gcr.io/<PROJECT_ID>/la-container-image:v1 

Confirm the operation.

1. Head back to the GCP console.
2. Navigate to Container Registry > Images.
3. Confirm the existence of la-container-image.

Deploy the workload.

1. Navigate to Kubernetes Engine > Workloads.
2. Expose the info panel.
3. Make sure Existing container image is selected.
4. For Image path, click Select.
5. In the pop-up, expand la-container-image, select the listed container image, and click Continue.
6. Click Continue.
7. On the next page, leave the values at their default.
8. Click Deploy.

Increase the number of pods.

1. Navigate to the YAML tab.
2. From the navigation at the top, select Edit.
3. On line 17, change the number of replicas to 4.
4. Click Save.
5. Confirm the operation.

Expose the deployment.

1. On the Deployment details page, click Expose.
2. Set the Service type to Load balancer.
3. Click Expose.
4. Confirm by clicking the External endpoints IP address.
