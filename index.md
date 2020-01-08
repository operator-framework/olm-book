# Operator Lifecycle Book

- [Introduction](docs/intro.md)

- [Glossary](docs/glossary.md)

## Foundational Concepts

- [What is an Operator?](docs/what-is-an-operator.md)
- [Why would I use OLM?](docs/)
- [What does OLM enable?](docs/what-does-olm-enable.md)
- [Operators on cluster](docs/)

## Under the hood

- [Operands](docs/)
- [Operator bundles](docs/)
- [Operator catalogs](docs/)
- [Subscriptions](docs/)
- [Operator dependencies and requirements](docs/operator-dependencies-and-requirements.md)
- [Operator update graphs and channels](docs/)
- [Operator versioning and release strategies](docs/)
- [Operand support matrices](docs/)
- [Operator install modes](docs/)
- [Operator scoping](docs/operator-scoping.md)

## Basic Use Cases

- [How do I install OLM?](docs/install-olm.md)
- [How do I package my Operator for OLM?](docs/packaging-an-operator.md)
- [How do I validate the package?](docs/)
- [How do I make my Operator part of a catalog?](docs/)
- [How do I install my Operator with OLM?](docs/how-do-i-install-my-operator-with-olm.md)
- [How do I list Operators available to install?](docs/list-available-operators.md)
- [How do I uninstall an Operator?](docs/uninstall-an-operator.md)
- [How do I discover the presence/availability of an Operator?](docs/discover-operator-presence.md)
- [How do I uninstall OLM?](docs/uninstall-olm.md)

## Advanced Use Cases

- [When do I need to update my Operator?](docs/when-to-update-my-operator.md)
- [How do I create an updated version of my Operator?](docs/)
- [How do I test an update before shipping?](docs/)
- [How do I ship an updated version of my Operator?](docs/)
- [How do I approve an update?](docs/)
- [How do I scope down an Operator?](docs/)
- [How can I install an Operator when I am not cluster admin?](docs/)
- [How do I rely on other Operators with my Operator?](docs/)
- [How can I configure / customize my Operator deployment?](docs/)
- [How can I set / override defaults to amend runtime behavior of my Operator?](docs/)
- [What annotations can I use to drive UIs?](docs/)
- [How do I change which users are able to use an Operator?](docs/)
- [How do I “hide” particular CRDs not intended for consumption by an end-user?](docs/)
- [How do I ship webhooks?](docs/)
- [When and how should a running Operator express that it is not upgradeable?](docs/)
- [When should an Operator upgrade its Operands?](docs/)
- [How should an Operator Author create and package an Operator for a singleton operand?](docs/)
- [How do I snapshot a Quay Appregistry operator catalog?](docs/snapshot-appr-catalog.md)

## Troubleshooting

- [Troubleshoot OLM installation](docs/)
- [Troubleshoot operator installation with OLM](docs/troubleshooting.md)

## OLM on OCP (OpenShift Container Platform)

- [FAQ](docs/openshift/faq.md)
- [Important changes by OCP release](docs/openshift/important-changes-by-release.md)
- [Operator Marketplace](docs/openshift/operator-marketplace.md)
