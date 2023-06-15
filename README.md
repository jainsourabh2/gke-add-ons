#### Pre-requisite:

1. Kubernets, Compute , Network APIs need to be enabled on the project. <br />
2. Subnet available for the mentioned zone region. <br />

#### Open the GCP shell and run the below steps to create a GKE cluster.
#### --enable-dataplane-v2 will add Cilium add-on
#### --enable-dataplane-v2 will add Cluster Autoscaler
#### --addons=NodeLocalDNS will add Node Local DNS Cache

export ZONE=us-east4-a <br />
gcloud container clusters create odin-addons-test --zone=$ZONE --enable-dataplane-v2 --enable-autoscaling --num-nodes 2 --min-nodes 2 --max-nodes 5 --addons=NodeLocalDNS <br />

#### Verification:(Wait for few mins)
#### Run the below command to validate the Cilium add-on
kubectl -n kube-system get pods -l k8s-app=cilium -o wide <br />

#### Run the below command to validate the Horizontal pod autoscaler add-on 
kubectl get pods -n kube-system -o wide | grep metrics-server <br /> 

#### Run the below command to validate the Node Local DNS Cache add-on
kubectl get pods -n kube-system -o wide | grep node-local-dns <br />

#### Cluster Overprovisioner : (https://wdenniss.com/gke-autopilot-spare-capacity)
#### Run the below commands to install Cluster Overprovisioner <br />

kubectl apply -f class.yaml <br />
kubectl apply -f ballon-deploy.yaml <br />

#### Sealed Secrets:
#### Run the below commands to install and validate Sealed Secrets
helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets <br />
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json >mysecret.json <br />
kubeseal -f mysecret.json mysealedsecret.json <br />

#### Prop-Autoscaler: 
#### Run the below commands to install Prop Autoscaler
helm repo add cluster-proportional-autoscaler https://kubernetes-sigs.github.io/cluster-proportional-autoscaler <br />
helm repo update <br />
helm upgrade --install cluster-proportional-autoscaler cluster-proportional-autoscaler/cluster-proportional-autoscaler --values prop.yaml <br />

#### Istio: 
#### Run the below commands to install Istio (https://istio.io/v1.16/docs/setup/install/helm/)
helm repo add istio https://istio-release.storage.googleapis.com/charts <br />
kubectl create namespace istio-system <br />
helm install istio-base istio/base -n istio-system <br />
helm install istiod istio/istiod -n istio-system --wait <br />
kubectl label namespace default istio-injection=enabled <br />
kubectl delete po -l app=nginx <br />

