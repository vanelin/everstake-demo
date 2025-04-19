# Omni

It deploys a Talos Kubernetes cluster using Omni, with the following tooling:

* Cilium for CNI & Hubble UI for observability
* ArgoCD for application management
* Prometheus & Grafana for monitoring

## Usage

Once the required machines are registered to Omni and machine classes have been configured, simply run

```bash
omnictl cluster template sync --file cluster-template.yaml
```

Omni will then being to allocate your machines, install Talos, and configure and bootstrap the cluster.

This setup makes use of the [Omni Workload Proxy](https://omni.siderolabs.com/how-to-guides/expose-an-http-service-from-a-cluster) feature,
which allows access to the HTTP front end services *without* the need of a separate external Ingress Controller or LoadBalancer.
Additionally, it leverages Omni's built-in authentication to protect the services, even those services that don't support authentication themselves.

## Applications

Applications are managed by ArgoCD, and are defined in the `apps` directory.
The first subdirectory defines the namespace and the second being the application name.
Applications can be made of Helm charts, Kustomize definitions, or just Kubernetes manifest files in YAML format.

## Extending

1. Commit the contents from the `omni` directory to a new repository
2. Configure ArgoCD to use that repository [bootstrap-app-set.yaml](apps/argocd/argocd/bootstrap-app-set.yaml)
3. Regenerate the ArgoCD bootstrap cluster manifest patch [argocd.yaml](infra/patches/argocd.yaml) (instructions can be found at the top of that file).
4. Commit and push these changes to a hosted git repository the Omni instance has access to.
5. Create a cluster with Omni as described above. 
