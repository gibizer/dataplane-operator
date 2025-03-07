# DataPlane Design

The dataplane-operator prepares nodes with enough operating system and other
configuration to make them suitable for hosting OpenStack workloads.

It uses the
[openstack-baremetal-operator](https://github.com/openstack-k8s-operators/openstack-baremetal-operator)
to optionally provision baremetal. Ansible is used to deploy and configure
software on the nodes.

## DataPlane Design and Resources

The dataplane-operator exposes the concepts of dataplanes, roles, nodes, and
services as CRD's:

* [OpenStackDataPlane](https://github.com/openstack-k8s-operators/dataplane-operator/blob/main/config/crd/bases/dataplane.openstack.org_openstackdataplanes.yaml)
* [OpenStackDataPlaneNodeSet](https://github.com/openstack-k8s-operators/dataplane-operator/blob/main/config/crd/bases/dataplane.openstack.org_openstackdataplanenodesets.yaml)
* [OpenStackDataPlaneService](https://github.com/openstack-k8s-operators/dataplane-operator/blob/main/config/crd/bases/dataplane.openstack.org_openstackdataplaneservices.yaml)

Each `NodeSet` in a dataplane is represented by a corresponding
OpenStackDataPlaneNodeSet resource.  The OpenStackDataPlaneNodeSet CRD provides for a
logical grouping of nodes of a similar type. This is analogous to the concept of "roles"
in TripleO. Similarities within a `NodeSet` are defined by the user, and could be of a 
small scope (ansible port), or a large scope (same network config, nova config,
provisioning config, etc). The properties that all nodes in a `NodeSet` share is set
in the NodeTemplate field of the `NodeSet`'s Spec. Node specific parameters are then
defined under the `spec.nodeTemplate.nodes` section specific to that node. This is
described in more detail below.

OpenStackDataPlaneNodeSet implements a baremetal provisioning interface to
provision the nodes in the role. This interface can be used to provision the
initial OS on a set of nodes.

A `NodeSet` also provides for an inheritance model of node attributes. Properties
from the `NodeTemplate` section on the spec will automatically be inherited by the
nodes defined in the `NodeSet`. Nodes can also set their own properties within their
own section defined under the `spec.nodeTemplate.nodes` field. Options defined here
will override the inherited values from the `NodeSet`. See
[inheritance](inheritance.md) for more in depth details about how inheritance
works between `NodeSets` and individual `Nodes`.

The `OpenStackDataPlane` CRD provides a way to group all `NodeSets` together
into a single resource that represents the entire dataplane.

`OpenStackDataPlaneService` is a CRD for representing an Ansible based service to
orchestrate across the nodes. A composable service interface is provided that
allows for customizing which services are run on which roles and nodes, and for
defining custom services.

## Deployment

The deployment handling in `dataplane-operator` is implemented within the
deployment package at `pkg/deployment`. A `NodeSet` uses an ansible
inventory containing all the nodes defined by that `NodeSet` CR when it triggers
a deployment.
