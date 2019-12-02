# How do I list Operators available to install?

There is an extension API in OLM named `PackageManifest` that contains information about the content served by `CatalogSources`, which is essentially a repository of CSVs, CRDs, and packages that define a operator in the cluster. By querying that API, you can see the list of available operators.

There are two different types of `CatalogSource` in OLM: global and namespaced `CatalogSource`. The global `CatalogSource` contains operators that will be available for all namespaces while namespaced `CatalogSource` only contains operators that are only available for a specific namespace. Therefore the list of `packagemanifests` can differ depending on from which namespace it has been executed.

By default, OLM ships with the namespace `olm` configured the place where global catalogs are stored. The content of these catalogs are visible in all namespaces of the cluster:

```bash
$ kubectl get catalogsources -n olm

NAME                    DISPLAY               TYPE   PUBLISHER        AGE
operatorhubio-catalog   Community Operators   grpc   OperatorHub.io   11m
```

## PackageManifest Commands

You can use these example commands via kubectl CLI (kubectl) to list operators available to install. `PackageManifest` will return the union of global, which are available globally, and namespaced operators in namespace you're requesting.

The list of available operators will be displayed as an output:

```bash
$ kubectl get packagemanifest
NAME                               CATALOG               AGE
cassandra-operator                 Community Operators   26m
etcd                               Community Operators   26m
postgres-operator                  Community Operators   26m
prometheus                         Community Operators   26m
wildfly                            Community Operators   26m
```

If there are no `CatalogSource` objects present in the current namespace of the command above the list will contain all Operators found in global catalogs. To include Operators available to install from namespace catalogs simply supply it with the `kubectl` command:

```bash
$ kubectl get packagemanifests

NAME                               CATALOG               AGE
...
my-operator                        My Catalog            31m
...
```

## Information relevant for installation

From the `packagemanifest` of an Operator you can discover available update channels and latest version like so:

```bash
$ kubectl describe packagemanifest my-operator

Name:         my-operator
Namespace:    default
Labels:       catalog=operatorhubio-catalog
              catalog-namespace=olm
              provider=CNCF
              provider-url=
Annotations:  <none>
API Version:  packages.operators.coreos.com/v1
Kind:         PackageManifest
Metadata:
  Creation Timestamp:  2019-11-19T13:42:18Z
  Self Link:           /apis/packages.operators.coreos.com/v1/namespaces/default/packagemanifests/my-operator
Spec:
Status:
  Catalog Source:               operatorhubio-catalog
  Catalog Source Display Name:  Community Operators
  Catalog Source Namespace:     olm
  Catalog Source Publisher:     OperatorHub.io
  Channels:
    Current CSV:  my-operator.v1.15.0
    [...]
    Name:           stable
  Default Channel:  stable
  Package Name:     jaeger
```

This will also show you information about the catalog object from which this Operator is served. Note that only the latest version in `status.channels.currentCSV` will be shown.