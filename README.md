#### Pre-requisite:
1. Kubernets, Compute , Network APIs need to be enabled on the project.
2. Subnet available for the mentioned zone region.

#### Open the GCP shell and run the below steps to create a GKE cluster.

export ZONE=us-east4-a
gcloud container clusters create odin-addons-test --zone=$ZONE --enable-dataplane-v2 --enable-autoscaling --num-nodes 2 --min-nodes 2 --max-nodes 5 --addons=NodeLocalDNS

#### Verification:(Wait for few mins)
#### Run the below command to validate the Cilium add-on
kubectl -n kube-system get pods -l k8s-app=cilium -o wide

#### Run the below command to validate the Horizontal pod autoscaler add-on
kubectl get pods -n kube-system -o wide | grep metrics-server

#### Run the below command to validate the Node Local DNS Cache add-on
kubectl get pods -n kube-system -o wide | grep node-local-dns

#### Cluster Overprovisioner : https://wdenniss.com/gke-autopilot-spare-capacity
#### Run the below commands to install Cluster Overprovisioner

kubectl apply -f class.yaml \n
kubectl apply -f ballon-deploy.yaml \n

#### Sealed Secrets
#### Run the below commands to install and validate Sealed Secrets
helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets<br>
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json >mysecret.json \n
kubeseal -f mysecret.json mysealedsecret.json \n

#### Prop-Autoscaler: 
#### Run the below commands to install Prop Autoscaler
helm repo add cluster-proportional-autoscaler https://kubernetes-sigs.github.io/cluster-proportional-autoscaler
helm repo update
helm upgrade --install cluster-proportional-autoscaler cluster-proportional-autoscaler/cluster-proportional-autoscaler --values prop.yaml

#### Istio: 
#### Run the below commands to install Istio (https://istio.io/v1.16/docs/setup/install/helm/)
helm repo add istio https://istio-release.storage.googleapis.com/charts
kubectl create namespace istio-system
helm install istio-base istio/base -n istio-system
helm install istiod istio/istiod -n istio-system --wait
kubectl label namespace default istio-injection=enabled
kubectl delete po -l app=nginx

