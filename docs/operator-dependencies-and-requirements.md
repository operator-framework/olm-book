# Operator Dependency and Requirement Resolution

## Overview on Dependency Principles

OLM manages the dependency resolution and upgrade lifecycle of running operators.

Dependencies within OLM exist when an operator will not function as intended if a second operator is not present on cluster and are often introduced when installing or updating an operator.

OLM does not allow an operator to define a dependency on a specific version (e.g. `etcd-operator v0.9.0`) or instance (e.g. `etcd v3.2.1`) of an operator. Instead, an operator may define a dependency on a specific (Group, Version, Kind) of an API provided by a seperate operator. This encourages operator authors to depend on the interface and not the implementation; thereby allowing operators to update on individual release cycles.

If a dependency is ever discovered, OLM will attempt to resolve said dependency by searching through known `CatalogSources` for an operator that `owns` the API. If an operator is found that `owns`the `required` API, OLM will attempt to deploy the operator that `owns` the API. If no `owner` is found, OLM will fail to deploy the operator with the dependency.

### Similarity with Package Managers

OLM manages operator dependency resolution very similarily to OS package managers, such as apt/dkpg and yum/rpm. However, given that operators are always running, OLM attempts to ensure that it never deploys a combination of operators that do not work with each other.

This means that OLM will never:

* install a set of operators that require APIs that can't be provided
* update an operator in a way that breaks another that depends upon it

### OLM Dependency Resolution Specifics

Dependency resolution begins when a `Subscription` is reconciled by OLM.

When resolving a `Subscription`, OLM will look at all `Subscription` in the namespace and identify the next offering of each operator based on the upgrade graph. By updating all `Subscriptions` in the namespace, OLM avoids version deadlock that could be introduced when two operators have dependencies on each other.

Once OLM has identified the list of operator that should be installed in the namespace, OLM will attempt to resolve any required APIs that are missing from the cluster by querying known `CatalogSources`.

Rather than returning the first `CatalogSource` that contains the missing API, OLM will attempt to identify a  prioritized `CatalogSource` - one that provided an operator that depends on the missing API. If the prioritized `CatalogSource` does not contain the API, OLM will search through the remaining `CatalogSources` in the namespace for the API. If an operator is found that provides the API, OLM will create a `Subscription` for the operator using that `CatalogSource`.

If the required API cannot be resolved, OLM will not install operators that rely on that API.

## Defining a Dependency

An operator can define dependencies within its `ClusterServiceVersion (CSV)``. An operator can specify:

* A `required` CRD by adding it to the `spec.customresourcedefinitions.required` field.
* A `required` API Service by adding it to the `spec.apiservicedefinitions.required` field.

> Note: If your operator defines a dependency, packaging it in the same `CatalogSource` as the operator that fulfills the dependency ensures that OLM will always resolve the dependency.

## Defining Ownership over an API

An operator can define which APIs it owns within its CSV. An operator can specify:

* An `owned` CRD by adding it to the `spec.customresourcedefinitions.owned` field.
* An `owned` API Service by adding it to the `spec.apiservicedefinitions.owned` field.
