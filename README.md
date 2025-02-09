# RollingUpgrades
> Reliable, extensible rolling-upgrades of Autoscaling groups in Kubernetes

RollingUpgrades provides a Kubernetes native mechanism for doing rolling-updates of instances in an AutoScaling group using a CRD and a controller.

## What does it do?

* RollingUpgrade is highly inspired by the way kops does rolling-updates.

* It provides similar options for the rolling-updates as kops and more.

* The RollingUpgrade Kubernetes custom resource has the following options in the spec:
  - `asgName`: Name of the autoscaling group to perform the rolling-update.
  - `region`: Name of the AWS region in which the ASG exists.
  - `preDrain.script`: The script to run before draining a node.
  - `postDrain.script`: The script to run after draining a node. This allows for performing actions such as quiescing network traffic, adding labels, etc.
  - `postDrain.waitSeconds`: The seconds to wait after a node is drained.
  - `postDrain.postWaitScript`: The script to run after the node is drained and the waitSeconds have passed. This can be used for ensuring that the drained pods actually were able to start elsewhere.
  - `nodeIntervalSeconds`: The amount of time in seconds to wait after each node in the ASG is terminated.
  - `postTerminate.script`: Optional bash script to execute after the node has terminated.

* After performing the rolling-update of the nodes in the ASG, RollingUpgrade puts the following data in the "Status" field.
  - `currentStatus`: Whether the rolling-update completed or errored out.
  - `startTime`: The RFC3339 timestamp when the rolling-update began. E.g. 2019-01-15T23:51:10Z
  - `endTime`: The RFC3339 timestamp when the rolling-update completed. E.g. 2019-01-15T00:35:10Z
  - `nodesProcessed`: The number of ec2 instances that were processed.

## Design

For each RollingUprade custom resource that is submitted, the following flowchart shows the sequence of actions taken to [perform the rolling-update](docs/RollingUpgradeDesign.png)

## Dependencies
- Kubernetes cluster on AWS with nodes in AutoscalingGroups. rolling-upgrades have been tested with Kubernetes clusters v1.12+.
- An IAM role with at least the policy specified below. The upgrade-manager should be run with that IAM role.

## Installing

### Complete step by step guide to create a cluster and run rolling-upgrades

For a complete, step by step guide for creating a cluster with kops, editing it and then running rolling-upgrades, please see [this](docs/step-by-step-example.md)

### Existing cluster in AWS

If you already have an existing cluster created using kops, follow the instructions below.

* Ensure that you have a Kubernetes cluster on AWS.
* Install the CRD using: `kubectl apply -f config/crd/bases`
* Install the controller using:
`kubectl create -f deploy/rolling-upgrade-controller-deploy.yaml`

* Note that the rolling-upgrade controller requires an IAM role with the following policy
```
{
    "Effect": "Allow",
    "Action": [
        "ec2:TerminateInstances",
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:TerminateInstanceInAutoScalingGroup"
    ],
    "Resource": [
        "*"
    ]
}
```
- If the rolling-upgrade controller is directly using the IAM role of the node it runs on, the above policy will have to be added to the IAM role of the node.
- If the rolling-upgrade controller is using it's own role created using KIAM, that role should have the above policy in it.

## For more details and FAQs, refer to [this](docs/faq.md)
