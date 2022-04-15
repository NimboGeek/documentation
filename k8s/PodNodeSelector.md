---
marp: true
---
<!-- theme: gaia -->
<!-- _class: lead -->
<!-- paginate: true -->
<!-- backgroundColor: #fff -->
<!-- backgroundImage: url('https://marp.app/assets/hero-background.svg') -->
<!-- headingDivider: 2 -->

# Namespaced pod selector

Jérôme Narbonne

## Prepare the cluster

Deploy on your kuberentes cluster the admission plugins `PodNodeSelector`.

Example with minikube:

```bash
minikube start  --extra-config=apiserver.enable-admission-plugins=PodNodeSelector
```

## Label nodes

Then label your nodes with node affinity labels:

```bash
kubectl label node kubeprod01 env=production
```

## Create a namespace

Finally, create a namespace with the annotation `scheduler.alpha.kubernetes.io/node-selector`  matching previously created label:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  annotations:
    scheduler.alpha.kubernetes.io/node-selector: env=production
```

## Conclusion

From now, all pods created in the namespace will automatically be added with a node selector matching the label of the namespace.
