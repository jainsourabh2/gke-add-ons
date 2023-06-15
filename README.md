# Pre-requisite:
1. Kubernets, Compute , Network APIs need to be enabled on the project.
2. Subnet available for the mentioned zone region.

# Open the GCP shell and run the below steps.

export ZONE=us-east4-a
gcloud container clusters create odin-addons-test --zone=$ZONE --enable-dataplane-v2 --enable-autoscaling --num-nodes 2 --min-nodes 2 --max-nodes 5 --addons=NodeLocalDNS

# Verification:(Wait for few mins)
# Run the below command to validate the Cilium add-on
kubectl -n kube-system get pods -l k8s-app=cilium -o wide

# Run the below command to validate the Horizontal pod autoscaler add-on
kubectl get pods -n kube-system -o wide | grep metrics-server

# Run the below command to validate the Node Local DNS Cache add-on
kubectl get pods -n kube-system -o wide | grep node-local-dns


# Cluster Overprovisioner : https://wdenniss.com/gke-autopilot-spare-capacity

Class.yaml

apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: balloon-priority
value: -10
preemptionPolicy: Never
globalDefault: false
description: "Balloon pod priority."

kubectl apply -f Class.yaml

ballon-dpeloy.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: balloon-deploy
spec:
  replicas: 10
  selector:
    matchLabels:
      app: balloon
  template:
    metadata:
      labels:
        app: balloon
    spec:
      priorityClassName: balloon-priority
      terminationGracePeriodSeconds: 0
      containers:
      - name: ubuntu
        image: ubuntu
        command: ["sleep"]
        args: ["infinity"]
        resources:
            requests:
              cpu: 200m
              memory: 250Mi

kubectl apply -f ballon-dpeloy.yaml

helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets

Create Secret:
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json >mysecret.json
Create Sealed Secret:
kubeseal -f mysecret.json mysealedsecret.json


Prop-Autoscaler: 
helm repo add cluster-proportional-autoscaler https://kubernetes-sigs.github.io/cluster-proportional-autoscaler
helm repo update
helm upgrade --install cluster-proportional-autoscaler \
    cluster-proportional-autoscaler/cluster-proportional-autoscaler --values prop.yaml

prop.yaml

affinity: {}
config:
  linear:
    coresPerReplica: 4
    nodesPerReplica: 3
    preventSinglePointFailure: true
#  ladder:
#    coresToReplicas:
#      - [ 1, 1 ]
#      - [ 64, 3 ]
#      - [ 512, 5 ]
#      - [ 1024, 7 ]
#      - [ 2048, 10 ]
#      - [ 4096, 15 ]
#    nodesToReplicas:
#      - [ 1, 1 ]
#      - [ 2, 2 ]
#  linear:
#    coresPerReplica: 2
#    nodesPerReplica: 1
#    min: 1
#    max: 100
#    preventSinglePointFailure: true
#    includeUnschedulableNodes: true
image:
  repository: registry.k8s.io/cpa/cluster-proportional-autoscaler
  pullPolicy: IfNotPresent
  tag:
imagePullSecrets: []
fullnameOverride:
nameOverride:
nodeSelector: {}
options:
  alsoLogToStdErr:
  logBacktraceAt:
  logDir:
  #  --v=0: log level for V logs
  logLevel:
  # Defaulting to true limits use of ephemeral storage
  logToStdErr: true
  maxSyncFailures:
  namespace:
  nodeLabels: {}
  #  label1: value1
  #  label2: value2
  pollPeriodSeconds:
  stdErrThreshold:
  target: deployment/nginx-deployment
  vmodule:
podAnnotations: {}
podSecurityContext: {}
  # fsGroup: 2000
replicaCount: 1
resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi
securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000
serviceAccount:
  create: true
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  # If set and create is false, no service account will be created and the expectation is that the provided service account already exists or it will use the "default" service account
  name:
tolerations: []
priorityClassName: ""

Istio : https://istio.io/v1.16/docs/setup/install/helm/
kubectl label namespace default istio-injection=enabled
kubectl delete po -l app=nginx

