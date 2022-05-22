Introduction

we will  introduce ourselves to working with Google Cloud Deployment Manager. We  will start with a simple single-VM deployment, move on to a more complex  deployment with multiple dependencies, and finish by deploying a group  of imported template configurations in a single deployment.

Solution

Be sure to launch your lab in an incognito window (or other private web browser) to avoid issues with cached logins.

Download configuration files used for this lab

We will perform the majority of our activities in Cloud Shell.
- From the web console, open Cloud Shell from the top-right
- Download the configuration files we will use for this lab by copy/pasting the following command:
    gsutil -m cp -r gs://acg-gcp-labs-resources/deployment-manager-basic/* . 
Note: The period at the end of the command is required.

Create your first single VM deployment

Before we create our first deployment, we must first edit our configuration file to reflect the current project.
- From Cloud Shell, open the editor window by clicking the pencil icon in the top-right.
- In the new window that appears, click on vm-web.yaml.
- In the vm-web.yaml file, change all (YOUR_PROJECT) fields to reflect your current project ID. There should be 2 total.
	- Your project ID can be found here on the GCP Console page in the Project info section
- Back in the command prompt, deploy your first deployment by typing the following:
    gcloud deployment-manager deployments create single-vm --config vm-web.yaml 
- Once the deployment completes, go to  the web console and verify that a Compute Engine instance has been  created. You should be able to click the external IP address to view its  website.
- Still in the web console, delete your deployment by going to the top-left menu > Deployment Manager. Select your deployment and click DELETE.
Deploy web yaml
```
# CHANGE YOUR PROJECT ID'S BEFORE DEPLOYING
resources:
- type: compute.v1.instance
  name: web-1
  properties:
    zone: us-east1-c
    machineType: https://www.googleapis.com/compute/v1/projects/(YOUR_PROJECT)/zones/us-east1-c/machineTypes/f1-micro # CHANGE ME
    disks:
    - deviceName: base-1
      type: PERSISTENT
      boot: true
      autoDelete: true
      initializeParams:
        sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/family/debian-9
    metadata:
      items:
      - key: startup-script
        value: |
          apt update
          apt install -y apache2
          cat <<EOF > /var/www/html/index.html
          <html><body>
          <h2>Welcome to your custom website.</h2>
          <h2>VERSION 1.</h2>
          </body></html>
          EOF
    tags:
      items: ["http-server"]
    networkInterfaces:
    - network: https://www.googleapis.com/compute/v1/projects/(YOUR_PROJECT)/global/networks/default # CHANGE ME
      accessConfigs:
      - name: External NAT
        type: ONE_TO_ONE_NAT
- type: compute.v1.firewall
  name: default-allow-http 
  properties:
    targetTags: ["http-server"]
    sourceRanges: ["0.0.0.0/0"]
    allowed:
      - IPProtocol: TCP
        ports: ["80"] 
```

Deploy complex deployment with dependencies

We are now going to deploy a more complex deployment manager configuration that requires deploying resources in a certain order.
- Back in the Cloud Shell Editor, select the file depedencies.yaml.
- Under the resource for server-1, scroll down to the NetworkInterfaces  properties and notice that the network and subnetwork have a reference  field supplied. This tells Deployment Manager that deploying this  instance depends on other resources in the same deployment being created  first. You can view other dependencies throughout the rest of the  config file.
- As before, change all (YOUR_PROJECT) to your current project ID.
- Back in command prompt, deploy this configuration by typing the following:
    gcloud deployment-manager deployments create vpcs --config dependencies.yaml 
```
# CHANGE YOUR PROJECT ID'S BEFORE DEPLOYING
resources:
- type: compute.v1.instance
  name: server-1
  properties:
    zone: us-central1-a
    machineType: https://www.googleapis.com/compute/v1/projects/(YOUR_PROJECT)/zones/us-central1-a/machineTypes/f1-micro #CHANGE ME
    disks:
    - deviceName: boot
      type: PERSISTENT
      boot: true
      autoDelete: true
      initializeParams:
        sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/family/debian-9
    networkInterfaces:
    - network: $(ref.network-1.selfLink)
      subnetwork: $(ref.subnet-a.selfLink)
      accessConfigs:
      - name: External NAT
        type: ONE_TO_ONE_NAT
- type: compute.v1.network
  name: network-1
  properties:
    autoCreateSubnetworks: false
- name: subnet-a
  type: compute.v1.subnetwork
  properties:
    ipCidrRange: 10.0.1.0/24
    network: $(ref.network-1.selfLink)
    privateIpGoogleAccess: false
    region: us-central1
- name: ssh-network-1
  type: compute.v1.firewall
  properties:
    network: $(ref.network-1.selfLink)
    sourceRanges: ["0.0.0.0/0"]
    allowed:
    - IPProtocol: TCP
      ports: ["22"]
- name: icmp-network-1
  type: compute.v1.firewall
  properties:
    network: $(ref.network-1.selfLink)
    sourceRanges: ["0.0.0.0/0"]
    allowed:
    - IPProtocol: ICMP
- type: compute.v1.instance
  name: server-2
  properties:
    zone: us-east1-b
    machineType: https://www.googleapis.com/compute/v1/projects/(YOUR_PROJECT)/zones/us-east1-b/machineTypes/f1-micro #CHANGE ME
    disks:
    - deviceName: boot
      type: PERSISTENT
      boot: true
      autoDelete: true
      initializeParams:
        sourceImage: https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/family/debian-9
    networkInterfaces:
    - network: $(ref.network-2.selfLink)
      subnetwork: $(ref.subnet-b.selfLink)
      accessConfigs:
      - name: External NAT
        type: ONE_TO_ONE_NAT
- type: compute.v1.network
  name: network-2
  properties:
    autoCreateSubnetworks: false
- name: subnet-b
  type: compute.v1.subnetwork
  properties:
    ipCidrRange: 10.0.2.0/24
    network: $(ref.network-2.selfLink)
    privateIpGoogleAccess: false
    region: us-east1
- name: ssh-network-2
  type: compute.v1.firewall
  properties:
    network: $(ref.network-2.selfLink)
    sourceRanges: ["0.0.0.0/0"]
    allowed:
    - IPProtocol: TCP
      ports: ["22"]
- name: icmp-network-2
  type: compute.v1.firewall
  properties:
    network: $(ref.network-2.selfLink)
    sourceRanges: ["0.0.0.0/0"]
    allowed:
    - IPProtocol: ICMP
```

View deployment manifest and delete deployment

After deployment is complete, let's view the manifest:
- Still in command prompt, view details of your deployment:
    gcloud deployment-manager deployments describe vpcs 
- Look for a field in the form of manifest-(TIMESTAMP). Copy this field.
- Type in the following to view the full manifest, supplying the manifest ID you copied above:
    gcloud deployment-manager manifests describe (manifest-(TIMESTAMP)) --deployment vpcs 
- You can also view the same manifests in the web console > Deployment Manager.
- Delete your deployment using the following command:
    gcloud deployment-manager deployments delete vpcs 

Create deployment with templates

- In Cloud Shell, browse to your templates directory:
    cd templates 
- In the Code Editor, view the template-config.yaml and the template .jinja files.
- This time, we do not need to change  the project ID because the template files are able to automatically  resolve the current project ID.
- Deploy your template-enabled configuration by typing:
    gcloud deployment-manager deployments create templates --config template-config.yaml 
- View results in web console, and attempt to open website via external IP.
- Delete deployment using one of the methods above.
  
