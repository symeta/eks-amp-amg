# configure amp
intall helm cmd tool on cloud9:
```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```


install helm on the eks cluster. the commands are as below:
```shell
# add prometheus Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# add grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts
```

install the prometheus server and create a namespace named prometheus:
```shell
kubectl create namespace prometheus

helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```
check the status of prometheus:
```shell
kubectl get all -n prometheus
```
In order to access the Prometheus server URL, we are going to use the kubectl port-forward command to access the application. In Cloud9, run:
```shell
kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
```
In your Cloud9 environment, click Tools / Preview / Preview Running Application. Scroll to the end of the URL and append:
```shell
/targets
```
In the web UI, you can see all the targets and metrics being monitored by Prometheus:

![Screen Shot 2022-03-29 at 9 26 20 AM](https://user-images.githubusercontent.com/97269758/160513992-bcb100e2-5908-47b7-aa83-6f8524c12da0.png)

use aws cli to create the managed prometheus workspace using the following command:
```shell
aws amp create-workspace --alias demo-prometheus --region us-west-2
```

The shell script shown below can be used to execute the following actions after substituting the placeholder variable YOUR_EKS_CLUSTER_NAME with the name of your Amazon EKS cluster.

* Creates an IAM role with an IAM policy that has permissions to remote-write into an AMP workspace
* Creates a Kubernetes service account that is annotated with the IAM role
* Creates a trust relationship between the IAM role and the OIDC provider hosted in your Amazon EKS cluster

```shell
##!/bin/bash
CLUSTER_NAME=YOUR_EKS_CLUSTER_NAME
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
OIDC_PROVIDER=$(aws eks describe-cluster --name $CLUSTER_NAME --query "cluster.identity.oidc.issuer" --output text | sed -e "s/^https:\/\///")

PROM_SERVICE_ACCOUNT_NAMESPACE=prometheus
GRAFANA_SERVICE_ACCOUNT_NAMESPACE=grafana
SERVICE_ACCOUNT_NAME=iamproxy-service-account
SERVICE_ACCOUNT_IAM_ROLE=EKS-AMP-ServiceAccount-Role
SERVICE_ACCOUNT_IAM_ROLE_DESCRIPTION="IAM role to be used by a K8s service account with write access to AMP"
SERVICE_ACCOUNT_IAM_POLICY=AWSManagedPrometheusWriteAccessPolicy
SERVICE_ACCOUNT_IAM_POLICY_ARN=arn:aws:iam::$AWS_ACCOUNT_ID:policy/$SERVICE_ACCOUNT_IAM_POLICY
#
# Setup a trust policy designed for a specific combination of K8s service account and namespace to sign in from a Kubernetes cluster which hosts the OIDC Idp.
# If the IAM role already exists, then add this new trust policy to the existing trust policy
#
echo "Creating a new trust policy"
read -r -d '' NEW_TRUST_RELATIONSHIP <<EOF
 [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::${AWS_ACCOUNT_ID}:oidc-provider/${OIDC_PROVIDER}"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "${OIDC_PROVIDER}:sub": "system:serviceaccount:${GRAFANA_SERVICE_ACCOUNT_NAMESPACE}:${SERVICE_ACCOUNT_NAME}"
        }
      }
    },
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::${AWS_ACCOUNT_ID}:oidc-provider/${OIDC_PROVIDER}"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "${OIDC_PROVIDER}:sub": "system:serviceaccount:${PROM_SERVICE_ACCOUNT_NAMESPACE}:${SERVICE_ACCOUNT_NAME}"
        }
      }
    }
  ]
EOF
#
# Get the old trust policy, if one exists, and append it to the new trust policy
#
OLD_TRUST_RELATIONSHIP=$(aws iam get-role --role-name $SERVICE_ACCOUNT_IAM_ROLE --query 'Role.AssumeRolePolicyDocument.Statement[]' --output json)
COMBINED_TRUST_RELATIONSHIP=$(echo $OLD_TRUST_RELATIONSHIP $NEW_TRUST_RELATIONSHIP | jq -s add)
echo "Appending to the existing trust policy"
read -r -d '' TRUST_POLICY <<EOF
{
  "Version": "2012-10-17",
  "Statement": ${COMBINED_TRUST_RELATIONSHIP}
}
EOF
echo "${TRUST_POLICY}" > TrustPolicy.json
#
# Setup the permission policy grants write permissions for all AWS StealFire workspaces
#
read -r -d '' PERMISSION_POLICY <<EOF
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "aps:RemoteWrite",
            "aps:QueryMetrics",
            "aps:GetSeries",
            "aps:GetLabels",
            "aps:GetMetricMetadata"
         ],
         "Resource":"*"
      }
   ]
}
EOF
echo "${PERMISSION_POLICY}" > PermissionPolicy.json

#
# Create an IAM permission policy to be associated with the role, if the policy does not already exist
#
SERVICE_ACCOUNT_IAM_POLICY_ID=$(aws iam get-policy --policy-arn $SERVICE_ACCOUNT_IAM_POLICY_ARN --query 'Policy.PolicyId' --output text)
if [ "$SERVICE_ACCOUNT_IAM_POLICY_ID" = "" ]; 
then
  echo "Creating a new permission policy $SERVICE_ACCOUNT_IAM_POLICY"
  aws iam create-policy --policy-name $SERVICE_ACCOUNT_IAM_POLICY --policy-document file://PermissionPolicy.json 
else
  echo "Permission policy $SERVICE_ACCOUNT_IAM_POLICY already exists"
fi

#
# If the IAM role already exists, then just update the trust policy.
# Otherwise create one using the trust policy and permission policy
#
SERVICE_ACCOUNT_IAM_ROLE_ARN=$(aws iam get-role --role-name $SERVICE_ACCOUNT_IAM_ROLE --query 'Role.Arn' --output text)
if [ "$SERVICE_ACCOUNT_IAM_ROLE_ARN" = "" ]; 
then
  echo "$SERVICE_ACCOUNT_IAM_ROLE role does not exist. Creating a new role with a trust and permission policy"
  #
  # Create an IAM role for Kubernetes service account 
  #
  SERVICE_ACCOUNT_IAM_ROLE_ARN=$(aws iam create-role \
  --role-name $SERVICE_ACCOUNT_IAM_ROLE \
  --assume-role-policy-document file://TrustPolicy.json \
  --description "$SERVICE_ACCOUNT_IAM_ROLE_DESCRIPTION" \
  --query "Role.Arn" --output text)
  #
  # Attach the trust and permission policies to the role
  #
  aws iam attach-role-policy --role-name $SERVICE_ACCOUNT_IAM_ROLE --policy-arn $SERVICE_ACCOUNT_IAM_POLICY_ARN  
else
  echo "$SERVICE_ACCOUNT_IAM_ROLE_ARN role already exists. Updating the trust policy"
  #
  # Update the IAM role for Kubernetes service account with a with the new trust policy
  #
  aws iam update-assume-role-policy --role-name $SERVICE_ACCOUNT_IAM_ROLE --policy-document file://TrustPolicy.json
fi
echo $SERVICE_ACCOUNT_IAM_ROLE_ARN

# EKS cluster hosts an OIDC provider with a public discovery endpoint.
# Associate this Idp with AWS IAM so that the latter can validate and accept the OIDC tokens issued by Kubernetes to service accounts.
# Doing this with eksctl is the easier and best approach.
#
eksctl utils associate-iam-oidc-provider --cluster $CLUSTER_NAME --approve

```

Create a file called amp_ingest_override_values.yaml with the following content in it.
```yaml
serviceAccounts:
  ## Disable alert manager roles
  ##
  server:
        name: "iamproxy-service-account"
  alertmanager:
    create: false

  ## Disable pushgateway
  ##
  pushgateway:
    create: false

server:
  remoteWrite:
    -
      queue_config:
        max_samples_per_send: 1000
        max_shards: 200
        capacity: 2500

  ## Use a statefulset instead of a deployment for resiliency
  ##
  statefulSet:
    enabled: true

  ## Store blocks locally for short time period only
  ##
  retention: 1h
  
## Disable alert manager
##
alertmanager:
  enabled: false

## Disable pushgateway
##
pushgateway:
  enabled: false

```

Execute the following command to install the Prometheus server configuration and configure the remoteWrite endpoint
```sh
export SERVICE_ACCOUNT_IAM_ROLE=EKS-AMP-ServiceAccount-Role
export SERVICE_ACCOUNT_IAM_ROLE_ARN=$(aws iam get-role --role-name $SERVICE_ACCOUNT_IAM_ROLE --query 'Role.Arn' --output text)
WORKSPACE_ID=$(aws amp list-workspaces --alias eks-workshop | jq .workspaces[0].workspaceId -r)
helm install prometheus-for-amp prometheus-community/prometheus -n prometheus -f ./amp_ingest_override_values.yaml \
--set serviceAccounts.server.annotations."eks\.amazonaws\.com/role-arn"="${SERVICE_ACCOUNT_IAM_ROLE_ARN}" \
--set server.remoteWrite[0].url="https://aps-workspaces.${AWS_REGION}.amazonaws.com/workspaces/${WORKSPACE_ID}/api/v1/remote_write" \
--set server.remoteWrite[0].sigv4.region=${AWS_REGION}

```
