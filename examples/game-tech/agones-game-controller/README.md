# Amazon EKS Deployment with Agones Gaming Kubernetes Controller
This example shows how to deploy and run Gaming applications on Amazon EKS with Agones Kubernetes Controller

 - Deploy Private VPC, Subnets and all the required VPC endpoints
 - Deploy EKS Cluster with one managed node group in an VPC
 - Deploy Agones Kubernetes Controller using Helm Providers
 - Deploy a simple gaming server and test the application

# What is Agones
Agones is an Open source Kubernetes Controller with custom resource definitions and is used to create, run, manage and scale dedicated game server processes within Kubernetes clusters using standard Kubernetes tooling and APIs.
This model also allows any matchmaker to interact directly with Agones via the Kubernetes API to provision a dedicated game server

# What is GameLift
Amazon GameLift enables developers to deploy, operate, and scale dedicated, low-cost servers in the cloud for session-based, multiplayer games.
Built on AWS global computing infrastructure, GameLift helps deliver high-performance, high-reliability, low-cost game servers while dynamically scaling your resource usage to meet worldwide player demand.

## How to Deploy
### Prerequisites:
Ensure that you have installed the following tools in your Mac or Windows Laptop before start working with this module and run Terraform Plan and Apply
1. [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
3. [Kubectl](https://Kubernetes.io/docs/tasks/tools/)
4. [Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)

### Deployment Steps
#### Step1: Clone the repo using the command below

```shell script
git clone https://github.com/aws-ia/terraform-aws-eks-blueprints.git
```

#### Step2: Run Terraform INIT
Initialize a working directory with configuration files

```shell script
cd examples/game-tech/agones-game-controller
terraform init
```

#### Step3: Run Terraform PLAN
Verify the resources created by this execution

```shell script
export AWS_REGION=<ENTER YOUR REGION>   # Select your own region
terraform plan
```

#### Step4: Finally, Terraform APPLY
to create resources

```shell script
terraform apply
```

Enter `yes` to apply

### Configure `kubectl` and test cluster
EKS Cluster details can be extracted from terraform output or from AWS Console to get the name of cluster.
This following command used to update the `kubeconfig` in your local machine where you run kubectl commands to interact with your EKS Cluster.

#### Step5: Run `update-kubeconfig` command

`~/.kube/config` file gets updated with cluster details and certificate from the below command

    $ aws eks --region <enter-your-region> update-kubeconfig --name <cluster-name>

#### Step6: List all the worker nodes by running the command below

    $ kubectl get nodes

#### Step7: List all the pods running in `agones-system` namespace

    $ kubectl get pods -n agones-system

## Step8: Install K9s
This step is to install K9s client tool to interact with EKS Cluster

     curl -sS https://webinstall.dev/k9s | bash

Just type k9s after the installation and then you will see the output like this

         k9s

![Alt Text](https://github.com/aws-ia/terraform-aws-eks-blueprints/blob/9c6f8ea3e710f7b0137be07835653a2bf4f9fdfe/images/k9s-agones-cluster.png "K9s")

## Step9: Add EKS Workshop IAM role as EKS Cluster Administrator

    $ aws sts get-caller-identity

    ROLE="    - rolearn: arn:aws:iam::${ACCOUNT_ID}:role/TeamRole\n      username: admin\n      groups:\n        - system:masters"

    kubectl get -n kube-system configmap/aws-auth -o yaml | awk "/mapRoles: \|/{print;print \"$ROLE\";next}1" > /tmp/aws-auth-patch.yml

    kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"

## Step10: Deploying the Sample game server

    kubectl create -f https://raw.githubusercontent.com/googleforgames/agones/release-1.15.0/examples/simple-game-server/gameserver.yaml

    kubectl get gs

Output looks like below

    NAME                       STATE   ADDRESS         PORT   NODE                                        AGE
    simple-game-server-7r6jr   Ready   34.243.345.22   7902   ip-10-1-23-233.eu-west-1.compute.internal   11h

## Step11: Testing the Sample Game Server

    sudo yum install netcat

    nc -u <ADDRESS> <PORT>

    e.g.,  nc -u 34.243.345.22 7902

Output looks like below

    TeamRole:~/environment/eks-blueprints (main) $ echo -n "UDP test - Hello Workshop" | nc -u 34.243.345.22 7902
    Hello Workshop
    ACK: Hello Workshop
    EXIT
    ACK: EXIT

# Deploy GameLift FleetIQ

Amazon GameLift FleetIQ optimizes the use of low-cost Spot Instances for cloud-based game hosting with Amazon EC2. With GameLift FleetIQ, you can work directly with your hosting resources in Amazon EC2 and Auto Scaling while taking advantage of GameLift optimizations to deliver inexpensive, resilient game hosting for your players and makes the use of low-cost Spot Instances viable for game hosting

This [blog](https://aws.amazon.com/blogs/gametech/introducing-the-gamelift-fleetiq-adapter-for-agones/) will go through the details of deploying EKS Cluster using eksctl and deploy Agones with GameLift FleetIQ

Download the shell script and execute

    curl -O https://raw.githubusercontent.com/awslabs/fleetiq-adapter-for-agones/master/Agones_EKS_FleetIQ_Integration_Package%5BBETA%5D/quick_install/fleet_eks_agones_quickinstall.sh

## How to Destroy
The following command destroys the resources created by `terraform apply`

```shell script
cd examples/eks-cluster-with-new-vpc
terraform destroy --auto-approve
```

<!--- BEGIN_TF_DOCS --->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.0.1 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 3.66.0 |
| <a name="requirement_helm"></a> [helm](#requirement\_helm) | >= 2.4.1 |
| <a name="requirement_kubernetes"></a> [kubernetes](#requirement\_kubernetes) | >= 2.6.1 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 3.66.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_aws_vpc"></a> [aws\_vpc](#module\_aws\_vpc) | terraform-aws-modules/vpc/aws | v3.2.0 |
| <a name="module_eks-blueprints"></a> [eks-blueprints](#module\_eks-blueprints) | ../../.. | n/a |
| <a name="module_eks-blueprints-kubernetes-addons"></a> [eks-blueprints-kubernetes-addons](#module\_eks-blueprints-kubernetes-addons) | ../../..//modules/kubernetes-addons | n/a |

## Resources

| Name | Type |
|------|------|
| [aws_availability_zones.available](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/availability_zones) | data source |
| [aws_eks_cluster.cluster](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/eks_cluster) | data source |
| [aws_eks_cluster_auth.cluster](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/eks_cluster_auth) | data source |
| [aws_region.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_cluster_version"></a> [cluster\_version](#input\_cluster\_version) | Kubernetes Version | `string` | `"1.21"` | no |
| <a name="input_environment"></a> [environment](#input\_environment) | Environment area, e.g. prod or preprod | `string` | `"preprod"` | no |
| <a name="input_tenant"></a> [tenant](#input\_tenant) | Account Name or unique account unique id e.g., apps or management or aws007 | `string` | `"aws001"` | no |
| <a name="input_zone"></a> [zone](#input\_zone) | zone, e.g. dev or qa or load or ops etc... | `string` | `"dev"` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_configure_kubectl"></a> [configure\_kubectl](#output\_configure\_kubectl) | Configure kubectl: make sure you're logged in with the correct AWS profile and run the following command to update your kubeconfig |

<!--- END_TF_DOCS --->
