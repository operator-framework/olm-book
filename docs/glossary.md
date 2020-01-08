# OLM Glossary

A list of OLM terms, with definitions and common Aliases.

## CustomResourceDefinitions (CRDs)

The following [CustomResourceDefinitions](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) are defined by the OLM.

### ClusterServiceVersion

**Definition**: The ClusterServiceVersion represents a particular version of a ClusterService and its operator. It includes metadata such as name, description, version, repository link, labels, icon, etc. It declares `owned`/`required` CRDs, cluster requirements, and install strategy that tells OLM how to create required resources and set up the operator as a [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

**Aliases**: CSV(s)

### CatalogSources

**Definition**: The CatalogSource represents a repository of bundles, which are collections of resources that must contain [CSVs](#ClusterServiceVersion), [CRDs](#CustomResourceDefinitions), and package definitions. There are multiple implementations of a CatalogSource backend, the current recommendation is to use a [registry image](#Index).

**Aliases**: CatSrc(s)

### InstallPlans

**Definition**: The InstallPlan defines a set of resources to be created in order to install or upgrade to a specific version of a ClusterService defined by a CSV.

**Aliases**: IP(s)

### OperatorGroups

**Definition**: The OperatorGroup selects a set of target namespaces in which to generate required RBAC access for its member Operators.

**Aliases**: OG(s)

### OperatorSources

**Definition**: The OperatorSources are a way of pointing to external app registry namespaces that contain a catalog of operators. Applying an OperatorSource to a cluster makes the operators in that OperatorSource available for installation in that cluster.

**Aliases**: OpSrc(s)

### Subscriptions

**Definition**: The Subscription defines the channel and the source of Operator updates and is used to keep the CSV updated by binding with a channel in a package.

**Aliases**: Subs(s)

## OLM Concepts

### Bundle

**Definition**: A collection of Operator [CSV](#ClusterServiceVersion), manifests, and metadata which together form a unique version of an Operator that can be installed onto the cluster. 

### Bundle Image

**Definition**: An image of a bundle is built from operator manifests and contains exactly one [bundle](#Bundle). The bundle images are stored and distributed by OCI spec container registries such as Quay.io or DockerHub.

### Channel

**Definition**: The channel defines a stream of updates for an operator and is used to roll out updates for subscribers. The head points at the latest version of that channel. For example, a stable channel would have all stable versions of an operator arranged from the earliest to the latest. An operator can have several channels, and a subscription binding to a certain channel would only look for updates in that channel.

### Catalog Image 

**Definition**: A catalog image is a containerized datastore that describes a set of operator and update metadata that can be installed onto a cluster via OLM.

**Aliases**: OPM Index

### Dependency

**Definition**: An Operator may have a dependency on another Operator being present in the cluster. For example, the Vault Operator has a dependency on the Etcd Operator for its data persistence layer. OLM resolves these dependencies by ensuring all specified versions of Operators and CRDs are installed on the cluster during the installation phase. This dependency is resolved by finding and installing an Operator in a Catalog that satisfies the required CRD API, and not related to [packages](#Packages)/[bundles](#Bundles).

**Aliases**: Operator Dependency, GVK Dependency, API Dependency, Required CRD

### Index 

**Definition**: The Index refers to an image of a database (a database snapshot) that contains information about Operator bundles including CSVs, CRDs, etc of all versions. This index can host a history of used operators on a cluster and be maintained by adding or removing operators.

**Aliases**: Registry DB, Catalog DB, OPM registry

### Package

**Definition**: A package is a directory that encloses all released history of an Operator with each version contained
 in the bundle format. A released version of an Operator is described in a ClusterServiceVersion manifest alongside the CustomResourceDefinitions.

### Registry

**Definition**: A database which stores (Bundle Images)[#BundleImage] of Operators, each with all of its latest/historical versions in all [channels](#Channel). 

### Upgrade Graph

**Definition**: An upgrade graph links versions of [CSV](#ClusterServiceVersions) together, similar to the upgrade graph of any other packaged software. Operators can be installed sequentially, or certain versions can be skipped. The update graph is expected to grow only at the head with newer versions being added. This is automatically resolved as part of [index](#Index).
