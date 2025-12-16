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