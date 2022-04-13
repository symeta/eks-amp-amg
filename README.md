# preparation & create eks cluster @gcr

# preparation

## Create an EC2 instance to be the client to create the EKS cluster

## install kubectl
```sh
sudo curl --silent --location -o /usr/local/bin/kubectl \
   https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```
## update aws cli
```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
## Install jq, envsubst (from GNU gettext utilities) and bash-completion
```sh
sudo yum -y install jq gettext bash-completion moreutils
```
## Install yq for yaml processing
```sh
echo 'yq() {
  docker run --rm -i -v "${PWD}":/workdir mikefarah/yq "$@"
}' | tee -a ~/.bashrc && source ~/.bashrc
```
## Verify the binaries are in the path and executable
```sh
for command in kubectl jq envsubst aws
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done
```
## Enable kubectl bash_completion
```sh
kubectl completion bash >>  ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```

## create an IAM role for the EC2 instance
<img width="1170" alt="Screen Shot 2022-03-30 at 4 51 16 PM" src="https://user-images.githubusercontent.com/97269758/160792010-0152073a-1a00-4f61-a328-0f5ab4897a69.png">
change the role of cloud9 ec2 instance:
<img width="592" alt="Screen Shot 2022-03-30 at 4 53 23 PM" src="https://user-images.githubusercontent.com/97269758/160792169-beb1dee7-594e-4fbb-9636-ad9e2639e9cf.png">

## configure aws cli with current region as default
```sh
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))
```
## Check if AWS_REGION is set to desired region
```sh
test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
```
## save these into bash_profile
```sh
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```

## Create a CMK for the EKS cluster to use when encrypting your Kubernetes secrets
```sh
aws kms create-alias --alias-name alias/<aws eks cluster name> --target-key-id $(aws kms create-key --query KeyMetadata.Arn --output text)
```
## retrieve the ARN of the CMK to input into the create cluster command.
```sh
export MASTER_ARN=$(aws kms describe-key --key-id alias/<aws eke cluster name> --query KeyMetadata.Arn --output text)
```
## save the MASTER_ARN environment variable into the bash_profile
```sh
echo "export MASTER_ARN=${MASTER_ARN}" | tee -a ~/.bash_profile
```
## install eksctl
```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin

mkdir -p $HOME/bin && cp ./eksctl $HOME/bin/eksctl && export PATH=$PATH:$HOME/bin

echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```

Confirm the eksctl command works

```sh
eksctl version
```

Enable eksctl bash-completion

```sh
eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
```
# create eks cluster

## create an eks cluster through aws cli
It's highly suggested that the eks cluster is created through aws cli rather than aws console, because there are quite a number of dependencies that makes the cluster creation through console a mission impossibile.
Create an eksctl deployment file (eksworkshop.yaml) use in creating your cluster using the following syntax:
```yaml
cat << EOF > eksworkshop.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: <aws eks cluster name>
  region: ${AWS_REGION}
  version: "1.19"

availabilityZones: ["${AZS[0]}", "${AZS[1]}", "${AZS[2]}"]

managedNodeGroups:
- name: nodegroup
  desiredCapacity: 3
  instanceType: t3.small
  ssh:
    enableSsm: true

# To enable all of the control plane logs, uncomment below:
# cloudWatch:
#  clusterLogging:
#    enableTypes: ["*"]

secretsEncryption:
  keyARN: ${MASTER_ARN}
EOF
```
create the eks cluster
```sh
eksctl create cluster -f eksworkshop.yaml
```
the creation process is shown as below. it takes around 15 minutes to finish the cluster creation

<img width="1155" alt="Screen Shot 2022-03-30 at 3 25 50 PM" src="https://user-images.githubusercontent.com/97269758/160796355-c0b7b17e-fa74-4f8c-ab04-8fb3fa4191ca.png">



