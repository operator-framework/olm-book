# How do I ship an operator that supports multiple node architectures?

It is assumed that all operators run on linux hosts, but can be configured to manage workloads on other architectures if worker nodes are available.

Which os/arches are supported by the operator can be indicated by adding labels to the ClusterServiceVersion providing the operator.

## Supported Labels

Labels indicating supported os and arch are defined by:

```yaml
labels:
    operatorframework.io/arch.<GOARCH>: supported
    operatorframework.io/os.<GOOS>: supported
```

Where `<GOARCH>` and `<GOOS>` are one of the values [listed here](https://github.com/golang/go/blob/master/src/go/build/syslist.go).

## Multiple Architectures

Some operators may support multiple node architectures or oses. In this case, multiple labels can be added. For example, an operator that support both windows and linux workloads:

```yaml
labels:
    operatorframework.io/os.windows: supported
    operatorframework.io/os.linux: supported
```

## Defaults

If a ClusterServiceVersion does not include an `os` label, it is treated as if it has the following label:

```yaml
labels:
    operatorframework.io/os.linux: supported
```


If a ClusterServiceVersion does not include an `arch` label, it is treated as if it has the following label:

```yaml
labels:
    operatorframework.io/arch.amd64: supported
```

## Filtering available operators by os or arch

Only windows:

```sh
$ kubectl get packagemanifests -l operatorframework.io/os.windows=supported
```

## Caveats

Only the labels on the [HEAD of the default channel](glossary.md#channel-head) are considered for filtering PackageManifests by label.

This means, for example, that providing an alternate architecture for an operator in the non-default channel is possible, but will not be available for filtering in the PackageManifest API.
