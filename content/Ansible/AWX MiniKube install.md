---
title: "AWX MiniKube Install"
description: "Deploy AWX on a single node with MiniKube"
draft: false
---

Installing AWX using MiniKube was pretty simple.

Install minikube:
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
sudo rpm -Uvh minikube-latest.x86_64.rpm
```

Install podman:
```
dnf -y install podman
```

Start a cluster: 
```
minikube start --cpus=4 --memory=6g --addons=ingress
```

Checl node status:
```
minikube kubectl -- get nodes
```

```
minikube kubectl -- get pods -A
```

Create an alias for easier useage:
```
alias kubectl="minikube kubectl --"
```

Clone the awx operator repository:
```
git clone git@github.com:ansible/awx-operator.git
```

Get the current tag:
```
cd awx-operator 
git tag
```

Check out the latest tag:
```
git checkout tags/2.9.0
```

Create a kubctl wrapper script in your PATH
```
sudo tee /usr/local/bin/kubectl > /dev/null << 'EOF' #!/bin/sh exec minikube kubectl -- "$@" EOF sudo chmod +x /usr/local/bin/kubectl
```

Run the make target:
```
make deploy KUBECTL="minikube kubectl --"
```

Create kustomization.yaml, fill in tag from earlier:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=2.9.0

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.9.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

Install the manifest:
```
kubectl apply -k .
```

Verify the pod is running:
```
kubectl get pods -n awx
```

Set the namespace to awx so we don't have to keep using `-n awx`:
```
kubectl config set-context --current --namespace=awx
```

Create the file `awx-demo.yml` with the following content:
```
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
```

Change awx-demo to the name you want for your project.

Add this file to the resources section of your `kustomization.yaml`:
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=2.9.0
  - awx-demo.yml

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.9.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

Apply changes to create the AWX instance in your cluster:
```
kubectl apply -k .
```

After a few minutes, verify the new resources are created, grab the node port for later:
```
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
```

```
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

Check logs for current status: 
```
kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager
```

Check the pods status:
```
kubectl get pods -n awx
```

```
NAME                                               READY   STATUS             RESTARTS   AGE
awx-demo-postgres-13-0                             1/1     Running            0          7m7s
awx-demo-task-79d8f64ffd-pv5xb                     4/4     Running            0          6m42s
awx-demo-web-c6d789748-nt4n7                       3/3     Running            0          5m20s
awx-operator-controller-manager-76fbc57b56-8fwvm   1/2     ImagePullBackOff   0          17m
```

Get the URL:
```
minikube service awx-demo-service -n awx
```

```
┌───────────┬──────────────────┬─────────────┬───────────────────────────┐
│ NAMESPACE │       NAME       │ TARGET PORT │            URL            │
├───────────┼──────────────────┼─────────────┼───────────────────────────┤
│ awx       │ awx-demo-service │ http/80     │ http://192.168.49.2:32597 │
└───────────┴──────────────────┴─────────────┴───────────────────────────┘
🎉  Opening service awx/awx-demo-service in default browser...
```


Get the admin password using (`<resourcename>-admin-password`):
```
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode ; echo
```

