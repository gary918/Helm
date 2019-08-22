# Use Helm to Deploy Applications to AKS
A guide line of deploying a simple web application (jmalloc/echo-server) into AKS cluster by using Helm.
<img src="image/helm_arch.jpg" alt="">
## Environment Preparation
### Install Docker on Windows 10
- Refer to [Install Docker Desktop on Windows](https://docs.docker.com/docker-for-windows/install/)
- Remember to switch on Hyper-V on Windows 10
    - Use 'Turn Windows Features on or off'  
- Remember to enable virtualization in your laptop's BIOS setting.
### Install Azure CLI
### Create AKS Cluster on Azure
- Enable RBAC
## Helm Installation and Configuration on Windows 10
- Refer to [Installing Helm](https://helm.sh/docs/using_helm/)
    - `choco install kubernetes-helm`
    - `helm version`
## Write a Simple Helm Chart
`C:\Users\ganwa\git\helm>mkdir charts`
`C:\Users\ganwa\git\helm>cd charts`
Create your own helm chart.
`C:\Users\ganwa\git\helm\charts> helm create samplechart
Creating samplechart`
Edit values.yaml
Edit deployment.yaml
Verify dependencies and templates.
```
C:\Users\ganwa\git\helm\charts> helm lint samplechart
==> Linting samplechart
[INFO] Chart.yaml: icon is recommended
```
Package helm chart (add --debug to get more information)
```
C:\Users\ganwa\git\helm\charts> helm package samplechart
Successfully packaged chart and saved it to: C:\Users\ganwa\git\helm\charts\samplechart-0.1.0.tgz
```
Check if this application has been successfuly published to the repository.
```
C:\Users\ganwa\git\helm\charts> helm search samplechart
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
local/samplechart       0.1.1           1.0             A Helm chart for Kubernetes
```
If you cannot get the application, it is probably because your chart has not been managed by Helm. Then you need to start a local Repository server.
## Deploy the Applicatin to AKS Cluster
Log in Azure if needed.
Get the credentials for connecting AKS cluster.
`az aks get-credentials --resource-group <ResourceGroupName> --name <ClusterName>`
Check if you've connected to the AKS cluster.
`kubectl get pods`
Create a service account and role binding for the Tiller service.
`kubectl apply -f helm-rbac.yaml`
Configure Helm (only work for cmd)
`helm init --service-account tiller --node-selectors "beta.kubernetes.io/os"="linux"`
Install Helm chart
`helm install --dry-run --debug local/samplechart`
`helm install local/samplechart --name sampleapp`
Check the installation
`kubectl get services`
Watch the service until it gets a external IP. It usually takes about 2 mins.
`kubectl get service sampleapp-samplechart --watch`
Access the URL
`curl http://<external-ip>:port`
Or type the following URL in your browser
`http://<external-ip>:port/.ws`
The application will echo your input.

## Other Helm Operations
To upgrade your application, first change the version in Chart.yaml. Then re-package the chart.
`helm package samplechart`
The upgrade command will automatically get the latest version of your application. You can also use `--version:xxx` to define which version you want to move to.
`helm upgrade sampleapp local/samplechart`
Delete your application
`helm delete sampleapp`

## References
- [Helm Introduction](https://www.hi-linux.com/posts/21466.html)
- [Install applications with Helm in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm)