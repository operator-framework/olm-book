# Subscriptions

Subscriptions are Custom Resources that relate an operator to a catalog source. Subscriptions describe which channel
of an operator package to subscribe to and whether to install the resources automatically or manually. If set to automatic,
the Subscription ensures OLM will manage and upgrade the operator to ensure the latest version is always running in the cluster.

Each Subscription in a namespace acts as a part of a set of operators for the namespace - think of a Subscription as an 
entry in a python `requirements.txt`. If OLM is unable to resolve part of the set, it knows that resolving the entire set will fail, 
so it will bail out of the installation of operators for that particular namespace. Subscriptions are separate objects
but within a namespace they are all synced and resolved together. 

Here's an example of a Subscription definition
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: my-operator
  namespace: openshift-operators 
spec:
  channel: stable
  name: my-operator
  source: redhat-operators 
  sourceNamespace: openshift-marketplace
```

This Subscription object defines the name and namespace of the operator, as well as the catalog from which the operator
data can be found. The channel (such as alpha, beta, or stable) helps determine which version of the operator should be installed
from the catalog source. 

It is also possible to pass configuration information to the operator via the [config](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/design/subscription-config.md) field within a Subscription.
