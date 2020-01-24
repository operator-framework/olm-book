# Adding an operator to a catalog

A pre-requisite for adding an operator to a catalog is for all of the [packaging](packaging-an-operator.md) to be completed. If that hasn't been done, stop and do that first.

## Creating a new bundle image manually

This section contains instructions for creating a new bundle image for adding as a new standalone catalog. However, there are new and improved ways that should be used instead that are described in the [next section](#creating-a-new-registry-image). This section is recommended reading only if you wish to get a better understanding of how data in an image is read in general.

### High level overview

In order to describe what we're working towards, a view from the top is helpful. Essentially the manifest files will be put into a container along with a few binaries. The binaries provide the functionality to serve the manifest files over gRPC and a gRPC health probe to report the gRPC connection status to Kubernetes.

Once an image is built containing the operator manifests, a `CatalogSource` can be created that tells OLM about the image. OLM will pull in metadata (over gRPC) from the image and make that information available to the [package manifest list](list-available-operators.md).

### Building the bundle image manually

For convienence, there is a [published image](http://quay.io/operator-framework/upstream-registry-builder) that includes the operator-registry tools for making a bundle image. These tools include the `initializer` that takes a directory of manifests and converts them into one file that is an embedded database. The other critical tool is the operator-registry server that is used to both read the database and to make it accessible over gRPC, which was also described earlier.

The [Dockerfile](https://github.com/operator-framework/operator-registry/blob/f544e227bb0e549f156aae7b319aeac976756e3f/upstream-example.Dockerfile) to use looks like this:

```yaml
FROM quay.io/operator-framework/upstream-registry-builder as builder

COPY manifests manifests
RUN ./bin/initializer -o ./bundles.db

FROM scratch
COPY --from=builder /build/bundles.db /bundles.db
COPY --from=builder /build/bin/registry-server /registry-server
COPY --from=builder /bin/grpc_health_probe /bin/grpc_health_probe
EXPOSE 50051
ENTRYPOINT ["/registry-server"]
CMD ["--database", "bundles.db"]
```

Ensure that your manifests directory is in the same directory as this file and then build with a command that resembles:

`docker build -t your-bundle:latest -f upstream-example.Dockerfile .`

## Creating a new registry image

A registry image is a container image that contains the manifests, but uses a different format to put data into the container. Currently, the `opm` command has some alpha level functionality for generating a registry image. Note that due to this alpha status, the command line arguments are hidden from the help output of opm. One can execute `opm alpha` to see what arguments are available. In this case, the build command is the one that should be used. For example, using the manifests in the OLM repo one can generate an etcd registry image using a command similar to:

`bin/opm alpha bundle build -c singlenamespace-alpha, clusterwide-alpha -e singlenamespace-alpha -d manifests/etcd/0.6.1 -p etcd -t quay.io/your/etcd-operator:v0.6.1

These images end up being smaller than the previously described bundle images because no binaries are in the container, just manifest files and labels to describe its content. (On the implementation side, what ends up happening is upon detecting an image of this type an image is started with a gRPC server binary that is injected on the fly.)

### Adding to an existing catalog

This section describes how to add a registry image to an existing catalog image (all catalogs are backed by images that contain databases or manifest data). To be clear, bundle images can only contain manifests for one type of operator. A catalog image does not have this limitation.

Here again, the `opm` tool is used for catalog image modifications.

 opm index add --bundles quay.io/operator-framework/operator-bundle-prometheus:0.15.0 --from-index quay.io/operator-framework/monitoring:1.0.0 --tag quay.io/operator-framework/monitoring:1.0.1

 Note that the bundle image can not be added to a catalog image, only the registry image as described in the previous section.

### Adding to a new catalog

This example demonstrates how to create a completely new catalog image based on the [new registry image](#creating-a-new-registry-image).

opm index add --bundles quay.io/your/etcd-operator:v0.6.1 --tag your-index:v1
