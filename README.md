- `k delete all --all -n grade-submission` -> Delete all resources in a namespace

## Pods

- `kubectl apply -f grade-submission-portal-pod.yaml` -> Creates the pod
- `k get pods` -> Lists the pods
- `k describe pod <pod-name>` -> Describes the pod in detail
- `k logs <pod-name>` -> Fetches the logs of the pod
- `k logs -f <pod-name>` -> Stream the logs of the pod
- `k logs -f grade-submission-portal -c grade-submission-portal-health-checker` -> Stream the logs of a specific container in the pod
- `kubectl port-forward --address 0.0.0.0 grade-submission-portal 8080:5001` -> Port forwarding to access the application running inside the pod
- `k delete pod grade-submission-portal`
- `k delete pod --all`

## Services

- `ClusterIP Service` is a natural load balancer at the pod level.
- `NodePort Service` exposes a machine port bound to the pod.

## Namespaces

- `kubectl get namespace`
- `kubectl create namespace grade-submission`
- `kubectl get pods,services -n grade-submission`

## Deployments -> Replicas of a Pod

Used for **stateless services**

- `kubectl get deployments,services -n grade-submission`

## Liveness and Readiness

Liveness The liveness endpoint returns a 200 status if the application is operational, or a 500 status if it's not. The liveness probe checks this endpoint and considers the app healthy only if it receives a 200 status. Any other response, or no response at all, triggers a container restart.

Readiness The readiness endpoint verifies if the application has successfully connected to all components necessary for serving traffic. It returns a 200 status only when all required connections and initializations are complete, and a 500 status otherwise. The readiness probe uses this endpoint to determine if a container is ready to accept traffic. If the probe receives anything other than a 200 status, or no response, it keeps the container out of service.

Initial Delay and Period The initial delay sets how long to wait before the first probe runs, while the period determines the frequency of subsequent probes. For example, if an app takes 20 seconds to start, set the liveness probe's initial delay to at least 20 seconds. This ensures the probe only begins checking after the application has had sufficient time to initialize:
```
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 25
  periodSeconds: 5
```

For readiness probes, the initial delay is less critical. An unresponsive app during startup correctly indicates it's not yet ready for traffic.
```
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
```


Usage Use liveness probes to detect and restart unhealthy containers. Use readiness probes to determine when a container is ready to start accepting traffic. Together, they ensure your application remains healthy and responsive in a Kubernetes environment.

## StatefulSet and PVC

`PVC` finds storage in any node in the cluster and bounds to it, each replicaset managed by the `StatefulSet` controller is bound to the same PVC

- `kubectl get statefulset -n grade-submission`
- `kubectl get pvc -n grade-submission`

## ConfigMap & Secrets

Secrets: **data** field expects base64 encoded values

- `echo -n "admin" | base64`
- `k apply -f ./grade-submission-portal/` -> deploy directory

## Horizontal Pod Autoscaler (HPA)

You need the metrics server installed

- `k top pod -n grade-submission`
- `k get hpa -n grade-submission`

## Ingress

The Kubernetes community NGINX Ingress Controller is being retired in March 2026. You need to install the nginx ingress controller

- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.14.1/deploy/static/provider/cloud/deploy.yaml`
- `k get svc -n ingress-nginx`

## Helm

- `helm template ./09-helm/grade-submission-api/` -> Preview the rendered manifests
- `helm package .` -> Package a **Chart**
- `helm install grade-submission-api ./grade-submission-api-1.0.0.tgz -n grade-submission`
- `helm uninstall grade-submission-api -n grade-submission` -> Uninstall release
- `helm list -A`
- `helm upgrade grade-submission-api ./grade-submission-api-1.0.0.tgz -n grade-submission`
- `helm upgrade grade-submission-api . -n grade-submission` -> Directly packages and upgrades
- `helm rollback grade-submission-api 2 -n grade-submission` -> Rollback to previous release

## Helm Package manager

- `helm repo add bitnami https://charts.bitnami.com/bitnami`
- `helm search repo`
- `helm search repo bitnami/mongodb --versions`
- `helm show values bitnami/mongodb > default_values.yaml` -> Extract the default values into a file and override them in values.yaml
- `kubectl create namespace mongodb`
- `helm install mongodb bitnami/mongodb --version 15.6.13 -f values.yaml -n mongodb` -> values.yaml overrides default_values.yaml

Connect to services in another namespace:

- `<service-name>.<service-namespace>.svc.cluster.local:27017`
- `mongodb.mongodb.svc.cluster.local:27017`, specified in `grade-submission-api/values.yaml`

Helm serves as a powerful package manager for Kubernetes, simplifying the deployment of complex software like Elasticsearch, MongoDB, MySQL, and Redis.

Process of Deploying Complex Software with Helm
- Add Helm Repository: `helm repo add bitnami https://charts.bitnami.com/bitnami`. This adds the Bitnami repository, which hosts many popular software charts.

- Update Helm Repositories: `helm repo update`. Ensures you have the latest chart versions available.

- Search for Available Charts: `helm search repo bitnami/mongodb`. Find the chart you need and check its available versions.

Research Default Values:

- Examine the default values.yaml file in the chart's documentation.

- Understand which values you need to modify for your use case.

Create Custom Values File:

Create a file, e.g., my-mongodb-values.yaml, with your custom settings.

Only include values that differ from the defaults.

Install the Chart with Custom Values: `helm install my-mongodb bitnami/mongodb -f my-mongodb-values.yaml`

This command installs MongoDB, applying your custom configuration on top of the default values. Only the fields specified in my-mongodb-values.yaml will override the corresponding default values, while all other settings remain at their default.

Research Before Deployment
Thoroughly read the chart's documentation, and understand the implications of changing default values.

Conclusion

By leveraging Helm as a package manager, you can significantly simplify the deployment and management of complex software in Kubernetes environments, allowing you to focus more on your application and less on the intricacies of Kubernetes configurations.