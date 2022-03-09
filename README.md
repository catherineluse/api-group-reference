I made this reference to help with this issue https://github.com/rancher/dashboard/issues/4888 in which we made an effort to help users understand the context of resources in the role creation form. It helps if we group them by API group and scope (global, cluster or namespaced).

I used these references to verify that in the role creation form, the resources are assigned to the right API groups. In some cases, resources in different API groups can have the same name, so they can appear in multiple groups.

There are two references. The first, `mapApiGroupToResource`, lets you **control + F** an API group to find all the resources that are in the API group.

The second, `mapResourceToApiGroup`, lets you **control + F** a resource to find all the API groups that could be associated with a resource, given the name of the resource.

This is how I generated the references and validated the API groups for each resource:

1. Installed Rancher on a K3s v1.22.5 cluster.
2. Installed apps on the local cluster - CIS Benchmark, Istio, Logging, Longhorn, Monitoring, OPA Gatekeeper. These helm charts installed a bunch of custom resource definitions.
3. Got a list of all the schemas on the local cluster. Now that all the CRDs are included, that increases the number of schemas, so we have over 1,000 schemas.
4. For each schema, I looked at its ID to get its corresponding resource name an API group. Usually its ID was in the format `<API-group>.<resource-name>`. Then I auto-generated the references out of those IDs. Basically for each schema, I went to its API group in the API group list and added the resource name under it, and I went to the resource in the resource list and added the API group name under it.
5. For every resource in the original hardcoded list of resources, I looked it up in the resource reference to see what API group it belonged to and check if it belonged to multiple groups.
6. Then for every API group we listed resources for, I checked the API group reference to see if we weirdly left something out, in other words, if we listed all resources in an API group except one. I added a few resources based on this process:


- For monitoring, I added these resources, which were missing from the monitoring API group:
	- Routes
	- Receivers
	- Probes
	- ThanosRulers
	- AlertmanagerConfigs
- For istio, I added this resource because all the other resources in the API group were added except this:
	- WorkloadGroups
- For storage.k8s.io, I added this resource because all the other resources in the storage API group were added except this:
	- CSIStorageCapacitys

7. When going through the original hardcoded list of resources, if I could not find it in my API group reference list, that meant one of two things: either it belonged to the core Kubernetes API, or it belonged to a Helm chart app that wasn't installed, so the local cluster didn't have a schema for it. For the former, I grouped them under "Core K8s API" and for the latter, I removed them because after discussion in Slack, it was decided that those resources are deprecated and/or too obscure to be useful.