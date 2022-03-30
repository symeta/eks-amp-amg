# create eks cluster

## create an eks cluster through aws cli
Create an eksctl deployment file (eksworkshop.yaml) use in creating your cluster using the following syntax:
```yaml
cat << EOF > eksworkshop.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eksworkshop-eksctl
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
```shell
eksctl create cluster -f eksworkshop.yaml

the creation process is shown as below. it takes around 15 minutes to finish the cluster creation
<img width="1155" alt="Screen Shot 2022-03-30 at 3 25 50 PM" src="https://user-images.githubusercontent.com/97269758/160795667-fb426d10-3b86-4d08-a493-06092292c53d.png">
