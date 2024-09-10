
# OpenAppSec Project with Minikube & Ingress

This project is an implementation of ![Open AppSec](https://docs.openappsec.io/getting-started/start-with-kubernetes/install-using-interactive-cli-tool-ingress-nginx) (open-appsec) security on top of an NGINX Ingress Controller in a Minikube cluster. The project leverages a declarative approach, using YAML files to define Kubernetes resources for easy customization. This method can be preferable for developers who want to avoid the complexity of Helm charts or interactive CLI scripts, and prefer full control over the resources in a Kubernetes cluster.

The architecture of this project secures a simple hello-world application exposed via NGINX Ingress using Open AppSec security policies enforced via custom rules.

## Architecture Overview
The structure consists of:
- **Hello-World Service**: A basic service returning a "Hello World" message.
- **NGINX Ingress**: Used to expose the Hello-World service to the outside world.
- **Open AppSec**: Implemented on top of the ingress for security enforcement, utilizing multiple resources such as policies, CRDs, and services.

All these components are running inside a single node in the Minikube cluster.

## Diagrams:
### Architecture
![High-Level Diagram of the Architecture](https://github.com/miloskec/ingress-test-app/blob/master/docs-images/openappsec.png)

This architecture ensures that external traffic enters the cluster via the NGINX Ingress Controller, passes through the Open AppSec Agent (which applies security rules), and is then routed to the Hello-World application service.

### Requests
![SQL Injection query](https://github.com/miloskec/ingress-test-app/blob/master/docs-images/sql-inject-url.png)
![SQL Injection post](https://github.com/miloskec/ingress-test-app/blob/master/docs-images/sql-inject-post.png)
![Clean request](https://github.com/miloskec/ingress-test-app/blob/master/docs-images/no-injections.png)

## Installation Instructions

This project relies on Kubernetes Custom Resource Definitions (CRDs) and related services and deployments. The following steps outline the correct order to apply the YAML configurations.

### 1. Apply Custom Resource Definitions (CRDs)
CRDs define custom resources that Open AppSec uses, such as policies, practices, and trusted sources.

```bash
kubectl apply -f crds/crd-trustedsources.yaml
kubectl apply -f crds/crd-practices.yaml
kubectl apply -f crds/crd-sourcesidentifiers.yaml
kubectl apply -f crds/crd-customresponses.yaml
kubectl apply -f crds/crd-exceptions.yaml
kubectl apply -f crds/crd-logtriggers.yaml
kubectl apply -f crds/crd-policies.yaml
```

### 2. Create the Namespace

```bash
kubectl apply -f appsec-namespace.yaml
```

### 3. Apply Configurations
These are settings that Open AppSec relies on to manage the environment and enforce rules.

```bash
kubectl apply -f resources/appsec-settings-configmap.yaml
kubectl apply -f resources/appsec-settings-secret.yaml
kubectl apply -f resources/open-appsec-nginx-ingress-configmap.yaml
kubectl apply -f resources/open-appsec-nginx-ingress-admission-secret.yaml
```

### 4. Deploy Open AppSec Controllers
Controllers manage the Open AppSec security policies.

```bash
kubectl apply -f resources/open-appsec-nginx-ingress-controller-deployment.yaml
kubectl apply -f resources/open-appsec-nginx-ingress-controller-svc.yaml
kubectl apply -f resources/open-appsec-nginx-ingress-admission-svc.yaml
```

### 5. Deploy Supporting Services
These services handle additional functionality, such as learning, storage, and policy enforcement.

```bash
kubectl apply -f resources/open-appsec-learning-deployment.yaml
kubectl apply -f resources/open-appsec-learning-svc.yaml
kubectl apply -f resources/open-appsec-shared-storage-deployment.yaml
kubectl apply -f resources/open-appsec-shared-storage-svc.yaml
```

### 6. Deploy the Open AppSec Agent
The agent enforces the Open AppSec rules on the ingress traffic.

```bash
kubectl apply -f open-appsec-agent-deployment.yaml
```

### 7. Apply Security Policies
The policy ensures the proper security measures are enforced on the ingress.

```bash
kubectl apply -f open-appsec-policy.yaml
```

### 8. Deploy the Application and Services
Here, we deploy the Hello-World application and service that will be exposed by the ingress.

```bash
kubectl apply -f basic.yaml
```

### 9. Handle Webhook Issues (Optional)
If there are issues with the webhook during ingress deployment (for example, certificate validation errors), you can temporarily disable the webhook by running the following command:

```bash
kubectl delete validatingwebhookconfiguration open-appsec-open-appsec-k8s-nginx-ingress-admission
```

### 10. Deploy the Ingress Resources
Apply the ingress files that expose the Hello-World application and enforce Open AppSec security policies:

```bash
kubectl apply -f resources/example-ingress.yaml
kubectl apply -f resources/example-ingress-appsec.yaml
```

## Webhooks and Why They Are Disabled
Webhooks in Kubernetes are used for validating and mutating resource configurations. In our case, we have disabled the Open AppSec NGINX Ingress Controller Admission Webhook to avoid complications during development, as webhook failures can block ingress resource creation. In production environments, you should properly configure and use these webhooks for enhanced security.

---

## Notes
This setup is currently implemented using **Minikube**. Make sure the **ingress addon** is enabled before starting.

```bash
minikube addons enable ingress
```

Check ingress controller pods with:

```bash
kubectl get pods -n ingress-nginx
```

