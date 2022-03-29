# eks-amp-amg

## create an eks cluster through aws cli
create an eks cluster named demo-eks through aws cli leveraging on cloud9.
aws cli package has already been installed on cloud9 ec2 instance by default. can use
```
aws --version
```
to check the status of aws cli.

before creating the cluster, make sure that kubectl & eksctl is installed on the cloud9 instance.
execute the following command to install the kubectl tool.
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
```
execute the following command to install the eksctl tool.
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```
```
sudo mv /tmp/eksctl /usr/local/bin
```
```
eksctl version
```
after the eksctl tool has been successfully installed, execute the following command to create the eks cluster
```
eksctl create cluster \
    --name demo-eks \
    --version 1.21 \
    --without-nodegroup
```
after the eks cluster has been installed, manually add a nodegroup consisting at least one compute node from aws eks console. the compute node is used for installing prometheus server, and also functions as the souce of all the monitoring metrics data.

![Screen Shot 2022-03-29 at 9 13 19 AM](https://user-images.githubusercontent.com/97269758/160512727-40bb30fe-c2a1-4e9a-b487-1336ce05b261.png)


install helm on the eks cluster. the commands are as below:
```
# add prometheus Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# add grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts

```

install the prometheus server and create a namespace named prometheus:
```
kubectl create namespace prometheus

helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

check the status of prometheus:
```
kubectl get all -n prometheus
```
In order to access the Prometheus server URL, we are going to use the kubectl port-forward command to access the application. In Cloud9, run:
```
kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
```
In your Cloud9 environment, click Tools / Preview / Preview Running Application. Scroll to the end of the URL and append:
```
/targets
```
In the web UI, you can see all the targets and metrics being monitored by Prometheus:

![Screen Shot 2022-03-29 at 9 26 20 AM](https://user-images.githubusercontent.com/97269758/160513992-bcb100e2-5908-47b7-aa83-6f8524c12da0.png)

use aws cli to create the managed prometheus workspace using the following command:
```
aws amp create-workspace --alias demo-prometheus --region us-west-2
```

The shell script shown below can be used to execute the following actions after substituting the placeholder variable YOUR_EKS_CLUSTER_NAME with the name of your Amazon EKS cluster.

-Creates an IAM role with an IAM policy that has permissions to remote-write into an AMP workspace
-Creates a Kubernetes service account that is annotated with the IAM role
-Creates a trust relationship between the IAM role and the OIDC provider hosted in your Amazon EKS cluste
