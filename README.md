# Getting Started


# Install Istio
####For this installation, we use the demo configuration profile. It’s selected to have a good set of defaults for testing, but there are other profiles for production or performance testing.

If your platform has a vendor-specific configuration profile, e.g., Openshift, use it in the following command, instead of the demo profile. Refer to your platform instructions for details.
````
- $ istioctl install --set profile=demo -y
- ✔ Istio core installed
- ✔ Istiod installed
- ✔ Egress gateways installed
- ✔ Ingress gateways installed
- ✔ Installation complete
````
Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:
````
$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
````

# Deploy the sample application
Deploy the Bookinfo sample application:
````
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
````
Verify everything is working correctly up to this point. Run this command to see if the app is running inside the cluster and serving HTML pages by checking for the page title in the response:
````
$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
````
# Open the application to outside traffic
The Bookinfo application is deployed but not accessible from the outside. To make it accessible, you need to create an Istio Ingress Gateway, which maps a path to a route at the edge of your mesh.
Associate this application with the Istio gateway:
````
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
````
Ensure that there are no issues with the configuration:
````
$ istioctl analyze
✔ No validation issues found when analyzing namespace: default.
````
# Determining the ingress IP and ports
Follow these instructions to set the INGRESS_HOST and INGRESS_PORT variables for accessing the gateway. Use the tabs to choose the instructions for your chosen platform:

Execute the following command to determine if your Kubernetes cluster is running in an environment that supports external load balancers:
````
$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   172.21.109.129   130.211.10.121  80:31380/TCP,443:31390/TCP,31400:31400/TCP   17h
````
If the EXTERNAL-IP value is set, your environment has an external load balancer that you can use for the ingress gateway. If the EXTERNAL-IP value is <none> (or perpetually <pending>), your environment does not provide an external load balancer for the ingress gateway. In this case, you can access the gateway using the service’s node port.

Choose the instructions corresponding to your environment:

Follow these instructions if you have determined that your environment has an external load balancer.

Set the ingress IP and ports:
````
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
````

setup while loop to make calls to the service so we can monitor traffic
````
While ($true) { curl -UseBasicParsing http://127.0.0.1/productpage; Start-Sleep -Seconds 1;}
````

Choose a dashboard and show
````
istioctl dashboard grafana
````
## Reference Documentation
For further reference, please consider the following sections:
* [Official Istio documentation](https://istio.io/latest/docs/setup/getting-started/)