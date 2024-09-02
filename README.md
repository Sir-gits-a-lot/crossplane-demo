# crossplane-demo

kind create cluster --config kind-configs/kind-crossplane.yaml -n xplane

# Offline Chart
helm install crossplane --namespace crossplane-system --create-namespace ./helm-charts/crossplane

helm install argocd --namespace argocd --create-namespace ./helm-charts/argo-cd/

# Crossplane Demo

k apply -f ./prerequisites/providers

kubectl create secret generic aws-creds -n crossplane-system --from-file=creds=.aws/credentials

k apply -f ./prerequisites/crossplane-provider-config-aws.yaml

k apply -f ./crossplane-manifests/XRD-EKS.yaml

k apply -f ./crossplane-manifests/Composition-EKS.yaml

k apply -f ./prerequisites/argo-repo.yaml

k apply -f ./argo-manifests/argo-app.yaml



Extract kubeconfig of Workload Cluster

kubectl --namespace crossplane-system     get secret a6281e42-7ad4-4930-93c3-13bed5413fdf-ekscluster   --output jsonpath="{.data.kubeconfig}"     | base64 -d > teatom-eks.kubeconfig

# Extract kubeconfig through AWS CLI
aws eks update-kubeconfig --name teatom-crossplane --region ap-south-1 --kubeconfig teatom-eks.kubeconfig

aws eks update-kubeconfig --name teatom-crossplane --region ap-south-1



# Package based Workflow

# Create secret for private repository
kubectl create -n crossplane-system secret docker-registry regcred --docker-server=084726943794.dkr.ecr.ap-south-1.amazonaws.com --docker-username=AWS   --docker-password=$(aws ecr get-login-password --region ap-south-1)

# Build crossplane config. package
kubectl crossplane build configuration

# Login to AWS ECR
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 084726943794.dkr.ecr.ap-south-1.amazonaws.com

# Push a config. package
kubectl crossplane push configuration 084726943794.dkr.ecr.ap-south-1.amazonaws.com/crossplane-config:v0.0.2