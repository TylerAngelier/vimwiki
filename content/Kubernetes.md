# Oracle Linux Debug Container

Runs an Oralce Linux 7 Slim image from dockerhub and destroys it when you detach. Also adds an overide to not inject the Istio sidecar container. 

```bash
kubectl run -i --tty --rm ol7-debug-$USER --image=oraclelinux:7-slim --restart=Never \
  --overrides='{ "apiVersion": "v1", "metadata": {"annotations": { "sidecar.istio.io/inject":"false" } } }' \
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

>**Note:** Admission webhooks that need to guarantee they see the final state of the object in order to enforce policy should use a validating admission webhook, since objects can be modified after being seen by mutating webhooks.

## Types of Dynamic Admission Control Webhooks

### Mutating Admissions Webhook

Mutating admission webhooks are invoked first, and can modify objects sent to the API server to enforce custom defaults. After all object modifications are complete, and after the incoming object is validated by the API server, validating admission webhooks are invoked and can reject requests to enforce custom policies.

### Validating Admission Webhook

Validates the state of the resource before it is invoked by the API server. This is how you can **validate** and **enforce** that resources meet custom criteria. 

## References

* https://www.baeldung.com/java-kubernetes-admission-controller#sample_user_case
	* Good ole Baeldung showing how to do it in Spring Boot 👏

# Multi tenant Kubernetes

High level required steps:
* Need to remove all references to the `DEFAULT` namespace except for `PssNamespaceUtil`.
	* This means when we change the logic of getting the system namespace, it is enforced throughout the entire codebase. 
	* Low-Medium code changes required. 
 * Create system namespace at the top of `SystemResourceService`
	 * Add label/annotation that indicates the `nodeSelector` value to be used by the webhook applications.
 * Validate char limits on
	 * subscriber (10)
	 * env class (5)
	 * system (10)
	 * service names (27)
 * Create calico network policies for each system namespace
 * Make the required changes to the `releaseName` strategy
	 * Hard to be backwards compatable... If we're on the old namespace use the old 
 * Write and deploy two Admission Webhook applications
	 - Need to write a Mutating Admissions Webhook
		- Adds the `nodeSelector` if not present based on the namespace.
	- Need to write a Validating Admission Webhook
		- Validates the `nodeSelector` is added to any pod scheduled in a PSS namespace.
- Migration
	- Provision the entire system namespace and `default` namespace for non-migrated systems. 
	- Manual DNS change to point to the new load balancer. 
		- Use the same database under the hood.

## Naming
Can try to move the namespace but it's really nasty... Basically updating the Helm secret then finding all K8 resources and updating the `metadata.namespace` value manually (could _probably_ do it in code)..
https://blog.random.io/moving-helm-release-to-another-kubernetes-namespace/
Can't stand up everything in the new namespace because Helm won't allow it though...
```
"output": "Error: rendered manifests contain a resource that already exists. Unable to continue with install: ClusterRole \"traefik-ingress-controller\" in namespace \"\" exists and cannot be imported into the current release: invalid ownership metadata; annotation validation error: key \"meta.helm.sh/release-namespace\" must equal \"System_alpha-tangelie-dev-default\": current value is \"default\"\n",
```

Can set a different `release-name` to install the same chart multiple times (even in the same namespace). 
ReleaseName can only be `53` characters...
We need to add the fqSystemName to the release to ensure it's unique. How many characters can that be?
Subscriber Name: 10 char
System name: 10 char

```bash
System_<env>-<sub>-<sys>
# 9 static chars
# 5 = env
# 10 = sub
# 10 = sys
# Total: 34 char

# 53-34=19
# Adding a hyphen between system name and service name gives us 18 char

```
Can't do fqSystemName.. So if we reserve `26` char for the service name, that leaves us with `27` char for the identifier:
```bash
# 5 = env
# 10 = sub
# 10 = sys
# 27 = service-name
# 1 = hypen
<env><sub><system>-<service-name>
```


## Using Extensible Admission Controller

- Need to write a Mutating Admissions Webhook
	- Adds the `nodeSelector` if not present based on the namespace.
- Need to write a Validating Admission Webhook
	- Validates the `nodeSelector` is added to any pod scheduled in a PSS namespace.

General notes:
* Use a `namespaceSelector` to limit these controllers to only Platform and System namespaces.
	* https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#matching-requests-namespaceselector
* Could use this pattern to enforce resource limits too.
	* https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#example-of-idempotent-mutating-admission-webhooks
* Can likely only look only at `pod` `CREATE` and `UPDATE` operations
	* Need to inject `nodeSelector` on pods. Other resources like `Deployments` create the pods and it seems safer to inject it at the pod level to ensure every.
	* Maybe targeting a `Deployment` is better?
		* Less chatty
		* What about `statefulset`, `daemonset` arbitrary pods, etc?
* How to handle the TLS requirements?
	* Can use a `Service` to route the requests
	* Can Istio's sidecar handle this?
	* The helm chart for this admission controller can include the certificate?
	* Should run the `Service` and the pods on port `443`.
	* [Generate the cert with Helm](https://medium.com/nuvo-group-tech/move-your-certs-to-helm-4f5f61338aca)
* Need somewhere for this software to run...
	* Can put it in the `Platform` namespace but likely this software will need to manage that namespace too. Is there a chicken before the egg problem here? Can the controller manage the pods of itself (new pods)? 
		* If only runs on the `Platform` namespace, those nodes **cannot** go down as all pod creation would be rejected. 
	* Just put it in its own namespace and it can be scheduled on any node? Might be good enough for MVP.
		* Argubably this makes the most sense.. This software is part of how we manage PSS OKE clusters (think `kube-system`).
* Do we need a kill switch?
	* Need to be able to **very quickly** turn off these controllers in case of a bug that impacts production.
		* Validate with leadership + security it is better to have pods running in wrong nodes than to have a production outage.
		* "Turn off" is likely always return an `OK` response.
  


```bash
pkcs12 -export -in /secret/webhook-cert.pem -inkey /secret/webhook-key.pem -name webhook -out /shared-config/webhook.p12 -passout pass:
```

# Virtual Nodes

https://blogs.oracle.com/cloud-infrastructure/post/first-principles-making-kubernetes-serverless-with-oci-virtual-nodes
https://blogs.oracle.com/cloud-infrastructure/post/oke-virtual-nodes-deliver-serverless-experience

# Kubernetes on LXC Container

https://gist.github.com/triangletodd/02f595cd4c0dc9aac5f7763ca2264185
