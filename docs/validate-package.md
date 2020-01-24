# How do I validate the package?

Once you've [created your operator's package manifests](packaging-an-operator.md), you will want to ensure that your package is valid and in the correct format. To ensure this, you should take several steps to ensure that your package can be used to install your operator in OLM. We will publish the package into a catalog, install that catalog onto a Kube cluster, and then install the operator onto that cluster. If all of that succeeds and your operator is behaving as expected, your package is valid.

## Linting

You can perform some basic static verification on your package by using [`operator-courier`](https://github.com/operator-framework/operator-courier).

```
$ pip3 install operator-courier
$ operator-courier verify manifests/my-operator-package
```

You can also use `operator-courier` to verify that your operator will be displayed properly on [OperatorHub.io](https://operatorhub.io/).

```
$ operator-courier verify --ui_validate_io manifests/my-operator-package
```

## Add your package to a catalog

The [operator-registry project](https://github.com/operator-framework/operator-registry) defines a format for storing sets of operators and exposing them to make them available on a cluster. As part of adding your package to an operator-registry catalog, the operator-registry tools will verify that your operator is packaged properly ("Does it have a valid CSV of the correct format?", "Does my CRD properly reference my CSVs?", etc.). The simplest way to test that your package can be added to a catalog is by actually attempting to create a catalog that includes your operator.

To create a catalog that includes your package, simply build a container image that uses the operator-registry command line tools to generate a registry and serve it. For example, create a file in the root of your project called `registry.Dockerfile`

```Dockerfile
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

This Dockerfile assumes that your package is in a directory called `./manifests/` similar to [this example](https://github.com/operator-framework/operator-registry/tree/master/manifests). It copies your manifests into the builder image, runs `initializer`, then copies the output into the final scratch image and defines the run command to serve the operator-registry.

Then just use your favorite container tooling to build the container image and push it to a registry:

```
docker build -t example-registry:latest -f registry-Dockerfile .
docker push example-registry:latest
```

Your catalog is published and we are ready to use it on your cluster.

## Install your operator

Now that you have created an operator-registry image that hosts your operator's package, add that catalog to your cluster:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: example-manifests
  namespace: your-namespace
spec:
  sourceType: grpc
  image: example-registry:latest
```

This will cause OLM to pull your image and create a pod in the designated namespace (`your-namespace`) that hosts your package. Your Catalog is now installed onto your cluster and your package is available!

Once the catalog has been loaded, your Operators package definitions are read by the `package-server`, a component of OLM. Watch your Operator packages become available:

```
$ kubectl get packagemanifests -n your-namespace

NAME                     AGE
your-operator            13m
```

Once loaded, you can query a particular package for its Operators that it serves across multiple channels. To obtain the default channel run:

```
$ kubectl get packagemanifests your-operator -o jsonpath='{.status.defaultChannel}' -n your-namespace

alpha
```

With this information, the operators package name, the channel and the name and namespace of your catalog you can now [install your operator with OLM](how-do-i-install-my-operator-with-olm.md)

Now that your operator is installed from your package, poke around and ensure that it is working as you expect. If so, your package is valid!
