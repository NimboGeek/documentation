# API authentification with Service Account

Following instructions allow you to create an user account for authentification to the API server with restricted access.

## Create a namespace

```bash
kubectl create namespace devops-tools
```

## Create the service account

```bash
kubectl create serviceaccount api-service-account -n devops-tools
```

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-service-account
  namespace: devops-tools
EOF
```

## Create a Cluster Role

```bash
cat <<EOF | kubectl apply -f -
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-cluster-role
  namespace: devops-tools
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

## Create a CluserRole Binding

```bash
cat <<EOF | kubectl apply -f -
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: api-role-binding
subjects:
- namespace: devops-tools 
  kind: ServiceAccount
  name: api-service-account 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: api-cluster-role 
EOF
```

## Validate Service Account Access Using kubectl

```bash
kubectl auth can-i get pods --as=system:serviceaccount:devops-tools:api-service-account
```

```bash
kubectl auth can-i delete deployments --as=system:serviceaccount:devops-tools:api-service-account
```

## Validate Service Account Access Using API call

```bash
kubectl get serviceaccount api-service-account  -o=jsonpath='{.secrets[0].name}' -n devops-tools

kubectl get secrets  <service-account-token-name>  -o=jsonpath='{.data.token}' -n devops-tools | base64 -D

kubectl get endpoints | grep kubernetes

curl -k  https://35.226.193.217/api/v1/namespaces -H "Authorization: Bearer eyJhbGcisdfsdfsdfiJ9.eyJpc3MiOisdfsdfVhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3sdf3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImFwaS1zZXJ2aWNlsdfglkjoer876Y3BmNWYiLsdfsdfRlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmFwaS1zZXJ2aWNlLWFjY291bnQifQ.u5jgk2px_lEs3f5e5lh_UfS40fndtDKMTY5UvsdfrtsuhtgjrUj-ezrRXeLS8SLOae4DuOGGGbInSg_gIo6oO7bLHhCixWOBJNOA5gzrLVioof_kHDR8gH5crrsWoR-GSSsdfgsdfg6fA_LDOqdxzqMC0WlXt6tgHfrwIHerPPvkI6NWLyCqX9tn_akpcihd-bL6GwOKlph17l_ND710FnTkE7kBfdXtQWWxaPPe06UEmoKK9t-0gsOCBxJxViwhHkvwqetr987q9enkadfgd_2cY_CA"
```

[Reference](https://devopscube.com/kubernetes-api-access-service-account/)