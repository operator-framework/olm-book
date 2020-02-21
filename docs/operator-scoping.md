# Operator Scoping

OLM runs with cluster-admin privileges and is capable of granting permissions to operators that it deploys. By default, an operator can specify any set of permission(s) in the CSV and OLM will consequently grant it to the operator. In effect, an operator can achieve cluster-scoped privilege(s) which may not always be desired.

OLM introduces the concept of `OperatorGroups` to enable cluster admins complete control over the permissions that OLM grants operators that it deploys. An admin may create a single `OperatorGroup` in a given namespace. Any CSV created in that namespace is said to be a member operator of that `OperatorGroup`. With `OperatorGroups`, a cluster admin can:

* Define the set of permissions that OLM may grant to member operators
* Define the set of namespaces that OLM may grant namespaced permissions in.

## Configuring OperatorGroups

### Scoping Member Operators to Specific Namespaces

Using an `OperatorGroup`, a cluster admin can scope member operators' namespaced permissions to specific namespaces in two ways:

* Using a predefined list of namespaces
* Using a label selector.

#### Defining a set of namespaces

The set of namespaces can be hardcoded setting the `spec.targetNamespaces` of an `OperatorGroup` like so:

```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: my-group
  namespace: my-namespace
spec:
  targetNamespaces:
  - my-namespace
  - my-other-namespace
  - my-other-other-namespace
```

In the example above, member operator will be scoped to the following namespaces:

* my-namespace
* my-other-namespace
* my-other-other-namespace

#### Defining a set of namespaces with a label selector

Cluster admins may not know in advance which namespaces member operators should be scoped to. In this case, it may make sense to use a label selector to identify which namespaces permissions should be granted in. A namespace selector can be defined for an `OpearatorGroup` like so:

```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: my-group
  namespace: my-namespace
spec:
  selector:
    cool.io/prod: "true"
```

In the example above, member operators will be scoped to any namespaces with the `cool.io/prod: true` label. If no namespaces exist with the `cool.io/prod: true` label, OLM will fail to install any member operators.

>Note: In the case that both a selector and a list of namespaces are provided, the selector is ignored.

#### TargetNamespaces and their realationship to InstallModes

When creating `OperatorGroups` it is important to keep in mind that an operator may not support all namespace configurations. For example, an operator that is designed to run at the cluster level shouldn't be expected to work in an `OperatorGroup` that defines a single targetNamespace.

Operator authors are responsible for defining which `InstallModes` their operator supports within its `ClusterServiceVersion (CSV)`. There are four `InstallModes` that an operator can support:

* **OwnNamespace**: If supported, the operator can be configured to watch for events in the namespace it is deployed in.
* **SingleNamespace**: If supported, the operator can be configured to watch for events in a single namespace that the operator is not deployed in.
* **MultiNamespace**: If supported, the operator can be configured to watch for events in more than one namespace.
* **AllNamespaces**: If supported, the operator can be configured to watch for events in all namespaces.

>Note: If a CSV's spec omits an entry of InstallModeType, that type is considered unsupported unless support can be inferred by an existing entry that implicitly supports it.

Cluster admins cannot override which `InstallModes` an operator supports, and so should understand how to create an `OperatorGroup` that supports each `InstallMode`. Let's look at an example of an `OperatorGroup` implementing each type of installMode:

##### OwnNamespace

```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: own-namespace-operator-group
  namespace: own-namespace
spec:
  targetNamespaces:
  - own-namespace
```

##### SingleNamespace

```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: single-namespace-operator-group
  namespace: own-namespace
spec:
  targetNamespaces:
  - some-other-namespace
```

##### MultiNamespace

```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: multi-namespace-operator-group
  namespace: own-namespace
spec:
  targetNamespaces:
  - own-namespace
  - some-other-namespace
```

##### AllNamespaces

```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: all-namespaces-operator-group
  namespace: own-namespace
```

### Scoping Member Operator Permissions

When creating an `OperatorGroup`, cluster admins may specify a `ServiceAccount` that defines the set of permissions that may be granted to all member operators. OLM will ensure that when an operator is installed its privileges are confined to that of the `ServiceAccount` specified.

As a result a cluster-admin can limit an operator to a pre-defined set of RBAC rules. The Operator will not be able to do anything that is not explicitly permitted by these permissions. This enables self-sufficient installation of Operators by non-cluster-admin users with a limited scope.

#### Defining a ServiceAccount for an OperatorGroup

A `ServiceAccount` may be defined within an `OperatorGroup` like so:

```yaml
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: scoped-permissions-operator-group
  namespace: own-namespace
spec:
  serviceAccountName: member-operator-servicee-account
  targetNamespaces:
  - own-namespace
```

Any operator tied to this `OperatorGroup` will now be confined to the permission(s) granted to the specified `ServiceAccount`. If the operator asks for permission(s) that are outside the scope of the `ServiceAccount` the install will fail with appropriate error(s).

An example of scoping an operator can be found [here](openshift/coming-soon.md).

### Configuring the End User Experience

### Making Operators Available

Once a cluster admin has successfully created an `OperatorGroup`, he or she then has the opportunity to decide which operators should be offered as a part of that group. This is an important phase in configuring the cluster, as most users may not have the ability to install an operator into an `OperatorGroup`.

A cluster admin can add an operator to an `OperatorGroup` by creating a `Subscription` in the same namespace. An operator can be added and removed from an `OperatorGroup` at anytime.

Keep in mind that this process can be repeated for any number of `OperatorGroups`. This means that a cluster admin can decide for a set of operator to watch for events in one set of namespaces while defining a seperate set of operators that watch for events in a seperate set of namespaces.

### Enabling a User to use an Operator

Once a cluster admin has decided which operators are available on a cluster, it is now time to decide which user(s) may take advantage of an available operator.

Users interact with operators by creating a resource in a namespace that an operator is watching. As such, by tuning RBAC privileges a cluster admin has complete control over the set of users that may interact with an operator along with the set of APIs that user may take advantage of.

For example, if an operator were available that offered two APIs, a cluster admin may provide a user with full RBAC privileges over one API but not grant the user any privileges with the second API.
