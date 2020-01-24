# How do I package an Operator for OLM?

OLM requires you to provide metadata about your operator in order to ensure that it can be kept running safely on a cluster, and to provide information about how updates should be applied as you publish new versions of your operator.

This is very similar to packaging software for a traditional operating system - think of the packaging step for OLM as the stage at which you make your `rpm`, `dep`, or `apk` bundle.

## Writing your Operator Manifests

OLM uses an api called `ClusterServiceVersion` (CSV) to describe a single instance of a version of an operator. This is the main entrypoint for packaging an operator for OLM.

There are two important ways to think about the CSV:

 1. Like an `rpm` or `deb`, it collects metadata about the operator that is required to install it onto the cluster.
 2. Like a `Deployment` that can stamp out `Pod`s from a template, the `ClusterServiceVersion` describes a template for the operator `Deployment` and can stamp them out.

This is all in service of ensuring that when a user installs an operator from OLM, they can understand what changes are happening to the cluster, and OLM can ensure that installing the operator is a safe operation.

### Starting from an existing set of operator manifests

For this example, we'll use the example manifests from [the example memcached operator](https://github.com/operator-framework/operator-sdk-samples/tree/master/go/memcached-operator/deploy).

These manifests consist of:

- **CRDs** that define the APIs your operator will manage
- **Operator** (`operator.yaml`), containing the`Deployment` that runs your operator pods
- **RBAC** (`role.yaml`, `role_binding.yaml`, `service_account.yaml`) that configures the service account permissions your operator requires.

Building a minimal `ClusterServiceVersion` from these requires transplanting the contents of the Operator definition and the RBAC definitions into a CSV. Together, your CSV and CRDs will form the package that you give to OLM to install an operator.

#### Basic Metadata (Optional)

Let's start with a CSV that only contains some descriptive metadata:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  annotations:
  name: memcached-operator.v0.10.0
spec:
  description: This is an operator for memcached.
  displayName: Memcached Operator
  keywords:
  - memcached
  - app
  maintainers:
  - email: corp@example.com
    name: Some Corp
  maturity: alpha
  provider:
    name: Example
    url: www.example.com
  version: 0.10.0
```

Most of these fields are optional, but they provide an opportunity to describe your operator to potential or current users.

#### Installation Metadata (Required)

The next section to add to the CSV is the Install Strategy - this tells OLM about the runtime components of your operator and their requirements.

Here is an example of the basic structure of an install strategy:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  annotations:
  name: memcached-operator.v0.10.0
spec:
  install:
    # strategy indicates what type of deployment artifacts are used
    strategy: deployment
    # spec for the deployment strategy is a list of deployment specs and required permissions - similar to a pod template used in a deployment
    spec:
      permissions:
      - serviceAccountName: memcached-operator
        rules:
        - apiGroups:
          - ""
          resources:
          - pods
          verbs:
          - '*'
          # the rest of the rules
      # permissions required at the cluster scope
      clusterPermissions:
      - serviceAccountName: memcached-operator
        rules:
        - apiGroups:
          - ""
          resources:
          - serviceaccounts
          verbs:
          - '*'
          # the rest of the rules
      deployments:
      - name: memcached-operator
        spec:
          replicas: 1
          # the rest of a deployment spec
```

`deployments` is an array - your operator may be composed of several seperate components that should all be deployed and versioned together.

It's also important to tell OLM the ways in which your operator can be deployed, or its `installModes`. InstallModes indicate if your operator can be configured to watch, one, some, or all namespaces. Please see the [document on install modes]() and `OperatorGroups` for more information.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: memcached-operator.v0.10.0
spec:
  # ...
  installModes:
  - supported: true
    type: OwnNamespace
  - supported: true
    type: SingleNamespace
  - supported: false
    type: MultiNamespace
  - supported: true
    type: AllNamespaces
```

**Using `faq` to build an install strategy from an existing deployment and rbac**

`faq` is a wrapper around `jq` that can handle multiple input and output formats, like the yaml we're working with now. The following example requires that [faq be installed](https://github.com/jzelinskie/faq#installation) and references [the example memcached operator](https://github.com/operator-framework/operator-sdk-samples/tree/master/go/memcached-operator/deploy).

Here is a simple `faq` script that can generate an install strategy from a single deployment:

```sh
$ faq -f yaml  '{install: {strategy: "deployment", spec:{ deployments: [{name: .metadata.name, template: .spec }] }}}' operator.yaml
```

If you have an existing CSV `csv.yaml` (refer to the example from Basic Metadata) and you'd like to insert or update an install strategy from a deployment `operator.yaml`, a role `role.yaml`, and a service account `service_account.yaml`, that is also possible:

```sh
$ faq -f yaml -o yaml --slurp '.[0].spec.install = {strategy: "deployment", spec:{ deployments: [{name: .[1].metadata.name, template: .[1].spec }], permissions: [{serviceAccountName: .[3].metadata.name, rules: .[2].rules }]}} | .[0]' csv.yaml operator.yaml role.yaml service_account.yaml
```

#### Defining APIs (Required)

By definition, operators are programs that can talk to the Kubernetes API. Often, they are also programs that *extend* the Kubernetes API, by providing an interface via `CustomResourceDefinition`s or, less frequently, `APIService`s.

##### Owned APIs

Exactly which APIs are used and which APIs are watched or owned is important metadata for OLM. OLM uses this information to determine if dependencies are met and ensure that no two operators fight over the same resources in a cluster.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: memcached-operator.v0.10.0
spec:
  # ...
  customresourcedefinitions:
    owned:
    # a list of CRDs that this operator owns
      # name is the metadata.name of the CRD
    - name: cache.example.com
      # version is the version of the CRD (one per entry)
      version: v1alpha1
      # spec.names.kind from the CRD
      kind: Memcached
```

##### Required APIs

Similarly, there is a section `spec.customresourcedefinitions.required`, where dependencies can be specified. The operators that provide those APIs will be discovered and installed by OLM if they have not been installed.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: other-operator.v1.0
spec:
  # ...
  customresourcedefinitions:
    required:
    # a list of CRDs that this operator requires
    - name: cache.example.com
      version: v1alpha1
      kind: Memcached
```

Dependency resolution and ownership is discussed more in depth in our [registry documentation]().

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: memcached-operator.v0.10.0
spec:
  # ...
  customresourcedefinitions:
    owned:
    # a list of CRDs that this operator owns
      # name is the metadata.name of the CRD
    - name: cache.example.com
      # version is the version of the CRD (one per entry)
      version: v1alpha1
      # spec.names.kind from the CRD
      kind: Memcached
```

##### NativeAPIs (recommended)

There are often cases where you wish to depend on an API that is either provided natively by the platform (i.e. `Pod`) or sometimes by another operator that is outside the control of OLM.

In those cases, those dependencies can be described in the CSV as well, via `nativeAPIs`

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: other-operator.v1.0
spec:
  nativeAPIs:
  - group: ""
    version: v1
    kind: Pod
```

The absence of any required `nativeAPIs` from a cluster will pause the installation of the operator, and `OLM` will write a status into the `CSV` indicating the missing APIs.

TODO: example status

`nativeAPIs` is an optional field, but the more information you give OLM about the context in which your operator should be run, the more informed decisions OLM can make.

##### Extension apiservers and APIServices

Please see the document on [extension apiservers]() if your operator does not rely on CRDs to provide its API.

#### Operator SDK

TODO: link to SDK csv generation

#### Advanced and Optional features

Please see the documentation for [advanced operator configuration]() which includes additional suggestions for further integration with OLM.
