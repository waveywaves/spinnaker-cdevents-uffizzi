# PoC for CI/CD Interoperability using CDEvents

Technologies involved in PoC
- Spinnaker
- Uffizzi
- Tekton
- Knative
- CDEvents
- CloudEvents Player
- Spinnaker CDEvents Translator
- Uffizzi CDEvents Translator

# Proof-of-Concept Overview

In this PoC for CI/CD Interoperability using CDEvents, we have multiple technologies which are going to perform roles based on the events broadcasted across this system. The following steps define the flow of the entire system and the related entities which are triggered.
Note: A lot more events apart from the ones mentioned below would be broadcasted. Only the events which trigger other processes in the system are highlighted below.

1. A Pull Request is created to the Source Version Control Repository. This broadcasts a `change created` CDEvent which triggers the Uffizzi CDEvents Translator to create a Pull Request Preview Environment for the developer who has created the PR. Upon successful creation of the PR env `environment created` CDEvent is broadcasted.

2. Once the PR is merged, a `change merged` CDEvent is broadcasted. This is read by the broker. If this change had metadata related to creating a new release, then the Tekton EventListener would be called by a Trigger to build and push this image. Once the build is done and the artifact is pushed, the `build finished` event would be broadcaster  

3. The `build finished` event is read by the broker and propogated to the Spinnaker CDEvents Translator. The translator will convert this to a payload with the `artifact` url which would be used to deploy a new service.

4. After the service is deployed, a `service created` CDEvent would be propogated across the system letting us know that the flow is complete.

# Architecture

![Alt text](static/images/architecture.jpg?raw=true "CI/CD Interoperability PoC Architecture")

# Prerequisites

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

## Install Knative

Install knative on minikube using 
https://knative.dev/docs/install/quickstart-install

The following command needs to be running for the user to be able to access all the Kntaive services.
```
minikube tunnel --profile knative
```

### Install CloudEvents Player with Knative

[CloudEvents Player](https://github.com/ruromero/cloudevents-player) provides a UI which will allow us to see the CloudEvents moving in and out of the broker. This will give us a good idea of how the CDEvents are flowing throughout the system.


```
kn service create cloudevents-player \
--image ruromero/cloudevents-player:latest \
--env BROKER_URL=http://broker-ingress.knative-eventing.svc.cluster.local/default/example-broker
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

### Install the CDEventer for Tekton

Install the [CDEventer Controller](https://github.com/afrittoli/cdeventer) for Tekton which will provide a Custom Task which can be used to send CDEvents.

### Setup the flow for Tekton CDEvents propogation

1. Create the Secret, Service Account, Role, RoleBinding and Pipeline. 
2. Create TriggerTemplate with PipelineRun which will trigger the Pipeline
3. Create TriggerBinding and update TriggerTemplate to take in the commit from which the build needs to be completed.
4. Create Trigger. Create EventListener. Reference the Trigger in the EventListener.


## CloudEvents Providers

These providers are extra tools installed for us to better propogate CDEvents throughtout the entire system.

### Uffizzi CDEvents Provider

### Spinnaker CDEvents Provider

## Spinnaker Cloudevents Blogpost

https://iamondemand.com/blog/jenkins-and-spinnaker-turbocharge-your-ci-cd-with-cloud-native/
