# Kubernetes-Cluster-with-Istio---OPA
This project demonstrates how to enhance the security of a Kubernetes (v1.26) cluster by integrating:
Istio: As a service mesh to manage and secure inter-service communication using mTLS, traffic routing, and observability.
Open Policy Agent (OPA): For fine-grained policy enforcement on resource admission and traffic control via Istio’s Envoy proxy.
Kubernetes: As the container orchestration platform.

Goal: Block unsafe deployments, restrict traffic, and ensure compliance with security policies in a zero-trust environment.

🏗️ Architecture Diagram
The diagram shows Kubernetes services communicating through Istio, with OPA enforcing rules via the Envoy ext-authz filter.


![Untitled Diagram](https://github.com/user-attachments/assets/0ce6f8f6-03b9-43e0-a7d2-1c9def8bc089)



🛠️ Installation Steps
1. Prerequisites
  Kubernetes v1.26+
  kubectl
  Istio CLI (istioctl)
  OPA-Envoy plugin
  Kustomize or Helm (optional for deployment)

2. Set Up the Cluster
kind create cluster --name istio-opa-secure-cluster

3. Install Istio
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled

4. Deploy OPA with Envoy External Authorization
kubectl apply -f opa-istio-deployment.yaml

5. Apply Sample App and Policies
kubectl apply -f nginx-deployment.yaml
kubectl apply -f k8sRequiredregistry_constraint.yaml

🔒 Example Policies
✅ Allow Traffic Only from Approved Registries
package istio.authz
default allow = false
allow {
  input.attributes.request.http.headers["x-image-registry"] == "docker.io"
}


❌ Deny Pods Pulling Images from Untrusted Registries
package kubernetes.admission
deny[msg] {
  input.request.kind.kind == "Pod"
  not startswith(input.request.object.spec.containers[_].image, "mytrusted.registry/")
  msg := "Image must be from trusted registry"
}
📸 Screenshots

![image](https://github.com/user-attachments/assets/5b84f558-8ef6-46ff-8927-3ca32d5eb1df)
![image](https://github.com/user-attachments/assets/b8473b95-dcfa-4276-a81b-fa5fd34de218)
![image](https://github.com/user-attachments/assets/82629906-28f5-4f6d-b5d2-b8796fcb0eb6)

🚫 Denied Deployment

kubectl apply -f bad-pod.yaml

Output:

Error from server (Forbidden): error when creating "bad-pod.yaml":
admission webhook "validation.gatekeeper.sh" denied the request: Image must be from trusted registry
