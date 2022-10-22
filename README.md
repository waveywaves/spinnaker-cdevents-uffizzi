# PoC for CI/CD Interoperability using CDEvents

Technologies involved in PoC
- Spinnaker
- Uffizzi
- Tekton
- Knative
- CDEvents
- Cloudevents Player
- Spinnaker CDEvents provider
- Uffizzi CDEvents provider

# Architecture

![Alt text](static/images/architecture.jpg?raw=true "CI/CD Interoperability PoC Architecture")

# Prerequisites

## Install Knative

Install knative on minikube using 
https://knative.dev/docs/install/quickstart-install

The following command needs to be running for the user to be able to access all the Kntaive services.
```
minikube tunnel --profile knative
```

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

## Install Tekton 

Run the following command to deploy Tekton in the `tekton-pipelines` namespace.

```
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

After installing Tekton install Triggers whihch will help us create Triggers which can execute Pipelines based on Events. 
```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

Install the Tekton Dashboard from where we can monitor all the Tekton Pipelines.
```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
```

## CloudEvents Player

This is the UI which we are going to use to check what cloudevents are moving in and out of the broker. This will give us a good idea of the entire system. 
https://github.com/ruromero/cloudevents-player 

```
kn service create cloudevents-player \
--image ruromero/cloudevents-player:latest \
--env BROKER_URL=http://broker-ingress.knative-eventing.svc.cluster.local/default/example-broker
```

## CloudEvents Providers

These providers are extra tools installed for us to better propogate CDEvents throughtout the entire system.

### CDEventer for Tekton

https://github.com/afrittoli/cdeventer

### Uffizzi CDEvents Provider

### Spinnaker CDEvents Provider

## Spinnaker Cloudevents Blogpost

https://iamondemand.com/blog/jenkins-and-spinnaker-turbocharge-your-ci-cd-with-cloud-native/
