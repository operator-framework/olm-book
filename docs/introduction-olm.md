# OLM - Operator Lifecycle Manager

OLM is an open source toolkit to manage Kubernetes native applications, called Operators, in an effective, automated, and scalable way.

OLM extends Kubernetes to provide a declarative way to install, manage, and upgrade Operators and their dependencies in a cluster. 

# How OLM solved the cluster admin and developer needs.

## Operator Installation and Management

As there are multiple steps involved in deploying an Operator, including creating the deployment, adding the custom resource definitions, and configuring the necessary permissions, a management layer becomes necessary to facilitate the process.

Operator Lifecycle Manager (OLM) fulfills this role by introducing a packaging mechanism for delivering Operators and the necessary metadata for visualizing them in compatible UIs, including installation instructions and API hints in the form of CRD descriptors.

## Dependency Resolution

OLM manages the dependency resolution and upgrade lifecycle of running operators. 

If a dependency is ever discovered, OLM will attempt to resolve said dependency by searching through known `CatalogSources` for an operator that `owns` the API. If an operator is found that `owns`the `required` API, OLM will attempt to deploy the operator that `owns` the API. If no `owner` is found, OLM will fail to deploy the operator with the dependency.

## Work similar to package Managers

OLM verifies that all require APIs are available for the operator. If it is not available, then it will not install the operator.

OLM will never update an operator in a way that breaks another that depends upon it.

## Operator Groups

OLM introduces the concept of `OperatorGroups` to enable cluster admins complete control over the permissions that OLM grants operators that it deploys.

With `OperatorGroups`, a cluster admin can:

* Define the set of permissions that OLM may grant to member operators
* Define the set of namespaces that OLM may grant namespaced permissions in.

# Building blocks of OLM

## [Install OLM](https://github.com/operator-framework/olm-book/blob/5ee1f7c70286939a03304e49c80eb600364f31f0/docs/install-olm.md)


