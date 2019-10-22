# How do I list Operators available to install?

There is an `APIService` in OLM named `PackageManifest` that contains information existing `CatalogSources`, which is essentially a repository of CSVs, CRDs, and packages that define an application, and their `ConfigMaps` in the cluster. By querying that API, you can see the list of available operators.

There are two different types of `CatalogSource` in OLM: global and namespace `CatalogSource`. The global `CatalogSource` contains operators that will be avaiable for all namespaces while namespace `CatalogSource` only contains operators that are only available for a specific namespace.

## PackageManifest Commands

You can use these example commands via either OpenShift CLI (oc) or kubectl CLI (kubectl) to list available operators in a specific namespace. `PackageManifest` will return the union of global, which are available globally, and namespace operators in namespace you're requesting.

```bash
$ oc get packagemanifest -n <namespace>
```

or

```bash
$ kubectl get packagemanifest -n <namespace>
```

The list of available operators will be displayed as an output of those above commands:

```bash
$ kubectl get packagemanifest
NAME                               CATALOG               AGE
cassandra-operator                 Community Operators   26m
etcd                               Community Operators   26m
postgres-operator                  Community Operators   26m
prometheus                         Community Operators   26m
wildfly                            Community Operators   26m
```
