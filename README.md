 # my-tomcat-image — Tomcat + PSI Probe (Docker + Helm)

A small repository containing a Docker image build for Apache Tomcat (with PSI Probe) and a Helm chart to deploy it to Kubernetes (examples for UAT and PROD).

## Repository layout

- `my-tomcat-image/` — Docker image build context
  - `Dockerfile` — builds Tomcat image and (optionally) generates self-signed certs
  - `prob.war` — PSI Probe webapp (if present)

- `tomcat-helm-chart/` — Helm chart
  - `Chart.yaml` — chart metadata
  - `values.yaml` — base values
  - `values-uat.yaml` / `values-prod.yaml` — environment overrides
  - `templates/` — Kubernetes manifests (Deployment, Service, ConfigMaps, Secrets, helpers)

## Prerequisites

- Docker (or other OCI builder)
- Kubernetes cluster (tested with k3s)
- Helm 3.x
- `kubectl` configured to talk to the target cluster

## Quick start

1) Build the Docker image

```bash
cd my-tomcat-image
docker build -t my-tomcat-image:latest .
```

2) Push image to a registry (optional)

```bash
# Tag and push to your registry
docker tag my-tomcat-image:latest <registry>/my-tomcat-image:latest
docker push <registry>/my-tomcat-image:latest
```

3) Install the Helm chart

UAT:

```bash
helm upgrade --install tomcat-uat ./tomcat-helm-chart \
  -f ./tomcat-helm-chart/values.yaml \
  -f ./tomcat-helm-chart/values-uat.yaml
```

PROD:

```bash
helm upgrade --install tomcat-prod ./tomcat-helm-chart \
  -f ./tomcat-helm-chart/values.yaml \
  -f ./tomcat-helm-chart/values-prod.yaml
```

Notes:
- Prefer using an image registry for Kubernetes clusters that cannot access a local Docker daemon.
- Use `--set image.repository=<repo> --set image.tag=<tag>` or edit `values-*.yaml` to point to your image.

## Accessing the application

If the chart exposes a Service of type `ClusterIP`, you can port-forward locally for testing:

```bash
# adjust the service name as installed (example uses the `tomcat-uat` release)
kubectl port-forward svc/tomcat-uat-tomcat 8080:8080 8443:8443
# then open:
#  - http://localhost:8080 (Tomcat)
#  - https://localhost:8443/probe (PSI Probe)
```

In production, configure an Ingress or `ServiceType: LoadBalancer` and TLS using cert-manager or your cloud provider.

## Configuration and secrets

- Do NOT hardcode credentials in `values-*.yaml`. Use Kubernetes `Secret`s and Helm value injection.
- The chart includes `secret-credentials.yaml` and ConfigMaps for server configuration — update those files or provide values during `helm install`.

## Troubleshooting

- Pod CrashLoopBackOff: inspect logs and events

```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

- TLS warnings: expected with self-signed certs. Use proper CA-signed certs in production.
- Authentication issues: verify Helm values and Kubernetes Secrets

## Useful commands

```bash
# List resources for a release
kubectl get all -l app.kubernetes.io/instance=tomcat-uat

# Follow logs
kubectl logs -f deployment/tomcat-uat-tomcat

# Port-forward example
kubectl port-forward svc/tomcat-uat-tomcat 8080:8080

# Uninstall
helm uninstall tomcat-uat
```

## Security notes

- Run Tomcat as a non-root user (the Dockerfile in this repo is intended to do this).
- Store sensitive values in Kubernetes Secrets and restrict RBAC access to them.

# Port forward to access the services
kubectl port-forward service/tomcat-uat-tomcat-helm-chart 8080:8080
kubectl port-forward service/tomcat-uat-tomcat-helm-chart 8443:8443

Access URLs:

    Tomcat Home: http://localhost:8080

    PSI Probe: https://localhost:8443/probe

4. Default Credentials

UAT Environment:

    Username: uat_admin

    Password: uat_password123

PROD Environment:

    Username: prod_admin

    Password: prod_secure_password_456



# Conclusion

This solution successfully demonstrates a production-ready Tomcat deployment with comprehensive security, monitoring, and environment management. The Helm chart provides a robust foundation that can be extended for enterprise requirements while maintaining simplicity and reliability.
