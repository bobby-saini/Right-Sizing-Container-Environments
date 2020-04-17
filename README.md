# Best Practices for Managing Compute Resources in Container Environments (On-premise, Private & Public Cloud)

Date: 2020-04-14
Repo: vakarinc/Right-Sizing-Container-Environments
Description:
Best Practices for Managing Compute Resources in Container Environments (On-premise, Private & Public Cloud)

These are sample files used for creating containers with compute resources for learning in a lab environment 

##Mini-Lab – Right-Size Containers While Running (No Downtime)

### Create a directory or workspace to store files
mkdir /home/densify; cd /home/densify

### Enable kubectl command line completion
source <(kubectl completion bash)

To ensure our practice environment is isolated, create a new namespace called “densify-lab”.  Set “densify-lab” as your  as your current context to ensure all the commands run will only apply to this particular namespace.

### Create a namespace called “densify-lab”
kubectl create namespace densify-lab

### View current context info, note the default cluster and user name as we will use these to create a new context for our densify-lab namespace:
kubectl config current-context

### Create a new context called “densify” – Option 1
kubectl config set-context densify --namespace=densify-lab \
  --cluster=`kubectl config view -o jsonpath='{.contexts[*].context.cluster}'` \
  --user=`kubectl config view -o jsonpath='{.contexts[*].context.user}'`

### In case the above fails, you can using the following to specify the cluster and user – Option 2
kubectl config view

Refer to the “contexts/context” section in the output:
    cluster: <CLUSTER>
    user: <USERNAME>

### Create a new context called “densify”
kubectl config set-context densify –namespace=densify-lab --cluster=<CLUSTER> --user=<USER>

### Switch to the new context 
kubectl config use-context densify

### Validate you are in the “densify” context
kubectl config current-context
 
### Create a small Container Deployment of an application (2 copies of Wordpress).  Also, specify some resource requests and limits for both CPU and Memory.  See sample code below, cut and paste into a file using any editor: 
vi deployment.yaml

<--SNIP-->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wp
  template:
    metadata:
      labels:
        app: wp
    spec:
      containers:
      - name: wp
        image: wordpress:php7.1-apache
        resources:
          limits:
            cpu: "500m"
            memory: "128Mi"
          requests:
            cpu: "250m"
            memory: "64Mi"
<--SNIP-->
 
### Save the file and deploy the containers
kubectl apply -f deployment.yaml

### Confirm there are 2 wordpress pods running
kubectl get pods

### Validate the size of the containers, refer to the resource requests and limits section.  We will make changes and compare the results.
kubectl describe pod

### Modify the CPU and Memory settings for running containers
kubectl set resources deployment wordpress -c=wp \
  --limits=cpu=300m,memory=256Mi \
  --requests=cpu=200m,memory=192Mi

The changes will take effect automatically, new pods will be automatically deployed using the new resource requests and limits while the older ones using the incorrect settings will be terminated.

### Validate the new settings have been applied to the running containers
kubectl get pods
kubectl describe pod

### Other useful commands that could be used to apply the changes
kubectl edit deployment/wordpress -n densify
kubectl scale deployment/wordpress -n densify --replicas=2
kubectl autoscale deployments/wordpress -n densify --min=2 --max=5 --cpu-percent=80

