## Pods

- `kubectl apply -f grade-submission-portal-pod.yaml` -> Creates the pod
- `k get pods` -> Lists the pods
- `k describe pod <pod-name>` -> Describes the pod in detail
- `k logs <pod-name>` -> Fetches the logs of the pod
- `k logs -f <pod-name>` -> Stream the logs of the pod
- `k logs -f grade-submission-portal -c grade-submission-portal-health-checker` -> Stream the logs of a specific container in the pod
- `kubectl port-forward` --address 0.0.0.0 grade-submission-portal 8080:5001 -> Port forwarding to access the application running inside the pod
- `k delete pod grade-submission-portal`
- `k delete pod --all`

## Namespaces

- `kubectl get namespace`
- `kubectl create namespace grade-submission`
- `kubectl get pods,services -n grade-submission`