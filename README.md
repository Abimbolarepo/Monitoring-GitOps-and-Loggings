# Monitoring-GitOps-and-Loggings

![image](https://user-images.githubusercontent.com/95041171/215177334-1e783061-d217-4afb-a8c3-b13fad40b949.png)

This Project would help us monitor the Kubernetes cluster using Prometheus and Grafana, deploy applications, services, and other Kubernetes objects using a GitOps tool known as Argo CD and lastly get logs from the application, services, and other Kubernetes objects running in the Kubernetes cluster with EFK stack.

Kindly see the steps taken to implement the project.

# Step1. Install Prometheus and Grafana.

# Define public Kubernetes chart repository in the Helm configuration
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
# Update local repositories
helm repo update
# Search for newly installed repositories
helm repo list
# Create a namespace for Prometheus and Grafana resources
kubectl create ns prometheus
# Install Prometheus using HELM
helm install prometheus prometheus-community/kube-prometheus-stack -n prometheus
# Check all resources in Prometheus Namespace
kubectl get all -n prometheus

![image](https://user-images.githubusercontent.com/95041171/215171435-4d8589e0-d95c-449d-bbb3-82bbe5a3ed00.png)

Prometheus uses Prometheus Query Language(PQL) to query Kubernetes metrics which is not easy to visualise. Search for API Server requests in Prometheus Dashboard.

Access the Prometheus service using prt forwarding

# Port forward the Prometheus service
kubectl port-forward -n prometheus prometheus-prometheus-kube-prometheus-prometheus-0 9090

Open localhost:9090 in browser.

![image](https://user-images.githubusercontent.com/95041171/215172133-2c4ca9bf-4dc1-4d39-ac37-bd1e04f89c9c.png)

In order to login to the Grafana dashboard, username and password are required which can be retrieved by using the below command:

# Port forward the Prometheus service
kubectl port-forward -n prometheus prometheus-prometheus-kube-prometheus-prometheus-0 9090
# Get the Username
kubectl get secret -n prometheus prometheus-grafana -o=jsonpath='{.data.admin-user}' |base64 -d
# Get the Password
kubectl get secret -n prometheus prometheus-grafana -o=jsonpath='{.data.admin-password}' |base64 -d
# Port forward the Grafana service
kubectl port-forward -n prometheus prometheus-grafana-5449b6ccc9-b4dv4 3000

Open localhost:3000 in browser and login using the username and password.

![image](https://user-images.githubusercontent.com/95041171/215172769-515c2aeb-2c19-4280-8ef4-d5d00c038388.png)

Prometheus and Grafana have been deployed to the Kubernetes cluster and they can be accessed from the browser.

# Step2. Install Argo CD and deploy four (4) applications using Argo CD.

kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

This will create a new namespace, argocd, where Argo CD services and application resources will live.

![image](https://user-images.githubusercontent.com/95041171/215175031-93f6a0d5-8097-4cd9-9e54-4362f7d998d4.png)

# Service Type Load Balancer

Change the argocd-server service type to LoadBalancer:

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Port Forwarding

Kubectl port-forwarding can also be used to connect to the API server without exposing the service.

kubectl port-forward svc/argocd-server -n argocd 8080:443

The Argo CD was accessed and the aaplications were successfully deployed.

![image](https://user-images.githubusercontent.com/95041171/215175120-a2134b5e-b125-4210-ae5b-0311ac823c26.png)

![image](https://user-images.githubusercontent.com/95041171/215175188-233c44e4-adf4-4984-bafc-7e105f47ef06.png)

The applications are up and running, Grafana would be used to monitor the Kubernetes cluster and the metrics will be visualized.

![image](https://user-images.githubusercontent.com/95041171/215176197-2d7bbc25-76eb-4210-94fd-18df92b5ccfa.png)

![image](https://user-images.githubusercontent.com/95041171/215176245-85a642a7-4e1d-4ce0-bcc1-5a19b8a03af9.png)

# Step3. Install EFK
When running multiple services and applications on a Kubernetes cluster, a centralized, cluster-level logging stack can help you quickly sort through and analyze the heavy volume of log data produced by your Pods. One popular centralized logging solution is the Elasticsearch, Fluentd, and Kibana (EFK) stack.

# Create a Namespace
vi kube-logging.yaml

Inside your editor, paste the following Namespace object YAML:

![image](https://user-images.githubusercontent.com/95041171/215179587-a63305cd-39df-4188-b68d-b2603ba88b1f.png)
  
kubectl create -f kube-logging.yaml

# We can now deploy an Elasticsearch cluster into this isolated logging Namespace.

# Create the Elasticsearch StatefulSet
Create the Headless Service

vi elasticsearch_svc.yaml

![image](https://user-images.githubusercontent.com/95041171/215179857-a78814ee-a2c6-42d1-9aa0-90ad2c43e996.png)

kubectl create -f elasticsearch_svc.yaml

# Creating the StatefulSet

vi elasticsearch_statefulset.yaml

![image](https://user-images.githubusercontent.com/95041171/215180190-4d15d178-96ed-49ac-84a1-6f69fa461230.png)

![image](https://user-images.githubusercontent.com/95041171/215180351-e7fbc3f2-759a-498e-8779-804720b5a6a6.png)

kubectl create -f elasticsearch_statefulset.yaml

# create a storage class for the statefulset

vi storageclass.yaml

![image](https://user-images.githubusercontent.com/95041171/215181431-2e8133ad-2008-44c4-8e5c-99a6e869d531.png)

kubectl create -f storageclass.yaml

# Create the Kibana Deployment and Service.

vi kibana.yaml

![image](https://user-images.githubusercontent.com/95041171/215184169-d65ffcc9-e6ce-43d2-8c60-8ddf00257160.png)

kubectl create -f kibana.yaml

# To access the Kibana interface, we’ll once again forward a local port to the Kubernetes node running Kibana. Grab the Kibana Pod details using kubectl get:

kubectl get pods --namespace=kube-logging

# Here we observe that our Kibana Pod is called kibana-6c9fb4b5b7-plbg2.

Forward the local port 5601 to port 5601 on this Pod:

kubectl port-forward kibana-6c9fb4b5b7-plbg2 5601:5601 --namespace=kube-logging

# Now, in your web browser, visit the following URL:

http://localhost:5601

![image](https://user-images.githubusercontent.com/95041171/215184538-7873f296-24f0-4060-a41c-8c8a5e66f056.png)

# Createthe Fluentd DaemonSet

vi fluentd.yaml

![image](https://user-images.githubusercontent.com/95041171/215185423-18e20b08-ca21-49b3-a742-894e7ca727ef.png)

![image](https://user-images.githubusercontent.com/95041171/215185561-7789bfb8-7788-4918-9c41-ed4e7f30a153.png)

kubectl create -f fluentd.yaml

# Verify that your DaemonSet rolled out successfully using kubectl:

kubectl get ds --namespace=kube-logging

This indicates that there are some numbers of fluentd Pods running, which corresponds to the number of nodes in our Kubernetes cluster.

We can now check Kibana to verify that log data is being properly collected and shipped to Elasticsearch.

With the kubectl port-forward still open, navigate to http://localhost:5601.

Click on Discover in the left-hand navigation menu:

![image](https://user-images.githubusercontent.com/95041171/215186163-7d460729-4c26-4739-890d-47e927c2ca74.png)

You should see the following configuration window:

![image](https://user-images.githubusercontent.com/95041171/215186264-f54358c2-a802-4a05-8b17-19a45ce7c17c.png)

This allows you to define the Elasticsearch indices you’d like to explore in Kibana. To learn more, consult Defining your index patterns in the official Kibana docs. For now, we’ll just use the logstash-* wildcard pattern to capture all the log data in our Elasticsearch cluster. Enter logstash-* in the text box and click on Next step.

https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html









