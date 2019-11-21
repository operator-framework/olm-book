# How do I discover the presence/availability of an Operator?

## Discovering installed operators
Checking the presence of custom installed operators in the cluster is simply a matter of checking all the 
`ClusterServiceVersions` (CSV) that are available in the cluster. Every  operator installed by OLM has a corresponding CSV
associated with it that that contains all the details of the operator. Running `kubectl get csvs -A`
would return all CSVs across all namespaces and provide a high-level view of all custom installed operators. 

If the ClusterServiceVersion fails to show up or does not reach the `Succeeded` phase, please check the [troubleshooting documentation](https://) to debug your installation.  

## Finding available operators
Operators are available from a variety of sources, or catalogs. Some catalogs are bundled in the default installation of
OLM, and there are also online catalogs of operators available to install within the cluster.  

### Installing operators from within the cluster
A collection of operators come packaged with the default OLM installation and can be examined by querying the packageserver. 
These operators come from the community as well as officially supported Red Hat operators. The package server is an
 [apiserver extension](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/)  
that aggregates information on all operators available to install from catalog sources in the cluster. To see available
packages run `kubectl get packagemanifests`. 

To get more information about a particular operator installed within the cluster from a catalogsource, run 
`kubectl describe packagemanifests <operator name>`. This will provide some detailed information about the operator such as 
its intended use, applicable CRDs, and resources for support with that particular operator. 

### Finding operators to install 
A great resource to find all available operators is [OperatorHub](https://operatorhub.io/), an online catalog
of installable operators. There are over 80 available operators available to install, with details around the installation process,
the upgrade process, and operator support information. Installing operators from OperatorHub is seamless if OLM is installed. 

