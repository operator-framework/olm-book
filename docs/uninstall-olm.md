# How do I uninstall OLM?

You can symmetrically uninstall OLM as you did to install it. Specifically all OLM resources in [crds.yaml](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/deploy/upstream/quickstart/crds.yaml) and [olm.yaml](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/deploy/upstream/quickstart/olm.yaml) should be deleted. The `apiservices` should be removed as the first step, preventing it from becoming a dangling resource. 

Uninstalling OLM does not necessarily clean up the operators it maintained. Please clean up installed operator resources before uninstalling OLM, especially for resources that do not have an owner reference.

## Uninstall Released OLM

For uninstalling release versions of OLM, you can use the following commands:

```bash
export OLM_RELEASE=<olm-release-version>
kubectl delete apiservices.apiregistration.k8s.io v1.packages.operators.coreos.com
kubectl delete -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/${OLM_RELEASE}/crds.yaml
kubectl delete -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/${OLM_RELEASE}/olm.yaml
```

> NOTE: You can identify which version of OLM you are using by inspecting the version of the packageserver CSV.

> ```bash
> export OLM_NAMESPACE=<olm-namespace>
> kubectl -n $OLM_NAMESPACE get csvs
> NAME          DISPLAY        VERSION REPLACES PHASE
> packageserver Package Server 0.13.0           Succeeded
> ```

## Uninstall From Git Repository Master Branch

You can also uninstall OLM from the master branch of the [operator-framework/operator-lifecycle-manager](https://github.com/operator-framework/operator-lifecycle-manager/) repository with the following: 

```bash
kubectl delete apiservices.apiregistration.k8s.io v1.packages.operators.coreos.com
kubectl delete -f https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/crds.yaml
kubectl delete -f https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/olm.yaml
```

## Verify OLM Uninstall

Primarily, you can check that OLM has been uninstalled by checking the OLM namespace.

```bash
kubectl get namespace $OLM_NAMESPACE
Error from server (NotFound): namespaces "$OLM_NAMESPACE" not found
```

More specifically, you can verify that OLM has been uninstalled successfully by making sure that OLM **owned** `CustomResourceDefinitions` are removed:

```bash
kubectl get crd | grep operators.coreos.com
```

You can also check that the OLM `deployments` are terminated:

```bash
kubectl get deploy -n $OLM_NAMESPACE
No resources found.
```

The `role` and `rolebinding` in the OLM namespace are removed:

```bash
kubectl get role -n $OLM_NAMESPACE
No resources found.
```

```bash
kubectl get rolebinding -n $OLM_NAMESPACE
No resources found.
```

At last, the OLM namespace should also be terminated.
