---
title: Real-time Scoring of Python Scikit-Learn and Deep Learning Models on Azure
description: This reference architecture shows how to deploy Python models as web services on Azure to make real-time predictions.
author: njray
ms.date: 11/06/2018
ms.author: njray
---

# Real-time Scoring of Python Scikit-Learn and Deep Learning Models on Azure

This reference architecture shows how to deploy Python models as web services to make real-time predictions. Two scenarios are covered: deploying regular Python models, and the specific requirements of deploying deep learning models. Both scenarios use the architecture shown. A reference implementation for this architecture is available on GitHub.


![](./_images/python-model-architecture.png)

**Scenario 1:** This scenario shows how to deploy a frequently asked questions (FAQ) matching model as a web service to provide predictions for user questions. For this scenario, “Input Data” in the architecture diagram refers to text strings containing the user questions to match with a list of FAQs. This scenario is designed for the [scikit-learn](scikit) machine learning library for Python but can be generalized to any scenario that uses Python models to make real-time predictions.

This scenario uses a subset of Stack Overflow question data which includes original questions tagged as JavaScript, their duplicate questions, and their answers. It trains a scikit-learn pipeline to predict the match probability of a duplicate question with each of the original questions. These predictions are made in real time using a REST API  endpoint.

The application flow for this architecture is as follows:

1.  The client sends a HTTP POST request with the encoded question data.

2.  The Flask app extracts the question from the request

3.  The question is then sent to the scikit-learn pipeline model for featurization and scoring.

4.  The matching FAQ questions with their scores are then piped into a JSON object and returned to the client.

An example app that consumes the results is included with the scenario. (See the following figure.)

![](./_images/faq-matches.png)

**Scenario 2:** This scenario shows how to deploy a Convolutional Neural Network (CNN) model as a web service to provide predictions on images. For this scenario, “Input Data” in the architecture diagram refers to image files. CNNs are proven to be very effective in computer vision for tasks such as image classification and object detection. This scenario is designed for the frameworks TensorFlow, Keras (with the TensorFlow back end), and PyTorch but can be generalized to any scenario that uses deep learning models to  make real-time predictions.

This scenario uses a pre-trained ResNet-152 model trained on ImageNet-1K (1,000 classes) dataset to predict which category (see figure below) an image belongs to. These predictions are made in real time using a REST API endpoint.

![](./_images/Example-predictions.png)

The application flow for the deep learning model is as follows:

1.  The client sends a HTTP POST request with the encoded image data.

2.  The Flask app extracts the image from the request.

3.  The image is then appropriately preprocessed and sent to the model for scoring.

4.  The scoring result is then piped into a JSON object and returned to the client.

## Architecture

This architecture consists of the following components.

### Compute

**[Virtual machine](vm)** is used as an example to show a device—local or in the cloud—that can send an HTTP request.

**[Azure Kubernetes Service](aks)** (AKS) is used to deploy the application on a Kubernetes cluster. AKS simplifies the deployment and operations of Kubernetes and supports deployment of Python models by providing clusters that can be configured using CPU-only VMs for regular Python models or GPU-enabled VMs for deep learning models.

### Other Components

**[Internal load balancer](internal-lb)** provisioned by AKS is used to expose the service externally. Traffic from the load balancer is directed to the back-end pods.

**[Docker Hub](docker)** is used to store the Docker image that is deployed on Kubernetes cluster. Docker Hub was chosen for this architecture because it is easy to use and is the default image repository for Docker users. Azure Container Registry can also be used for this architecture.

## Performance considerations

For real-time scoring architectures, performance in terms of throughput becomes a dominant consideration. For deployment of regular Python models, it is generally accepted that CPUs are sufficient to handle the workload. However for deep learning workloads, when the speed is a bottleneck, GPUs generally provide better [performance](gpus-vs-cpus)  compared to CPUs. For these cases in particular, to match GPU performance, a cluster with large number of CPUs is usually needed.

You can use CPUs for this reference architecture in either scenario, but for deep learning models, GPUs provide significantly higher throughput values compared to a CPU cluster of similar cost. AKS supports the use of GPUs, one the reasons it is used in this architecture instead of Azure App Service, Azure container instances, or another service.  Additionally, deep learning deployments typically use models with a high number of parameters. Using GPUs prevents contention for resources between the model and the web  service, an issue in the CPU-only deployments.

## Scalability considerations

For regular Python models, where AKS cluster is provisioned with CPU-only VMs, take care when [scaling out the number of pods](manually-scale-pods). The goal is to fully utilize the cluster. Scaling depends on the CPU requests and limits defined for the pods. Kubernetes also supports [autoscaling](autoscale-pods) of the pods to adjust the number of pods in a deployment depending on CPU utilization or other select metrics. In addition, a [cluster autoscaler](autoscaler) (in preview) can scale agent nodes based on pending pods.

For deep learning scenarios, when provisioning AKS clusters with GPU-enabled VMs, resource limits on pods are such that one GPU is assigned to only one pod. Depending on the type of VM used, you must [scale the nodes of the cluster](scale-cluster) to meet the demand for the service. You can do this easily using the Azure CLI and kubectl.

## Monitoring and logging considerations

### AKS monitoring

For visibility into AKS performance, use the [Azure Monitor for containers](monitor-containers) feature. It collects memory and processor metrics from controllers, nodes, and containers that are available in Kubernetes through the Metrics API.

While deploying your application, monitor the AKS cluster to make sure it is working as expected, all the nodes are operational, and all pods are running. Although you can use [kubectl](kubectl), the Kubernetes command-line client, to retrieve pod status, Kubernetes also includes a web dashboard for basic monitoring of the cluster status and management.

![](./_images/kubernetes-dashboard.png)

To see the overall state of the cluster and nodes, go to the **Nodes** section of the Kubernetes dashboard. If a node is inactive or has failed, you can display the error logs from that page. Similarly, go to the **Pods** and **Deployments** sections for information about the number of pods and status of your deployment.

### AKS logs 

AKS automatically logs all stdout/stderr to the logs of the pods in the cluster. Use kubectl to see these and also node-level events and logs. For details, see the deployment steps.

In addition, use [Azure Monitor for containers](monitor-containers) to collect metrics and logs through a containerized version of the Log Analytics agent for Linux, which is stored in your Log Analytics workspace.

## Security considerations

Use [Azure Security Center](security-center) to get a central view of the security state of your Azure resources. Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment, but does not monitor AKS agent nodes. Security Center is configured per Azure subscription. Enable security data collection as described in the [Azure Security Center quick start guide](get-started). When data collection is enabled, Security Center automatically scans any VMs created under that subscription.

### Operations

To log on to an AKS cluster using your Azure Active Directory authentication token, configure AKS to use Azure Active Directory for [user authentication](aad-auth). Cluster administrators can also configure Kubernetes role-based access control (RBAC) based on a user's identity or directory group membership.

In addition, use [RBAC](rbac) to control access to the Azure resources that you deploy. RBAC lets you assign authorization roles to members of your DevOps team. A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained [permissions](permissions).

### HTTPS 

As a security best practice, have your app enforce HTTPS and redirect HTTP requests. Create an HTTPS [ingress controller](ingress-controller) on AKS to add security to the information being passed to and from the application.

### Authentication

This solution does not restrict access to the endpoints. To deploy this solution in an enterprise setting, secure the endpoints through API keys and add some form of user authentication to the application.

### Docker Hub

This solution uses a public registry to store the Docker image. The code that the application depends on, and the model, are contained within this image. Enterprise applications should use a private registry such as [Azure Container Registry](acr) to help guard against running malicious code and to help keep the information inside the container from being compromised.

### DDoS protection

Consider enabling [DDoS Protection Standard](ddos). Although basic DDoS protection is enabled as part of the Azure platform, DDoS Protection Standard provides mitigation capabilities that are tuned specifically to Azure virtual network resources.

### Logging

Use best practices before storing log data, such as scrubbing user passwords and other information that could be used to commit security fraud.

## Deployment

To deploy this reference architecture, follow the steps described in the GitHub repo for: 
  - [Regular Python models](github-python).
  - [Deep learning models](github-dl).

[aad-auth]: /azure/aks/aad-integration
[acr]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/
[aks]: /azure/aks/intro-kubernetes
[autoscaler]: /azure/aks/autoscaler
[autoscale-pods]: /azure/aks/tutorial-kubernetes-scale#autoscale-pods
[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[ddos]: /azure/virtual-network/ddos-protection-overview
[docker]: https://hub.docker.com/
[get-started]: /azure/security-center/security-center-get-started
[github-python]: https://github.com/Azure/MLAKSDeployment
[github-dl]: https://github.com/Microsoft/AKSDeploymentTutorial
[gpus-vs-cpus]: https://azure.microsoft.com/en-us/blog/gpus-vs-cpus-for-deployment-of-deep-learning-models/
[ingress-controller]: /azure/aks/ingress-tls
[internal-lb]: /azure/aks/internal-lb
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[manually-scale-pods]: /azure/aks/tutorial-kubernetes-scale#manually-scale-pods
[monitor-containers]: /azure/monitoring/monitoring-container-insights-overview
[permisions]: /azure/aks/concepts-identity
[rbac]: /azure/active-directory/role-based-access-control-what-is
[scale-cluster]: /azure/aks/scale-cluster
[scikit]: https://pypi.org/project/scikit-learn/
[security-center]: /azure/security-center/security-center-intro
[vm]: /azure/virtual-machines/
