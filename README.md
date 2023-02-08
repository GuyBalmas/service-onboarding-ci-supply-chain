# service-onboarding-ci-supply-chain

1. Create Dockerhub credentials secret:
```bash
#export creds as env vars
export DOCKERHUB_USER=<username>
export DOCKERHUB_PASSWORD=<password>

#create credentials secret
kubectl create secret docker-registry docker-registry-credentials \
    --docker-username=$DOCKERHUB_USER \
    --docker-password=$DOCKERHUB_PASSWORD \
    --docker-server=https://index.docker.io/v1/ \
    --namespace apps -o yaml --dry-run=client > dockerhub-secret.yaml

#apply
kubectl apply -f dockerhub-secret.yaml
```

2 .Create GitHub credentials secret:
```bash
#export creds as env vars
export GITHUB_USER=<username>
export GITHUB-ACCESS-TOKEN=<access-token-with-read-write-permissions>
```
Create `github-secret.yaml` file: 
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: github-secret
  annotations:
    tekton.dev/git-0: https://github.com        # ! required
type: kubernetes.io/basic-auth                  # ! required
stringData:
  username: ${GIT-USERNAME}
  password: ${GITHUB-ACCESS-TOKEN}
```
Apply
```bash
#apply
kubectl apply -f github-secret.yaml
```

3. Attach to default service-account
```bash
#get service account manifest
kubectl get sa default -n apps -o yaml > default-apps-sa.yaml
```
default-apps-sa.yaml
```yaml
apiVersion: v1
imagePullSecrets:
- name: tap-registry
kind: ServiceAccount
metadata:
  name: default
  namespace: apps
secrets:
- name: default-token-82fs2
- name: tap-registry
  # add dockerhub-secret
- name: docker-registry-credentials
  # add github-secret
- name: github-secret
```
```bash
#apply 
kubectl apply -f default-apps-sa.yaml

#verify
kubectl describe sa default -n apps
```

**output**:
```bash
Name:                default
Namespace:           apps
Labels:              <none>
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::384617649113:role/tap-sample-workload-tap-guy-lab-p8m8nm3rts18
                     policies.kyverno.io/last-applied-patches:
                       add-irsa-label.dev-ns-preperation.kyverno.io: added /metadata/annotations/eks.amazonaws.com~1role-arn
Image pull secrets:  tap-registry
Mountable secrets:   default-token-82fs2
                     tap-registry
                     docker-registry-credentials        # Here
                     github-secret                      # Here
Tokens:              default-token-82fs2
Events:              <none>
```

# Debug
```bash
k apply -f service-onboarding-ci-supply-chain.yaml

tanzu apps workload get ecsdemo-frontend

k describe po ecsdemo-frontend-build-2tmdr-pod

tanzu apps workload tail ecsdemo-frontend

kubectl -n apps logs ecsdemo-frontend-build-k5btv-pod -c step-build-and-push
```

# argo config
```bash
kubectl apply -f argo-config-template.yaml

k get clusterconfigtemplate.carto.run/argo-config-template


```