# Operator Dependency and Requirement Resolution

## Overview on Dependency Principles

OLM manages the dependency resolution and upgrade lifecycle of running operators.

Dependencies within OLM exist when an operator will not function as intended if a second operator is not present on cluster and are often introduced when installing or updating an operator.

OLM does not allow an operator to define a dependency on a specific version (e.g. `etcd-operator v0.9.0`) or instance (e.g. `etcd v3.2.1`) of an operator. Instead, an operator may define a dependency on a specific (Group, Version, Kind) of an API provided by a seperate operator. This encourages operator authors to depend on the interface and not the implementation; thereby allowing operators to update on individual release cycles.

If a dependency is ever discovered, OLM will attempt to resolve said dependency by searching through known `CatalogSources` for an operator that `owns` the API. If an operator is found that `owns` the `required` API, OLM will attempt to deploy the operator that `owns` the API. If no `owner` is found, OLM will fail to deploy the operator with the dependency.

### Similarity with Package Managers

OLM manages operator dependency resolution very similarily to OS package managers, such as apt/dkpg and yum/rpm. However, given that operators are always running, OLM attempts to ensure that it never deploys a combination of operators that do not work with each other.

This means that OLM will never:

* install a set of operators that require APIs that can't be provided
* update an operator in a way that breaks another that depends upon it

### OLM Dependency Resolution Specifics

Dependency resolution begins when a `Subscription` is reconciled by OLM.

When resolving a `Subscription`, OLM will look at the Operator metadata (specified in its `ClusterServiceVersion`) in order to determine required APIs either from the cluster itself or other Operators (via CRDs or APIServices). Once OLM has identified the list of Operators that should be installed it will create `Subscriptions` for those in the same namespace.

Throughout the existence of a `Subscription`, OLM will look at all `Subscription` objects in the namespace and detect available updates of each operators. By looking at all `Subscriptions` in the namespace at once, OLM avoids version deadlock that could be introduced when two operators have dependencies on each other.

In case multiple Operators serve a required API, rather than using the first match, OLM searches `CatalogSources` in a specific order. First is the same catalog that contains the Operator stating the dependency, followed by other catalogs in the same namespace as the required `Subscription` and then finally all remaining `CatalogSources` objects in other namespaces of the cluster.

If are required API cannot be resolved or found, OLM will not install operators that rely on that API.

## Defining a Dependency

An operator can define dependencies within its `ClusterServiceVersion (CSV)``. An operator can specify:

* A `required` CRD by adding it to the `spec.customresourcedefinitions.required` list.
* A `required` API Service by adding it to the `spec.apiservicedefinitions.required` list.
* A `required` Cluster-provided API by adding it to the `spec.nativeapis` field

> Note: If your operator defines a dependency, packaging it in the same `CatalogSource` as the operator that fulfills the dependency ensures that OLM will always resolve the dependency.

## Defining Ownership over an API

An operator can define which APIs it owns within its CSV. An operator can specify:

* An `owned` CRD by adding it to the `spec.customresourcedefinitions.owned` field.
* An `owned` API Service by adding it to the `spec.apiservicedefinitions.owned` field.
