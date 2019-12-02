# How do I install my operator with OLM?

[Once you've made your packaged operator available in a catalog](packaging-an-operator.md) it will appear in the `packagemanifest` list from which the Operators to install are selected. See [How do I list available Operators](list-available-operators.md#information-relevant-for-installation) how to retrieve the required information from the `PackageServer` in order start an installation of an Operator.

## Install with automatic updates enabled

First, retrieve the package name, channel and catalog name and catalog source namespace from the `packagemanifest` of the desired Operator to install.

```bash
$ kubectl describe packagemanifest my-operator
```

 With this information a `Subscription` object can be created. This represents the intent to install an Operator from a particular catalog and keep it updated throughout the life cycle of the Operator via newly released version that got subsequently added to the referenced catalog.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: <name-of-your-subscription>
  namespace: <namespace-you-want-your-operator-installed-in>
spec:
  channel: <channel-you-want-to-subscribe-to>
  name: <name-of-your-operator>
  source: <name-of-catalog-operator-is-part-of>
  sourceNamespace: <namespace-that-has-catalog>
 ```

The `spec.channel` property can also be omitted in which case the default channel will be picked. Optionally you can also define `spec.startingCSV` to denote that you want to install a specific version of your Operator and not the latest.

For example, if you want to install an operator named `my-operator`, from a catalog named `my-catalog` that is in the namespace `olm`, and you want to subscribe to the channel `stable`, your subscription yaml would look like this:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: subs-to-my-operator
  namespace: olm
spec:
  channel: stable
  name: my-operator
  source: my-catalog
  sourceNamespace: olm
 ``` 

Once you have the YAML, `kubectl apply -f subscription.yaml` to install your operator. 

You can inspect your `Subscription` with `kubectl describe subs <name-of-your-subscription> -n <namespace>`.

```yaml
k describe subs subs-to-my-operator -n olm 
Name:         subs-to-my-operator
Namespace:    olm
Labels:       <none>
Annotations:  <none>
API Version:  operators.coreos.com/v1alpha1
Kind:         Subscription
Metadata:
  Creation Timestamp:  2019-11-19T13:54:51Z
  Generation:          1
  Resource Version:    1787
  Self Link:           /apis/operators.coreos.com/v1alpha1/namespaces/operators/subscriptions/subs-to-my-operator
  UID:                 2864cc19-0ad4-11ea-8c0e-aa8cc6bce8d3
Spec:
  Channel:           stable
  Name:              my-operator
  Source:            my-catalog
  Source Namespace:  olm
Status:
  Catalog Health:
    Catalog Source Ref:
      API Version:       operators.coreos.com/v1alpha1
      Kind:              CatalogSource
      Name:              my-catalog
      Namespace:         olm
      Resource Version:  787
      UID:               674c52e3-0ad2-11ea-8c0e-aa8cc6bce8d3
    Healthy:             true
    Last Updated:        2019-11-19T13:54:51Z
  Conditions:
    Last Transition Time:  2019-11-19T13:54:51Z
    Message:               all available catalogsources are healthy
    Reason:                AllCatalogSourcesHealthy
    Status:                False
    Type:                  CatalogSourcesUnhealthy
  Current CSV:             my-operator.v1.15.0
  Install Plan Ref:
    API Version:       operators.coreos.com/v1alpha1
    Kind:              InstallPlan
    Name:              install-4xhbl
    Namespace:         operators
    Resource Version:  1744
    UID:               286e5546-0ad4-11ea-8c0e-aa8cc6bce8d3
  Installed CSV:       my-operator.v1.15.0
  Installplan:
    API Version:  operators.coreos.com/v1alpha1
    Kind:         InstallPlan
    Name:         install-4xhbl
    Uuid:         286e5546-0ad4-11ea-8c0e-aa8cc6bce8d3
  Last Updated:   2019-11-19T13:54:55Z
  State:          AtLatestKnown
Events:           <none>
```

The `Subscription` has successfully deployed the Operator when the `status.state` property reaches `AtLatestKnown`.

You can verify that the Operator is deployed by checking the `ClusterServiceVersion` (CSV) object in the same namespace where the `Subscription` was placed in. The name of the CSV is referenced in `status.installedCSV` of the `Subscription`:

```bash
$ kubectl get csv my-operator.v1.15.0 -n olm

NAME                  DISPLAY          VERSION           REPLACES              PHASE
my-operator.v1.15.0   My Operator      1.15.0            my-operator.v1.14.0   Succeeded
```

If the the CSV has transitioned to the _Succeeded_ phase the Operator deployment was successful. You inspect it manually like so:

```bash
$ kubectl get deployments -n olm

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
my-operator             1/1     1            1           9m48s
```

If the ClusterServiceVersion fails to show up or does not reach the `Succeeded` phase, please check the [troubleshooting documentation](troubleshooting.md) to debug your installation.

## Install with manual updates

It is also possible to create a `Subscription` that will not automatically install updates as soon as they appear in the catalog. For this the `spec.installPlanApproval` property can be set to `Manual` instead of the default `Automatic`:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: subs-to-my-operator
  namespace: olm
spec:
  channel: stable
  name: my-operator
  source: my-catalog
  sourceNamespace: olm
  installPlanApproval: Manual
```

The `Subscription` will not immediately cause the Operator to be deploy by creating a CSV this time. Instead it will remain in a pending state:

```bash
$ kubectl get subscriptions subs-to-my-operator -n operators -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,STATUS:.status.state,INSTALLPLAN:.status.installPlanRef.name

NAME          NAMESPACE   STATUS           INSTALLPLAN
my-operator   olm         UpgradePending   install-q4fmf
```

An `InstallPlan` object has been created as a result of the `Subscription` getting processed. This also happens when no approval method is selected but it now presents an opportunity to manually approve the Operator installation.

```bash
$ kubectl get installplan install-q4fmf -n olm

NAME           CSV                       APPROVAL   APPROVED
install-q4fmf  my-operator.v1.15.0       Manual     false
```

You can also inspect this objects `status.plan` section to get a preview of the resources that will be deloyed as part of this Operator.

You can then either interactively set the `spec.approved` property of the `InstallPlan` to `true` or _patch_ the object like so:

```bash
kubectl patch installplan install-q4fmf -n olm --type merge  -p '{"spec":{"approved":true}}'
installplan.operators.coreos.com/install-q4fmf patched
```

Now the `InstallPlan` will be processed by OLM, creating the objects that ship as part of the Operator and eventually create the `ClusterServiceVersion` to represent the deployed Operator:

```bash
$ kubectl get csv -n olm

NAME                  DISPLAY          VERSION           REPLACES              PHASE
my-operator.v1.15.0   My Operator      1.15.0            my-operator.v1.14.0   Succeeded
```

If the the CSV has transitioned to the  _Succeeded_ phase the Operator deployment was successful. You inspect it manually like so:

```bash
$ kubectl get deployments -n olm

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
my-operator             1/1     1            1           9m48s
```