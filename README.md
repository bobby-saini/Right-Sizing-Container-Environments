# Best Practices for Managing Compute Resources in Container Environments (On-premise, Private & Public Cloud)

This mini-lab includes instructions and files to show how a Container can be deployed and how its compute resoures can be updated while the application is running.

It creates a new namespace and deploys 2 copies of the Wordpress application using CPU and Memory requests and limits.  The goal of the lab is to show how these allocation settings can be updated while the application is running to minimize downtime. Generating the right-sizing recommendations is very complex and requires analytics software [Densify](http://www.Densify.com) to analyze Container configuration and historical utilization data using business policies.    

```` javascript
# Create a directory or workspace to store files
mkdir /home/densify; cd /home/densify

# Enable kubectl command line completion
source <(kubectl completion bash)
````

To ensure our practice environment is isolated, create a new namespace called “densify-lab”.  Set “densify-lab” as your  as your current context to ensure all the commands run will only apply to this particular namespace.

```` javascript
# Create a namespace called “densify-lab”
kubectl create namespace densify-lab

# View current context info, note the default cluster and user name as we will use these to create a new context for our densify-lab namespace:
kubectl config current-context

# Create a new context called “densify” – Option 1
kubectl config set-context densify --namespace=densify-lab \
  --cluster=`kubectl config view -o jsonpath='{.contexts[*].context.cluster}'` \
  --user=`kubectl config view -o jsonpath='{.contexts[*].context.user}'`

````

In case the above fails, you can using the following to specify the cluster and user – Option 2
kubectl config view

Refer to the “contexts/context” section in the output:
    cluster: <CLUSTER>
    user: <USERNAME>

Create a new context called “densify”
kubectl config set-context densify –namespace=densify-lab --cluster=<CLUSTER> --user=<USER>

```` javascript
# Switch to the new context 
kubectl config use-context densify

# Validate you are in the “densify” context
kubectl config current-context

````

Create a small Container Deployment of an application (2 copies of Wordpress).  Also, specify some resource requests and limits for both CPU and Memory. Refer to the deployment.yaml file or see sample code below, cut and paste into a file using any editor: 

```` javascript
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
````

```` javascript
# Save the file and deploy the containers
kubectl apply -f deployment.yaml

# Confirm there are 2 wordpress pods running
kubectl get pods

# Validate the size of the containers, refer to the resource requests and limits section.  We will make changes and compare the results.
kubectl describe pod

# Modify the CPU and Memory settings for running containers
kubectl set resources deployment wordpress -c=wp --limits=cpu=300m,memory=256Mi --requests=cpu=200m,memory=192Mi
````

The changes will take effect automatically, new pods will be automatically deployed using the new resource requests and limits while the older ones using the incorrect settings will be terminated.

```` javascript
# Validate the new settings have been applied to the running containers
kubectl get pods
kubectl describe pod

# Other useful commands that could be used to apply the changes
kubectl edit deployment/wordpress
kubectl scale deployment/wordpress --replicas=3
kubectl autoscale deployments/wordpress -n densify --min=3 --max=4 --cpu-percent=75
````
