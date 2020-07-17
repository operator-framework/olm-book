# Subscriptions

Subscriptions are Custom Resources that relate an operator to a CatalogSource. Subscriptions describe which channel of an operator package to subscribe to and whether to perform updates automatically or manually. If set to automatic, the Subscription ensures OLM will manage and upgrade the operator to ensure the latest version is always running in the cluster.

Here's an example of a Subscription definition:

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

This Subscription object defines the name and namespace of the operator, as well as the catalog from which the operator data can be found. The channel (such as alpha, beta, or stable) helps determine which stream of the operator should be installed from the CatalogSource.

## Manually Approving Upgrades via Subscriptions

By default, OLM will automatically approve updates to an operator as new versions become available via a CatalogSource. When creating a subscription, it is possible to disable automatic updates by setting the `approval` field to `Manual` like so:

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
  approval: Manual
```

Setting the `approval` field to manual will prevent OLM from automatically installing the operator. As such, you will need to approve the installPlan which can be done with the following commands:

```bash
kubectl -n openshift-operators get installplans
NAME            CSV                                APPROVAL   APPROVED
install-bfmxd   my-operator.v0.1.0                 Manual     false

$ kubectl -n openshift-operators patch installplan install-bfmxd -p '{"spec":{"approved":true}}' --type merge
installplan.operators.coreos.com/install-bfmxd patched

$ kubectl -n openshift-operators get installplans
NAME            CSV                                APPROVAL   APPROVED
install-bfmxd   my-operator.v0.1.0                 Manual     true
```

Now that the `install-bfmxd` installPlan is in the approved state, OLM will install the operator defined by the `my-operator.v0.1.0` CSV.

When the CatalogSource is updated with a newer version of that operator in the channel you selected, a new installPlan will be created in the namespace that you installed the operator to, as shown below:

```bash
$ kubectl -n openshift-operators get installplans
NAME            CSV                                APPROVAL   APPROVED
install-bfmxd   my-operator.v0.1.0                 Manual     true
install-svojy   my-operator.v0.2.0                 Manual     false
```

From here, you can approve `install-svojy` using the patch command shown earlier.

With the new installPlan in the approve state, the `my-operator.v0.2.0` CSV will be deployed to the cluster and if the CSV reaches the `Succeeded` state the old CSV will be deleted. If the new CSV fails to reach the `Succeeded` state, both CSVs will continue to exist and it is up to the user to resolve the failure. In either case, OLM will not delete old installPlans as they act as a record of CSVs that were installed on your cluster.

## How do I know when an update is available for an operator

It is possible to identify when there is a newer version of an operator available by inspecting the status of the operator's subscription. The value associated with the `currentCSV` field is the newest version that is known to OLM, and `installedCSV` is the version that is installed on the cluster.
