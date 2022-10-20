# PoC for CI/CD Interoperability using CDEvents

CI/CD systems involved in PoC
- Spinnaker
- Uffizzi
- Tekton

Event Broker used
- KNative

# Prerequisites

## Install Knative

Install knative on minikube using 
https://knative.dev/docs/install/quickstart-install

https://developingfordata.com/category/kubernetes/knative/


## Install Spinnaker using Helm

```
helm repo add spinnaker https://helmcharts.opsmx.com/
helm install my-release spinnaker/spinnaker
```

After running the above commands you should able to see the output below.
```
helm install my-release spinnaker/spinnaker 
"spinnaker" has been added to your repositories
NAME: my-release
LAST DEPLOYED: Tue Oct 18 15:32:15 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. You will need to create 2 port forwarding tunnels in order to access the Spinnaker UI:
  export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" -o jsonpath="{.items[0].metadata.name}")
  kubectl port-forward --namespace default $DECK_POD 9000

  export GATE_POD=$(kubectl get pods --namespace default -l "cluster=spin-gate" -o jsonpath="{.items[0].metadata.name}")
  kubectl port-forward --namespace default $GATE_POD 8084

2. Visit the Spinnaker UI by opening your browser to: http://127.0.0.1:9000

To customize your Spinnaker installation. Create a shell in your Halyard pod:

  kubectl exec --namespace default -it my-release-spinnaker-halyard-0 bash

For more info on using Halyard to customize your installation, visit:
  https://www.spinnaker.io/reference/halyard/

For more info on the Kubernetes integration for Spinnaker, visit:
  https://www.spinnaker.io/reference/providers/kubernetes-v2/
```

### Initial Test

For the initial test we will deploy a simple Nginx deployment.

Follow [this](https://earthly.dev/blog/spinnaker-kubernetes/) article to create
a simple nginx deployment to see if spinnaker works against the cluster properly.

## Install Tekton 

Run the following command to deploy Tekton in the `tekton-pipelines` namespace.

```
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.40.2/release.yaml
```

### Initial Test

Follow [https://earthly.dev/blog/building-k8s-tekton/] to create the resources and deploy your first Tekton Pipeline.

## Sockeye Cloudevents UI

https://github.com/n3wscott/sockeye#from-source

## Spinnaker Cloudevents Blogpost

https://iamondemand.com/blog/jenkins-and-spinnaker-turbocharge-your-ci-cd-with-cloud-native/

# Architecture

