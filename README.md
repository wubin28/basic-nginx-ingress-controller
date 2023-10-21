## create a basic NGINX ingress controller without customizing the defaults

```
# [Install kubectl on ubuntu](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management)
docker --version
kubectl version -o yaml
kubectl get nodes

# install azure-cli on macOS
brew update
brew install azure-cli
# [install azure-cli on ubuntu](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

az --version
az login

export NAMESPACE=basic-nginx-ingress-controller

kubectl create namespace $NAMESPACE

helm repo add ingress-nginx-repo https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx-release ingress-nginx-repo/ingress-nginx \
  -n $NAMESPACE \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz
# If you encounter any errors when running the command above
kc describe ingressclass nginx
kc delete ingressclass nginx
kubectl describe clusterrole ingress-nginx-release
kubectl delete clusterrole ingress-nginx-release
kubectl describe clusterrolebinding ingress-nginx-release
kubectl delete clusterrolebinding ingress-nginx-release
kubectl describe validatingwebhookconfiguration ingress-nginx-release-admission
kubectl delete validatingwebhookconfiguration ingress-nginx-release-admission

# watch the status 
helm list -n $NAMESPACE
# check the ip address of ingress nginx controller
kubectl get services -o wide -n $NAMESPACE
kubectl get pods -o wide -n $NAMESPACE

# [Run demo applications](https://learn.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli)
# [Code](https://github.com/wubin28/basic-nginx-ingress-controller)
# In this example, you use kubectl apply to deploy two instances of a simple Hello world application.
# Both applications are now running on your Kubernetes cluster. To route traffic to each application, create a Kubernetes ingress resource. The ingress resource configures the rules that route traffic to one of the two applications.
kubectl apply -f deployment-hello-world-one.yml -n $NAMESPACE
kubectl apply -f service-hello-world-one.yml -n $NAMESPACE
kubectl apply -f deployment-hello-world-two.yml -n $NAMESPACE
kubectl apply -f service-hello-world-two.yml -n $NAMESPACE

# In the following example, traffic to EXTERNAL_IP/hello-world-one is routed to the service named aks-helloworld-one. Traffic to EXTERNAL_IP/hello-world-two is routed to the aks-helloworld-two service. Traffic to EXTERNAL_IP/static is routed to the service named aks-helloworld-one for static assets.
kubectl apply -f ingress-hello-world.yml -n $NAMESPACE
# check the ingress
kubectl get ingresses -n $NAMESPACE
kubectl describe ingress hello-world-ingress -n $NAMESPACE
# delete the ingress
kubectl delete ingress hello-world-ingress -n $NAMESPACE

# To test the routes for the ingress controller, browse to the two applications. Open a web browser to the IP address of your NGINX ingress controller, such as EXTERNAL_IP. The first demo application is displayed in the web browser, as shown in the following example. Now add the /hello-world-two path to the IP address, such as EXTERNAL_IP/hello-world-two. The second demo application with the custom title is displayed.
kubectl get services -o wide -n $NAMESPACE
http://<EXTERNAL-IP>/
http://<EXTERNAL-IP>/hello-world-one
http://<EXTERNAL-IP>/hello-world-two

# clean up resources
# Delete the sample namespace and all resources
kubectl delete namespace $NAMESPACE
# Delete resources individually - Alternatively, a more granular approach is to delete the individual resources created.
# List the Helm releases with the helm list command.
helm list -n $NAMESPACE
# Uninstall the releases with the helm uninstall command
helm uninstall ingress-nginx -n $NAMESPACE
# Remove the two sample applications.
kubectl delete -f deployment-hello-world-one.yaml -n $NAMESPACE
kubectl delete -f service-hello-world-one.yaml -n $NAMESPACE
kubectl delete -f deployment-hello-world-two.yaml -n $NAMESPACE
kubectl delete -f service-hello-world-two.yaml -n $NAMESPACE
# Remove the ingress route that directed traffic to the sample apps.
kubectl delete -f ingress-hello-world.yaml
# Delete the namespace using the kubectl delete command and specifying your namespace name.
kubectl delete namespace $NAMESPACE
```
