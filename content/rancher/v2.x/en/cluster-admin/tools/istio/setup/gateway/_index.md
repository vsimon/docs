---
title: 4. Set up the Istio Gateway
weight: 4
---

The gateway to each cluster can have its own port or load balancer, which is unrelated to a service mesh. By default, each Rancher-provisioned cluster has one NGINX ingress controller allowing traffic into the cluster. 

You can use the NGINX ingress controller with or without Istio installed. If this is the only gateway to your cluster, Istio will be able to route traffic from service to service, but Istio will not be able to receive traffic from outside the cluster.

To allow Istio to receive external traffic, you need to enable Istio's gateway, which works as a north-south proxy for external traffic.  When you enable the Istio gateway, the result is that your cluster will have two ingresses.

You will also need to set up a Kubernetes gateway for your services. This Kubernetes resource points to Istio's implementation of the ingress gateway to the cluster.

For more information on the Istio gateway, refer to the [Istio documentation.](https://istio.io/docs/reference/config/networking/v1alpha3/gateway/) 

![In an Istio-enabled cluster, you can have two ingresses: the default Nginx ingress, and the default Istio controller.]({{<baseurl>}}/img/rancher/istio-ingress.svg)

# Enable the Istio Gateway

The ingress gateway is a Kubernetes service that will be deployed in your cluster. There is only one Istio gateway per cluster.

1. Go to the cluster where you want to allow outside traffic into Istio.
1. Click **Tools > Istio.**
1. Expand the **Ingress Gateway** section.
1. Under **Enable Ingress Gateway,** click **True.** The default type of service for the Istio gateway is NodePort. You can also configure it as a [load balancer.]({{<baseurl>}}/rancher/v2.x/en/k8s-in-rancher/load-balancers-and-ingress/load-balancers/)
1. Optionally, configure the ports, service types, node selectors and tolerations, and resource requests and limits for this service. The default resource requests for CPU and memory are the minimum recommended resources.
1. Click **Save.**

**Result:** The gateway is deployed, which allows Istio to receive traffic from outside the cluster.

# Add a Kubernetes Gateway that Points to the Istio Gateway

To allow traffic to reach Ingress, you will also need to provide a Kubernetes gateway resource in your YAML that points to Istio's implementation of the ingress gateway to the cluster.

1. Go to the namespace where you want to deploy the Kubernetes gateway and click **Import YAML.**
1. Upload the gateway YAML as a file or paste it into the form. An example gateway YAML is provided below.
1.  Click **Import.**

```yaml
apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
    name: bookinfo-gateway
    spec:
    selector:
        istio: ingressgateway # use istio default controller
    servers:
        - port:
            number: 80
            name: http
            protocol: HTTP
        hosts:
            - "*"
```

**Result:** You have configured your gateway resource so that Istio can receive traffic from outside the cluster.

Confirm that the resource exists by running:
```
kubectl get gateway
```

The result should be something like this:
```
NAME               AGE
bookinfo-gateway   64m
```

### Confirming that the Kubernetes Gateway Matches Istio's Ingress Controller

In this resource, the selector refers to Istio's default ingress controller by its label, in which the key of the label is `istio` and the value is `ingressgateway`.  To make sure the label is appropriate for the gateway, do the following:

1. Go to the `System` project in your cluster.
1. Within the `System` project, go to the namespace `istio-system`. 
1. Within `istio-system`, there is a workload named `istio-ingressgateway`.
1. Click the name of this workload and go to the **Labels and Annotations** section. You should see that it has the key `istio` and the value `ingressgateway`. This confirms that the selector in the Gateway resource matches Istio's default ingress controller.

You can route traffic into the service mesh with a load balancer or just Istio.

### [Next: Add Deployments and Services]({{<baseurl>}}/rancher/v2.x/en/cluster-admin/tools/istio/setup/deploy-workloads)