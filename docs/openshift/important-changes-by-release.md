# Important Changes by OCP Release

## 4.2

### New Global Catalog Namespace

The default _GCN_ (global catalog namespace) has been changed from OLM's namespace, `openshift-operator-lifecycle-manager`, to `operator-marketplace`. As a quick refresher, the GCN is a special namespace that contains operator catalogs that can be subscribed to from any namespace in the cluster. The important positive implication of this change is that default catalogs in the `operator-marketplace` namespace (`redhat-operators`, `certified-operators`, etc...) can be subscribed to from all namespaces without further configuration. The negative implication is that any subscriptions to catalogs in the `openshift-operator-lifecycle-manager` namespace must be co-located in that namespace.
