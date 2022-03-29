# create eks cluster

## create an eks cluster through aws cli
create an eks cluster named demo-eks through aws cli leveraging on cloud9.
aws cli package has already been installed on cloud9 ec2 instance by default. can use
```shell
aws --version
```
to check the status of aws cli.

before creating the cluster, make sure that kubectl & eksctl is installed on the cloud9 instance.
execute the following command to install the kubectl tool.
```shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
```
execute the following command to install the eksctl tool.
```shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```
```
sudo mv /tmp/eksctl /usr/local/bin
```
```shell
eksctl version
```
after the eksctl tool has been successfully installed, execute the following command to create the eks cluster
```shell
eksctl create cluster \
    --name demo-eks \
    --version 1.21 \
    --without-nodegroup
```
after the eks cluster has been installed, manually add a nodegroup consisting at least one compute node from aws eks console. the compute node is used for installing prometheus server, and also functions as the souce of all the monitoring metrics data.

![Screen Shot 2022-03-29 at 9 13 19 AM](https://user-images.githubusercontent.com/97269758/160512727-40bb30fe-c2a1-4e9a-b487-1336ce05b261.png)
