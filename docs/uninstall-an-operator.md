# How to uninstall an Operator?

In order to uninstall an operator, you need to delete the following resources:

- Subscription
- ClusterServiceVersion (CSV)

Both `Subscription` and `ClusterServiceVersion` are namespace objects meaning you need to delete a `Subscription` and a `CSV` in a specific namespace where you install the operator into.

## Delete a Subscription

If you wish to look up a list of `Subscription` in a specific namespace to see which `Subscription` you want to delete, you can use the following `kubectl` command:
```bash
$ kubectl get subscription -n <namespace>
```

You can delete the `Subscription` in the namespace that the operator was installed into using this command:
```bash
$ kubectl delete subscription <subscription-name> -n <namespace>
```

## Delete a ClusterServiceVersion (CSV)

If you wish to look up a list of `ClusterServiceVersion` in a specific namespace to see which `ClusterServiceVersion` you need to delete, you can use the example `kubectl` command:

```bash
$ kubectl get clusterserviceversion -n <namespace>
```

You can delete the `ClusterServiceVersion` in the namespace that the operator was installed into using this command:

```bash
$ kubectl delete clusterserviceversion <csv-name> -n <namespace>
```

By deleting `ClusterServiceVersion`, it will delete the operator's resources that OLM created for the operator such as deployment, pod(s), RBAC, and others. This also deletes any corresponding CSVs that OLM "Copied" into other namespaces watched by the operator.
