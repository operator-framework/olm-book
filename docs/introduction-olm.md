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

OLM requires you to provide metadata about your operator in order to ensure that it can be kept running safely on a cluster, and to provide information about how updates should be applied as you publish new versions of your operator. This is very similar to packaging software for a traditional operating system.

## [Package Validation](https://github.com/operator-framework/olm-book/blob/5ee1f7c70286939a03304e49c80eb600364f31f0/docs/validate-package.md)

Once you've [created your operator's package manifests](packaging-an-operator.md), you will want to ensure that your package is valid and in the correct format. To ensure this, you should take several steps to ensure that your package can be used to install your operator in OLM.

## [Publish Package into catalog](https://github.com/operator-framework/olm-book/blob/5ee1f7c70286939a03304e49c80eb600364f31f0/docs/validate-package.md#add-your-package-to-a-catalog)

We will publish the package into a catalog, install that catalog onto a Kube cluster, and then install the operator onto that cluster. If all of that succeeds and your operator is behaving as expected, your package is valid.

## [Subscriptions](https://github.com/operator-framework/olm-book/blob/5ee1f7c70286939a03304e49c80eb600364f31f0/docs/subscriptions.md)

Subscriptions are Custom Resources that relate an operator to a CatalogSource. Subscriptions describe which channel of an operator package to subscribe to and whether to perform updates automatically or manually. If set to automatic, the Subscription ensures OLM will manage and upgrade the operator to ensure the latest version is always running in the cluster.

## [Install Operator](https://github.com/operator-framework/olm-book/blob/5ee1f7c70286939a03304e49c80eb600364f31f0/docs/how-do-i-install-my-operator-with-olm.md)

[Once you've made your operator available in a catalog](openshift/coming-soon.md), [or you've chosen an operator from an existing catalog](openshift/coming-soon.md), you can install your operator by creating a Subscription to a specific channel. 

## [Uninstall Operator](https://github.com/operator-framework/olm-book/blob/5ee1f7c70286939a03304e49c80eb600364f31f0/docs/uninstall-an-operator.md)

Delete subscriptions and ClusterServiceVersion. Both `Subscription` and `CSV` are namespaced objects meaning you need to delete a `Subscription` and a `CSV` in a specific namespace where you install the operator into. 

By deleting `ClusterServiceVersion`, it will delete the operator's resources that OLM created for the operator such as deployment, pod(s), RBAC, and others.

# Link to Getting Started and Advanced doc links 

- [How do I install OLM?](docs/install-olm.md)
- [How do I validate the package?](docs/validate-package.md)
- [How do I install my operator with OLM?](docs/how-do-i-install-my-operator-with-olm.md)
- [How do I uninstall an Operator?](docs/uninstall-an-operator.md)



