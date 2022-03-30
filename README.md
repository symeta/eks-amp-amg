# preparation

## create a cloud9 instance to be the command lien concole for precedures afterwards

<img width="1004" alt="Screen Shot 2022-03-30 at 4 45 35 PM" src="https://user-images.githubusercontent.com/97269758/160790596-3a355280-31af-47d8-bdd9-414127facbcd.png">

## install kubectl
```sh
sudo curl --silent --location -o /usr/local/bin/kubectl \
   https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl
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
## set the AWS Load Balancer Controller version
```sh
echo 'export LBC_VERSION="v2.3.0"' >>  ~/.bash_profile
.  ~/.bash_profile
```
## create an IAM role for the cloud9 workspace
<img width="1170" alt="Screen Shot 2022-03-30 at 4 51 16 PM" src="https://user-images.githubusercontent.com/97269758/160792010-0152073a-1a00-4f61-a328-0f5ab4897a69.png">
change the role of cloud9 ec2 instance:
<img width="592" alt="Screen Shot 2022-03-30 at 4 53 23 PM" src="https://user-images.githubusercontent.com/97269758/160792169-beb1dee7-594e-4fbb-9636-ad9e2639e9cf.png">

## To ensure temporary credentials aren’t already in place we will remove any existing credentials file as well as disabling AWS managed temporary credentials:
```sh
aws cloud9 update-environment  --environment-id $C9_PID --managed-credentials-action DISABLE
rm -vf ${HOME}/.aws/credentials
```
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
## Use the GetCallerIdentity CLI command to validate that the Cloud9 IDE is using the correct IAM role
```sh
aws sts get-caller-identity --query Arn | grep eksworkshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
```

