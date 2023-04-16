# Oracle Linux Debug Container

Runs an Oralce Linux 7 Slim image from dockerhub and destroys it when you detach. Also adds an overide to not inject the Istio sidecar container. 

```bash
kubectl run -i --tty --rm ol8-debug-$USER --image=oraclelinux:8 --restart=Never \
  --annotations='sidecar.istio.io/inject=false' --annotations='traffic.sidecar.istio.io/excludeOutboundIPRanges=10.96.0.1/32' \
  -- bash
```

Install the development tools (things like `openssl`):

```bash
yum group install 'Development Tools'
```

# Flags

## Get just the name of pods

```bash
--no-headers -o custom-columns=":metadata.name"
# Example:
# kubectl get pods --no-headers -o custom-columns=":metadata.name"
```

## Find pods by label

```bash
-l <label>=<label-value>
# Example
# kubectl get pods -l app.kubernetes.io/instance=pss-control-plane
# kubectl get pods -l app=incidents-service
```

# Admission Controllers

Quoting from the Kubernetes [documentation](https://kubernetes.io/docs/admin/admission-controllers/):

> An admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized. The controllers […] are compiled into the kube-apiserver binary, and may only be configured by the cluster administrator. […] If any of the controllers in either phase reject the request, the entire request is rejected immediately and an error is returned to the end-user.

There are many admission controllers in the Kubernetes core, for example `LimitRanger` or `PodPreset`. A complete list is provided [here](https://kubernetes.io/docs/admin/admission-controllers/#what-does-each-admission-controller-do).

[Oracle OKE Supported Admission Controllers](https://docs.oracle.com/en-us/iaas/Content/ContEng/Reference/contengadmissioncontrollers.htm) #oci [[oci]]
[Admission Controllers Reference](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook) #kubernetes
## PodNodeSelector

Great article going over using PodNodeSelector Admission Controller: https://www.mgasch.com/2018/01/podnodesel/

Ok, so why would I use that one instead of simply specifying `NodeSelectors` in the pod manifest? Well, using this admission controller has some advantages:
-   As the name implies, it enforces node selectors at admission time before creating the pod object
-   This puts less pressure on the master components, for example the scheduler; if there is no matching namespace or a conflict, the pod object will not be created
-   You can define a default selector for namespaces that have no label selector specified
-   Ultimately, it gives the cluster administrator more control over the node selection process (hard enforcement)

# Dynamic Admission Control

https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/

>Admission webhooks are HTTP callbacks that receive admission requests and do something with them. You can define two types of admission webhooks, [validating admission webhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook) and [mutating admission webhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook). Mutating admission webhooks are invoked first, and can modify objects sent to the API server to enforce custom defaults. After all object modifications are complete, and after the incoming object is validated by the API server, validating admission webhooks are invoked and can reject requests to enforce custom policies.

> **Note**: Admission webhooks that need to guarantee they see the final state of the object in order to enforce policy should use a validating admission webhook, since objects can be modified after being seen by mutating webhooks.

## Types of Dynamic Admission Control Webhooks

### Mutating Admissions Webhook

Mutating admission webhooks are invoked first, and can modify objects sent to the API server to enforce custom defaults. After all object modifications are complete, and after the incoming object is validated by the API server, validating admission webhooks are invoked and can reject requests to enforce custom policies.

### Validating Admission Webhook

Validates the state of the resource before it is invoked by the API server. This is how you can **validate** and **enforce** that resources meet custom criteria. 

## References

* https://www.baeldung.com/java-kubernetes-admission-controller#sample_user_case
	* Good ole Baeldung showing how to do it in Spring Boot 👏

# Virtual Nodes

https://blogs.oracle.com/cloud-infrastructure/post/first-principles-making-kubernetes-serverless-with-oci-virtual-nodes
https://blogs.oracle.com/cloud-infrastructure/post/oke-virtual-nodes-deliver-serverless-experience

# Kubernetes on LXC Container

https://gist.github.com/triangletodd/02f595cd4c0dc9aac5f7763ca2264185
