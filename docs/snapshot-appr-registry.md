# How do I snapshot a Quay Appregistry operator catalog?

## What in tarnation is a _snapshot_?

For the purposes of this document, a _snapshot_ is:

- a point-in-time export of an Appregistry (appr) type catalog's content
- the result of converting an appr catalog to a container image type catalog
- an immutable artifact

## When/Why would I need a snapshot?

OLM always installs from the latest version of an appr catalog. As appr catalogs are updated, the latest versions of operators change, and older versions may be removed or altered. This behavior can cause problems maintaining reproducible installs over time.

As of OCP 4.3, Red Hat provided operators are distributed via appr catalogs. Creating a snapshot provides a simple way to use this content without incurring the aforementioned issues.

## Prerequisites

- Linux
- A version `oc` that has the `oc catalog build`, such as this one for OCP `4.3`: https://openshift-release-artifacts.svc.ci.openshift.org/4.3.0-0.nightly-2019-11-13-233341/openshift-client-linux-4.3.0-0.nightly-2019-11-13-233341.tar.gz
- A container image registry that supports [Docker v2-2](https://docs.docker.com/registry/spec/manifest-v2-2/)
- [grpcurl](https://github.com/fullstorydev/grpcurl) (optional, for testing)

## Setup

Users should authenticate with the target image registry. By default, `oc adm catalog build` uses `~/.docker/config.json` to determine credentials. 

## Taking a snapshot

Snapshot `redhat-operators` as `quay.io/my/redhat-operators:snapshot-0`.

```sh
$ oc adm catalog build --appregistry-endpoint https://quay.io/cnr --appregistry-org redhat-operators --to=quay.io/my/redhat-operators:snapshot-0
INFO[0013] loading Bundles                               dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605
INFO[0013] directory                                     dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605 file=manifests-829192605 load=bundles
INFO[0013] directory                                     dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605 file=3scale-operator load=bundles
INFO[0013] found csv, loading bundle                     dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605 file=3scale-operator.v0.3.0.clusterserviceversion.yaml load=bundles
INFO[0013] loading bundle file                           dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605/3scale-operator file=3scale-operator.package.yaml load=bundle
INFO[0013] loading bundle file                           dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605/3scale-operator file=3scale-operator.v0.3.0.clusterserviceversion.yaml load=bundle
...
Uploading ... 244.9kB/s
Pushed sha256:f73d42950021f9240389f99ddc5b0c7f1b533c054ba344654ff1edaf6bf827e3 to quay.io/my/redhat-operators:snapshot-0
```

Sometimes invalid manifests are accidentally introduced into Red Hat's catalogs, when this happens you may see some errors.

```sh
...
INFO[0014] directory                                     dir=/var/folders/st/9cskxqs53ll3wdn434vw4cd80000gn/T/300666084/manifests-829192605 file=4.2 load=package
W1114 19:42:37.876180   34665 builder.go:141] error building database: error loading package into db: fuse-camel-k-operator.v7.5.0 specifies replacement that couldn't be found
Uploading ... 244.9kB/s
...
```

These errors are usually non-fatal, and if the operator package mentioned doesn't contain an operator you plan to install or a dependency of one, then they can be ignored.

## Testing a snapshot

You can validate snapshot content by running it as a container and querying its gRPC API:

Pull a snapshot image.

```sh
$ docker pull quay.io/my/redhat-operators:snapshot-0
...
```

Run the snapshot image.

```sh
$ docker run -p -p 50051:50051 -it quay.io/my/redhat-operators:snapshot-0
...
```

Query the running snapshot for available packages using `grpcurl`.

```sh
$ grpcurl -plaintext  localhost:50051 api.Registry/ListPackages
{
  "name": "3scale-operator"
}
{
  "name": "amq-broker"
}
{
  "name": "amq-online"
}
...
```

Get the latest bundle in a channel.

```sh
$  grpcurl -plaintext -d '{"pkgName":"kiali-ossm","channelName":"stable"}' localhost:50051 api.Registry/GetBundleForChannel
{
  "csvName": "kiali-operator.v1.0.7",
  "packageName": "kiali-ossm",
  "channelName": "stable",
...
```

## Using a snapshot

You can resolve OLM `Subscriptions` with a snapshot image by referencing it in a `CatalogSource`:

Pull a snapshot image.

```sh
$  docker pull quay.io/my/redhat-operators:snapshot-0
...
```

Get the digest of the snapshot image.

```sh
$ docker inspect --format='{{index .RepoDigests 0}}' quay.io/my/redhat-operators:snapshot-0
quay.io/my/redhat-operators@sha256:f73d42950021f9240389f99ddc5b0c7f1b533c054ba344654ff1edaf6bf827e3
```

Assuming an `OperatorGroup` exists in namespace `my-ns` that supports your operator and its dependencies, create a `CatalogSource` using the snapshot digest.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: redhat-operators-snapshot
  namespace: my-ns
spec:
  sourceType: grpc
  image: quay.io/my/redhat-operators@sha256:f73d42950021f9240389f99ddc5b0c7f1b533c054ba344654ff1edaf6bf827e3
  displayName: Red Hat Operators Snapshot
```

Create a `Subscription` that resolves the latest available `servicemeshoperator` and its dependencies from the snapshot.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: servicemeshoperator
  namespace: my-ns
spec:
  source: redhat-operators-snapshot
  sourceNamespace: my-ns
  name: servicemeshoperator
  channel: 1.0
```

Updates to Red Hat's appr catalog's can be captured by a new snapshot, tested, and introduced to existing clusters by swapping out the `CatalogSource's` `spec.image` field with the new snapshot digest.

## My snapshot build succeeded, but the final image doesn't contain any operators!?

This can occur if the image repository specified by `--to` does not have Docker v2-2 enabled. As of November 2019, only select Quay namespaces have v2-2 enabled. The OLM team keeps a [redhat-operators snapshot repo](https://quay.io/repository/operator-framework/redhat-operators) available and manually adds snapshots upon request. If you need a new snapshot taken and do not have access to a v2-2 enabled repo, please reach out to `@olm-dev` in [#forum-operator-fw](https://coreos.slack.com/archives/C3VS0LV41).
