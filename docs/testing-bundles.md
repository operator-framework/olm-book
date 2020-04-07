## How do I Test My Operator Bundle?

#### Operator Bundle in 10 Steps

| Step      | Description |
| --------- | ----------- |
| 1         | Make the changes you want to your CSV       |
| 2         | Lint your bundle with operator-courier verify command        |
| 3         | Test the bundle with `operator-sdk scorecard` command        |
| 4         | If everything goes well, build operator bundle with operator-registry image + bundle        |
| 5         | Push to a repository with a testing tag or name        |
| 6         | Create a catalog source pointing to that bundle image        |
| 7         | Create an operator group on the desired namespace        |
| 8         | Create the subscription to your operator        |
| 9         | Check InstallPlan, CSV and deployment        |
| 10        | Finally, give a try on the operator via the UI        |

[Here](https://github.com/operator-framework/community-operators/blob/master/docs/using-scripts.md) you can find a few tips on how to build a makefile to automated your specific test.


#### Check the details below:
1. Make the changes you want to your CSV 

If you don't know what a CSV is begin [here](https://github.com/operator-framework/olm-book/blob/master/docs/packaging-an-operator.md#writing-your-operator-manifests).

Boost your operator presentation putting good metadata content in your bundle. Check those 3 references:

Documentation:

- [CSV Reference](https://github.com/operator-framework/operator-lifecycle-manager/blob/master/doc/design/building-your-csv.md)

- [The OpenShift Descriptors](https://github.com/openshift/console/tree/master/frontend/packages/operator-lifecycle-manager/src/components/descriptors)
- [OpenShift Operator Hub UI on Steroids - x-Descriptors](https://github.com/openshift/console/blob/master/frontend/packages/operator-lifecycle-manager/src/components/descriptors/reference/reference.md)

2. Lint your bundle with operator-courier verify command

`operator-courier verify --ui_validate_io my-operator-bundle/`

```
If there is no output, the bundle passed operator-courier validation.
If there are errors, your bundle will not work. If there are warnings
we still encourage you to fix them before proceeding to the next step.
```

Documentation:

[operator-courier](https://github.com/operator-framework/operator-courier)
[Testing your Operator](https://github.com/operator-framework/community-operators/blob/master/docs/testing-operators.md)

*   Please note that some commands have been updated and are not present on `testing your operator` documentation. Check the version release notes and the context help of your operator-sdk binary. But it's still a good source of information.

3. Test the bundle with `operator-sdk scorecard` command:

    - Setup the `.osdk-scorecard.yaml` configuration file in the root of your project. [Check the docs](https://github.com/operator-framework/operator-sdk/blob/master/doc/test-framework/scorecard.md#config-file).
    - Create your operator namespace if needed.
    - Then, run the scorecard: `operator-sdk scorecard` + add the parameters you think may be useful. If your config file is in the root of your project it may be just that. Check the context help with `operator-sdk scorecard --help`.
    - **Example**: Limit scorecard to `basic` test and directly provide kubeconfig: `operator-sdk scorecard --selector=suite=basic --kubeconfig ../auth/kubeconfig`
    - Check the errors, if you get some, then debug them one by one until you have all with a `pass` status. Generally it's related to your CSV, but don't forget to include a version field to your CR Spec on your api and a Status field with valid values on your CR status field.

An example of `.osdk-scorecard.yaml`:

```
scorecard:
  output: json
  bundle: "deploy/olm-catalog/nsm-operator/"
  plugins:
    - basic:
        namespace: nsm
        cr-manifest:
          - "deploy/crds/nsm.networkservicemesh.io_v1alpha1_nsm_cr.yaml"
        
    - olm:
        namespace: nsm
        cr-manifest:
          - "deploy/crds/nsm.networkservicemesh.io_v1alpha1_nsm_cr.yaml"
        csv-path: "deploy/olm-catalog/nsm-operator/0.0.1/nsm-operator.v0.0.1.clusterserviceversion.yaml"
```
Most common problems faced when running the scorecard tests and how to solve them:

* <b>Resources not listed under Custom Resource Definition</b>. It will check if you have the types of resources owned by your primary resource. So let's say your operator deploys deamonsets, deployments, secrets etc it must have something like this:

```
...
spec:
  customresourcedefinitions:
    owned:
    - description: A running Prometheus instance
      displayName: Prometheus
      kind: Prometheus
      name: prometheuses.monitoring.coreos.com
      resources:
      - kind: StatefulSet
        version: v1beta2
      - kind: Pod
        version: v1
      - kind: ConfigMap
        version: v1
      - kind: Service
        version: v1
      specDescriptors:
      ...
```
* <b>Status field not present</b>. It will try to run an http get against your api to check if your status field have valid information. A common approach for your custom resource is to use a phase field with three or four states like `creating`, `running` or `terminating` for example. If you do change this make sure to at least `operator-sdk generate k8s` and `operator-sdk generate crds` in order to generate the necessary updates in your api base code in correct format. If you wish to go further you can validate against the openapi scheme using the [openapi-gen tool](https://github.com/OpenAPITools/openapi-generator). Check the sample code below. It will be under your pkg/api directory.

```
// NSMPhase is the type for the operator phases
type NSMPhase string

// Operator phases
const (
	NSMPhaseInitial     NSMPhase = ""
	NSMPhasePending     NSMPhase = "Pending"
	NSMPhaseCreating    NSMPhase = "Creating"
	NSMPhaseRunning     NSMPhase = "Running"
	NSMPhaseTerminating NSMPhase = "Terminating"
)

// NSMStatus defines the observed state of NSM
// +k8s:openapi-gen=true
type NSMStatus struct {
	// Operator phases during deployment
	Phase NSMPhase `json:"phase"`
}    
```
Then it needs to be updated by your controller at some point like that:

```
	if nsm.Status.Phase == nsmv1alpha1.NSMPhaseInitial {
		nsm.Status.Phase = nsmv1alpha1.NSMPhaseCreating
		if updateErr := r.client.Status().Update(context.TODO(), nsm); updateErr != nil {
			reqLogger.Info("Failed to update status", "Error", err.Error())
		}
	}
```

* <b>Scorecard Resources not garbage collected</b>. The scorecard test should cleanup the Kubernetes objects it creates but prematurely terminating the test may cause some objects to remain. You may need to need to manually delete the following resources before running the test again:

```
oc delete secret scorecard-kubeconfig

oc delete serviceaccount <your-operator-service-account>

oc delete role/clusterrole <your-operator-role>

oc delete rolebinding/clusterrolebinding <your-operator-binding>

oc delete deployment <your-operator-deployment>
```


4. If everything goes well, build operator bundle with operator-registry image + bundle.
```
podman build -t quay.io/my-container-registry-namespace/my-manifest-bundle:<your tag here> -f <your container config file> .
```
Here is an example of a container configuration file for the registry-server. Please observe that the `COPY` command is copying your bundle manifests into a folder called manifests inside the image. So, don't forget to make them available in your path or change the path to suit your needs. + Observe that you can you use other container builders than podman as well. Consider buildah, it's quite interesting too!

```
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
Documentation:

[Operator Registry](https://github.com/operator-framework/operator-registry)

5. Push to a repository with a testing tag or name

`podman push -t quay.io/my-container-registry-namespace/my-manifest-bundle:<your tag here>`

**Note:** Be sure to use the latest version of `podman` (or `docker`). This will ensure that your image has been pushed with the [v2schema2](https://docs.docker.com/registry/spec/manifest-v2-2/) format.  You can verify this by running one of the following commands after successfully pushing the image:

```skopeo inspect --raw --creds 'username':'password' docker://<url>/<account>/<reponame>:<tag> | grep schemaVersion```

```docker manifest inspect <url>/<account>/<reponame>:<tag> | grep schemaVersion```

The `schemaVersion` should be set to `2`.

6. Create a catalog source pointing to that bundle image on the openshift-marketplace namespace:

Here's an example:
```
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: my-operators-catalog
  namespace: openshift-marketplace
spec:
  sourceType: grpc
  image: quay.io/my-container-registry-namespace/my-manifest-bundle:latest
  displayName: My Registry Display
  publisher: mypublisher
EOF
```
  - OBS: for kubernetes you may replace the namespace openshift-marketplace by olm.

7. Create an operator group on the desired namespace targeting the namespace where the operator will be installed. If the install plan should install in one namespace only it may be the same in both fields:

```
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: my-operator
  namespace: my-operator-ns
spec:
  targetNamespaces:
  - my-operator-ns
EOF
```
8. Create the subscription to your operator on the same namespaces targeted by the operator group:

```
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: my-operator-subscription
  namespace: my-operator-ns
spec:
  channel: alpha
  name: my-operator-package-name
  source: my-operators-catalog
  sourceNamespace: openshift-marketplace
EOF
```
  - OBS: for kubernetes you may replace the sourceNamespace openshift-marketplace by olm.

9. Check InstallPlan, CSV and deployment

`oc get installplan -n my-operator-namespace`

```
NAME            CSV                   APPROVAL    APPROVED
install-l8wvr   my-operator-v0.0.1    Automatic   true
```
`oc  get csv -n my-operator-namespace`
```
NAME                  DISPLAY                         VERSION   REPLACES   PHASE
my-operator.v0.0.1   My Operator                      0.0.1                Succeeded
```

10. Finally, give a try on the operator via the UI on the operator hub tab. You will see that it will appear as custom instead of community or certified. That's your test deployment. If you want to have a preview of that before running it on an actual cluster you can try the preview page on https://operatorhub.io/preview.


