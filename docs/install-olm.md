# How do I install OLM?

The OLM is installable on Kubernetes clusters. For the following instructions to work, you must have a Kubernetes cluster running and the `kubectl` is able to communicate with the API server of that cluster. For more information about configuring `kubectl`, please visit [here](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/).

Note: The OLM can be tested locally with a [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) cluster, currently supporting version 1.16.0. For more information see [Makefile](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/Makefile).

## Install Released OLM
For installing release versions of OLM, for example version 0.12.0, you can use the following command:

```bash
export olm_release=0.12.0
kubectl apply -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/${olm_release}/crds.yaml
kubectl apply -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/${olm_release}/olm.yaml
```

Learn more about available releases [here](https://github.com/operator-framework/operator-lifecycle-manager/releases).

## Install From Git Repository Master Branch

You can install OLM from the master branch of the [operator-framework/operator-lifecycle-manager](https://github.com/operator-framework/operator-lifecycle-manager/) repository with the following: 

```bash
kubectl create -f https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/crds.yaml
kubectl create -f https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/olm.yaml
```
You can also clone the entire git repository and use the [Makefile](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/Makefile) for deploying OLM locally on [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) for development purposes.

```bash
git clone https://github.com/operator-framework/operator-lifecycle-manager.git
cd operator-lifecycle-manager
make run-local
```

## Verify OLM Install

You can verify the necessary CustomResourceDefinitions are created from applying the `crds.yaml` file with the following:

```bash
$ kubectl get crd
NAME                                          CREATED AT
catalogsources.operators.coreos.com           2019-10-21T18:15:27Z
clusterserviceversions.operators.coreos.com   2019-10-21T18:15:27Z
installplans.operators.coreos.com             2019-10-21T18:15:27Z
operatorgroups.operators.coreos.com           2019-10-21T18:15:27Z
subscriptions.operators.coreos.com            2019-10-21T18:15:27Z
```
You can also visualize OLM deployments from applying `olm.yaml` file with the following:

```bash
$ kubectl get deploy -n olm
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
catalog-operator   1/1     1            1           5m52s
olm-operator       1/1     1            1           5m52s
packageserver      2/2     2            2           5m43s
```

## Verify The Default Catalog

OLM ships with a default catalog of packaged Operatrs. To verify that it loaded correctly run

```bash
$ kubectl get catalogsource -n olm
NAME                    DISPLAY               TYPE   PUBLISHER        AGE
operatorhubio-catalog   Community Operators   grpc   OperatorHub.io   13s
```

You inspect the catalog contents by listing all the `packagemanifests` that ship with it:

```bash
kubectl get packagemanifests                                                   
NAME                               CATALOG               AGE
akka-cluster-operator              Community Operators   84s
anchore-engine                     Community Operators   84s
appsody-operator                   Community Operators   84s
aqua                               Community Operators   84s
atlasmap-operator                  Community Operators   84s
aws-service                        Community Operators   84s
awss3-operator-registry            Community Operators   84s
banzaicloud-kafka-operator         Community Operators   84s
[...]
```

You can now proceed to either [install an Operator](./how-do-i-install-my-operator-with-olm.md) or add your own catalog.