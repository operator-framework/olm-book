# When do I need to update my operator 

When you install an operator from a catalog such as [quay.io/operator-framework/upstream-community-operator](https://quay.io/repository/operator-framework/upstream-community-operators), updated versions of one of more operators are pushed to the container image by rebuilding the catalog. If you have an older image of the catalog in your cluster, you can get the updates to your operators by fetching the latest release of the catalog's container image.

If the image used to build the `Catalogsource` uses a versioned tag, update the tag version of the image to fetch updates to operators in the `Catalogsource`.

For example:

```
$ oc get catsrc operatorhubio-catalog -n olm -o yaml | grep image:
    
    image: quay.io/operator-framework/upstream-community-operators:0.0.1

$ kubectl patch catsrc operatorhubio-catalog -n olm --type=merge -p '{"spec": {"image": "quay.io/operator-framework/upstream-community-operators:0.0.2"}}'

```

If the image used to build the `Catalogsource` uses the `latest` tag, simply delete the pod corresponding to the `CatalogSource`. When the pod is recreated, it will be recreated with the latest image of the catalog, which will contain updates to the operators in that catalog.

For example:

```
$ kubectl delete pods -n olm -l olm.catalogSource=operatorhubio-catalog

```
The operators that were installed from the catalog will be updated automatically or manually, depending on the value of `installPlanApproval` in the Subscription for the operator. For more information on approving manual updates to operators, please see [How do I approve an update?](docs/)   