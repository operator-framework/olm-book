# Troubleshooting

This document focuses on troubleshooting unexpected behavior when installing and managing operators with OLM. This document contains 3 sections where we highlight how to troubleshoot specific OLM components:

* [Troubleshooting the `CatalogSource`](#catalogsource-troubleshooting)
* [Troubleshooting the `Subscription`](#subscription-troubleshooting)
* [Troubleshooting the `ClusterServiceVersion (CSV)`](#clusterserviceversion-troubleshooting)

## Prereqs

Some of the commands listed below assume that you have [yq](https://github.com/mikefarah/yq) installed on your system. While `yq` is not required, it is a useful tool when parsing yaml. You can install `yq` by following the [official installation steps](https://github.com/mikefarah/yq#install).

## CatalogSource Troubleshooting

### How to debug a failing CatalogSource

The Catalog operator will constantly update the `Status` of `CatalogSources` to reflect its current state. You can check the `Status` of your `CatalogSource` with the following command:

`$ kubectl -n my-namespace get catsrc my-catalog -o yaml | yq r - status`

>Note: It is possible that the `Status` is missing, which suggests that the Catalog operator is encountering an issue when processing the `CatalogSource` in a very early stage.

If the `Status` block does not provide enough information, check the [Catalog operator's logs](#how-to-view-the-catalog-operator-logs).

If you are still unable to identify the cause of the failure, check if a pod was created for the `CatalogSource`. If a pod exists, review the pod's yaml and logs:

```bash
$ kubectl -n my-namespace get pods
NAME                                READY   STATUS    RESTARTS   AGE
my-catalog-ltdlp         1/1     Running   0          8m31s

$ kubectl -n my-namespace get pod my-catalog-ltdlp -o yaml
...

$ kubectl -n my-namespace logs my-catalog-ltdlp
...
```

### I'm not sure if a specific version of an operator is available in a CatalogSource

First verify that the `CatalogSource` contains the operator that you want to install:

```bash
$ kubectl -n my-namespace get packagemanifests
NAME                               CATALOG             AGE
...
portworx                           My Catalog Source   14m
postgres-operator                  My Catalog Source   14m
postgresql                         My Catalog Source   14m
postgresql-operator-dev4devs-com   My Catalog Source   14m
prometheus                         My Catalog Source   14m
...
```

If the operator is present, check if the version you want is available:

`$ kubectl -n my-namespace get packagemanifests my-operator -o yaml`

### My CatalogSource cannot pull images from a private registry

If you are attempting to pull images from a private registry, make sure to specify a secret key in the `CatalogSource.Spec.Secrets` field.

## Subscription Troubleshooting

This section assumes that you have a working `CatalogSource`.

### How to debug a failing Subscription

The Catalog operator will constantly update the `Status` of `Subscription` to reflect its current state. You can check the `Status` of your `Subscription` with the following command:

`$ kubectl -n my-namespace get subscriptions my-subscription -o yaml | yq r - status`

>Note: It is possible that the `Status` is missing, which suggests that the Catalog operator is encountering an issue when processing the `Subscription` in a very early stage.

If the `Status` block does not provide enough information, check the [Catalog operator's logs](#how-to-view-the-catalog-operator-logs).

### A subscription in namespace X can't install operators from a CatalogSource in namespace Y

`Subscriptions` cannot install operators provided by `CatalogSources` that are not in the same namespace unless the `CatalogSource` is created in the `olm` namespace.

### Why does a single failing subscription cause all subscriptions in a namespace to fail?

Each Subscription in a namespace acts as a part of a set of operators for the namespace - think of a Subscription as an entry in a python `requirements.txt`. If OLM is unable to resolve part of the set, it knows that resolving the entire set will fail, so it will bail out of the installation of operators for that particular namespace. Subscriptions are separate objects but within a namespace they are all synced and resolved together.

## ClusterServiceVersion Troubleshooting

### How to debug a failing CSV

If the OLM operator encounters an unrecoverable error when attempting to install the operator the `CSV` will be placed in the `failed` phase. The OLM operator will constantly update the `Status` with useful information regarding the state of the `CSV`. You can check the `Status` of your `CSV` with the following command:

`$ kubectl -n my-catalogsource-namespace get csv prometheusoperator.0.32.0 -o yaml | yq r - status`

>Note: It is possible that the Status is missing, which suggests that the OLM operator is encountering an issue when processing the `CSV` in a very early stage. You should respond by reviewing the logs of the OLM operator.

You should typically pay special attention to the information within the `status.reason` and `status.message` fields. Please look in the [failed CSV reasons](#failed-csv-reasons)

If the `Status` block does not provide enough information, check the [OLM operator's logs](#how-to-view-the-olm-operator-logs).

### Failed CSV Reasons

#### Reason: NoOperatorGroup

The `CSV` failed to install because it has been deployed in a namespace that does not include an `OperatorGroup`. For more information about `OperatorGroups` see [operator-scoping.md](operator-scoping.md).

#### Reason: UnsupportedOperatorGroup

The `CSV` is failing to install because it does not support he `OperatorGroup` defined in the namespace. For more information about `OperatorGroups` see [operator-scoping.md](operator-scoping.md).

### Failed CSV Messages

#### Messages Ending with "field is immutable"

The `CSV` is failing because its install strategy changes some immutable field of an existing `Deployment`. This usually happens on upgrade, after an operator author publishes a new version of the operator containing such a change. In this case, the issue can be resolved by publishing a new version of the operator that uses a different `Deployment` name, which will cause OLM to generate a completely new `Deployment` instead of attempting to patch any existing one.

## Debugging the OLM or Catalog operators

### How to enable verbose logging on the OLM and Catalog operators

Both the OLM and Catalog operators have `-debug` flags available that display much more useful information when diagnosing a problem. If necessary, add this flag to their deployments and perform the action that is showing undersired behavior.

### How to view the Catalog operator logs

To view the Catalog Operator logs, use the following commands:

```bash
$ kubectl -n olm get pods
NAME                                READY   STATUS    RESTARTS   AGE
catalog-operator-5bdc79c56b-zbqbl   1/1     Running   0          5m30s
olm-operator-6999db5767-5r5zs       1/1     Running   0          5m31s
operatorhubio-catalog-ltdlp         1/1     Running   0          5m28s
packageserver-5c76df75bb-mq4qd      1/1     Running   0          5m26s

$ kubectl -n olm logs catalog-operator-5bdc79c56b-zbqbl
...
```

### How to view the OLM operator logs

To view the OLM Operator logs, use the following commands:

```bash
$ kubectl -n olm get pods
NAME                                READY   STATUS    RESTARTS   AGE
catalog-operator-5bdc79c56b-zbqbl   1/1     Running   0          5m30s
olm-operator-6999db5767-5r5zs       1/1     Running   0          5m31s
operatorhubio-catalog-ltdlp         1/1     Running   0          5m28s
packageserver-5c76df75bb-mq4qd      1/1     Running   0          5m26s

$ kubectl -n olm logs olm-operator-6999db5767-5r5zs
...
```
