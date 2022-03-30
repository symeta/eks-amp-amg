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
