# Final project for IBA devops course 


## CI/CD automation workflow using GitHub Actions, ArgoCD, and Helm charts deployed on K8s cluster

Project task inculde:
Create an EKS cluster with two t3.medium nodes. Deploy any web application to Kubernetes and connect monitoring using Grafana, Prometheus. 
The web application must be accessible via a browser. The application must be deployed via a CI/CD tool. 
Recommended tool for CD in Kubernetes is ArgoCD.

## Implementation steps

 - Prepare AWS EKS using Terraform
 - GitHub actions setting CI part
 - Install ArgoCD in EKS for continious deployment
 - Install Prometheus & Grafana for monitoring

![screenshot](./pics/flow.png)

#### EKS rollout using Terraform

Let's use hashicorp EKS cluster (AWS) example:

```bash
git clone https://github.com/hashicorp-education/learn-terraform-provision-eks-cluster

cd learn-terraform-provision-eks-cluster
```

We will modify vpc name and set only 2 private and public subnets for EKS cluster in main.tf

Only one node group will be used with 2 desired t3.medium instances.


![screenshot](./pics/maintf-1.png)

![screenshot](./pics/maintf-2.png)

Region also will be changed in variables.tf 

![screenshot](./pics/vartf.png)


Initialize Terraform workspace and provision rollout of EKS:

```bash
sudo terraform init
sudo terraform apply -auto-approve
```

Configure kubectl to work with EKS:

```bash
sudo aws eks --region us-east-1 update-kubeconfig     --name ivanf-eks-training
```

## GitHub actions. OIDC and Telegram message preparation


OpenID Connect (OIDC) Identity Provider for GitHub Actions, which help to configure workflows that request temporary, on-demand credentials from any service provider on the internet that supports OIDC authentication.

So using OpenID Connect it's possible to remove the need to have keys stored in GitHub Actions.

Steps:

 - Configure AWS Identity and Access Management (IAM) in our AWS account to believe what the GitHub Actions Identity Provider says.
 - Make an IAM role available to GitHub Actions entities with specific properties.
 - Add an Actions workflow to request and use credentials from AWS.
 
![screenshot](./pics/OIDC.png)

[main.tf.yml](https://github.com/voyager1122/IBA-DEVOPS-PROJECT/blob/main/terraform-openid/main.tf)


We have to configure AWS account ID and github repo which we is planned to have permit access to AWS ECR

![screenshot](./pics/oidc-1.png)

![screenshot](./pics/oidc-2.png)

```bash
sudo terraform init
sudo terraform apply -auto-approve
```

![screenshot](./pics/oidc-aws-iam.png)

![screenshot](./pics/oidc-aws-ecr.png)



To send notification messages to Telegram we need to prepare two secret variable to be used by actions

![screenshot](./pics/tgsecrets.png)

Please follow this guide how to get Telegram token and chatid:

[telegram messaging in github actions](https://github.com/appleboy/telegram-action)


## GitHub actions. CI implementation

[CI.yml](https://github.com/voyager1122/IBA-DEVOPS-PROJECT/blob/main/.github/workflows/main.yml)


Steps:
 - Checkout report
 - Dockerfile linter
 - Helm chart linter 
 - Connect to AWS EKS
 - Build docker image
 - Update helm chart repo to be detedcted and fetched by ArgoCD later 
 - Send notification message to Telegram
 
![screenshot](./pics/tg-message.png)
 
## ArgoCD - continious deployment

ArgoCD installed using Lens which was connected to EKS

![screenshot](./pics/lens-1.png)
 
![screenshot](./pics/lens-2.png)

![screenshot](./pics/lens-3.png)


Next step is to connect github as repo and add APP 

![screenshot](./pics/argo-connect-repo.png)

![screenshot](./pics/argo-app-svc.png)

![screenshot](./pics/argo-app-svc.png)

![screenshot](./pics/argo-app-status2.png)

![screenshot](./pics/argo-app-status.png)

We can check app's url in service:

![screenshot](./pics/argo-app-svc-url.png)

![screenshot](./pics/lens-app-svc.png)


The result is running web app in K8s cluster

![screenshot](./pics/web-app.png)

#### Monitoring. Installation of Prometheus and Grafana

Prometheus and Grafana were installed by manifests.

[prometheus.yaml](https://github.com/voyager1122/IBA-DEVOPS-PROJECT/blob/main/prometheus.yaml)

[grafana.yaml](https://github.com/voyager1122/IBA-DEVOPS-PROJECT/blob/main/grafana.yaml)

```yaml
sudo kubectl apply -f prometheus.yaml
sudo kubectl apply -f grafana.yaml
```

![screenshot](./pics/argo-prometheus-grafana.png)

Dashboard were imported manually using dashboard ID

https://github.com/dotdc/grafana-dashboards-kubernetes


![screenshot](./pics/argo-prometheus-grafana.png)



 