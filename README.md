#### Pre-requisite:
1. Kubernets, Compute , Network APIs need to be enabled on the project.
2. Subnet available for the mentioned zone region.

#### Open the GCP shell and run the below steps.

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

kubectl apply -f class.yaml
kubectl apply -f ballon-deploy.yaml

#### Sealed Secrets
helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json >mysecret.json
kubeseal -f mysecret.json mysealedsecret.json

#### Prop-Autoscaler: 
helm repo add cluster-proportional-autoscaler https://kubernetes-sigs.github.io/cluster-proportional-autoscaler
helm repo update
helm upgrade --install cluster-proportional-autoscaler cluster-proportional-autoscaler/cluster-proportional-autoscaler --values prop.yaml

Istio : https://istio.io/v1.16/docs/setup/install/helm/
kubectl label namespace default istio-injection=enabled
kubectl delete po -l app=nginx

