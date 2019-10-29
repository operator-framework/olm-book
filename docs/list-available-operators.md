# How do I list Operators available to install?

There is an extension API in OLM named `PackageManifest` that contains information about existing `CatalogSources`, which is essentially a repository of CSVs, CRDs, and packages that define an operator in the cluster. By querying that API, you can see the list of available operators.

There are two different types of `CatalogSource` in OLM: global and namespaced `CatalogSource`. The global `CatalogSource` contains operators that will be available for all namespaces while namespaced `CatalogSource` only contains operators that are only available for a specific namespace.

## PackageManifest Commands

You can use these example commands via kubectl CLI (kubectl) to list available operators in a specific namespace. `PackageManifest` will return the union of global, which are available globally, and namespaced operators in namespace you're requesting.
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
