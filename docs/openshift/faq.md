# FAQ (OpenShift Specific)

This FAQ is reserved for questions specifically pertaining to using OLM on OpenShift.

## Developing an Operator

### Q: I'm developing a new version of an operator already published to one of the default catalogs (redhat-operators, certified-operators, or community-operators). How can I hide the published versions from OLM and the OperatorHub UI?

Deleting the default `OperatorSource` and `CatalogSource` of the catalog your operator exists in will prevent OLM and the OperatorHub UI from using it. However, default `OperatorSources` are regenerated upon deletion. Instructions to disable regeneration can be found in the [`marketplace-operator` configuration section](operator-marketplace.md#configuration).
