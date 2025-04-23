########################
# Installing Istio CLI #
########################

# Go to https://github.com/istio/istio/releases
# Pick a release
# Download and unpack the release, and move it to a directory set in the `PATH`

echo $PATH
cd istio-1.11.3
pwd  # Example output /root/istio-1.11.3/bin

PATH=$PATH:/root/istio-1.11.3/bin

####################
# Installing Istio #
####################

istioctl profile list

istioctl profile dump demo

istioctl manifest install --set profile=demo

kubectl get crds | grep 'istio.io'

kubectl --namespace istio-system get services

# Confirm that `EXTERNAL-IP` of `istio-ingressgateway` is not `pending`, unless using minikube

kubectl --namespace istio-system get pods

############################
# Manual Sidecar Injection #
############################
 

vi alpine.yml #Add Below service and deployment yaml and save.

apiVersion: v1
kind: Service
metadata:
  name: alpine
  labels:
    app: alpine
spec:
  ports:
  - port: 80
    name: http
  selector:
    app: alpine

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpine
spec:
  selector:
    matchLabels:
      app: alpine
  template:
    metadata:
      labels:
        app: alpine
    spec:
      containers:
      - name: alpine
        image: alpine
        command: ["sleep"]
        args: ["100000"]

istioctl kube-inject --filename istio/alpine.yml

istioctl kube-inject --filename istio/alpine.yml | kubectl apply --filename -

kubectl get pods

kubectl describe pod --selector app=alpine

kubectl delete --filename istio/alpine.yml

###############################
# Automatic Sidecar Injection #
###############################

kubectl apply --filename istio/alpine.yml

kubectl get pods

kubectl label namespace default istio-injection=enabled

kubectl describe namespace default

kubectl rollout restart deployment alpine

kubectl get pods

kubectl describe pod --selector app=alpine

kubectl label namespace default istio-injection-

kubectl rollout restart deployment alpine

kubectl get pods

kubectl delete --filename istio/alpine.yml

