# CI/CD, GitOps and ArgoCD

## CI/CD
- CI/CD = continuous integration (build) / continuous deployment

- developers push code in a repository (git, github, gitlab etc)
- a build system monitors the code repository and triggers an automatic build when code is pushed on master branch
  - for example jenkins, bamboo, tekton, argo events etc
- the build continue with tests. if those fails, developers are notified and the process stops
- if the test pass, the build artifacts are stored, for example new version docker images in a docker registry
- also, the CD process will be prepared and/or triggered
- the CD process will usually automatically deploy the new software version in a dev/staging environment
- and then, on-click, deploy in a production environment

## GitOps
- Git plays a key role in GitOps. 
- Git is used as a *single source of truth* for what infrastructure and applications should look like 
- based on *declarative configuration* (not imperative instructions)
- automated processes are then used to take this configuration and make sure the target environment *actual state* matches the *desired state* (described in the repository)

## ArgoCD
In this whole equation, *ArgoCD* is just a simple brick, the CD / deployment part. It is a new tool but quite popular, it is a very well fit in the Kubernetes culture and also it has a nice web interface.

- ArgoCD is running in a kubernetes cluster
- and has the concept of *application*, which is in turn has
    - SOURCE: a git repository ( != application code repo)
      - kubernetes manifests files (yaml) (and/or helm charts and/or kustomize). basically, how you deploy
    - DEST: a kubernetes cluster 
    - when the source changes, ArgoCD automatically deploy what changed in the destination cluster.


## Install and demo
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Access Argo CD
```sh
kubectl port-forward svc/argocd-server -n argocd 8083:443
```
Open https://localhost:8083

Username: admin
Password: `run the command below to print the password`
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

Create main app:

```yaml
cat > main-app.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: main-argocd-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/cod-r/docker-k8s-course.git
    path: argocd
    directory:
      recurse: true
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true 
      selfHeal: true
      allowEmpty: false
EOF
```

### Test GitOps
1. Create a simple pod
```yaml
cat > argocd/applications/test-pod.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-nginx
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1.23.1
EOF
```
2. Commit and push

