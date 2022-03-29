# eks-amp-amg

## create an eks cluster through aws cli
create an eks cluster named demo-eks through aws cli leveraging on cloud9.
aws cli package has already been installed on cloud9 ec2 instance by default. can use
```
aws --version
```
to check the status of aws cli.

before creating the cluster, make sure that eksctl is installed on the cloud9 instance.
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
