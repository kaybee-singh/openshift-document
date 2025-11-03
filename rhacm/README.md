RHACM Terminologies.

1. Policy - 
2. Governance - It provides policy based approach and those policies will have a set of rules to enforce some policy standards. 
3. Policy Controller - To apply remediation and to look for policy violation.

Governance Components.

Dashboard
Policies
Policy Controller
stolostron/policy-collection  repository on github contains the policy examples, which can be used.

Use of RHACM Governace.

To manage compliance of multiple clusters on the basis of policies. It will ensure if cluster is compliant with the policy. If the cluster is not compliant the policy controller can be used to remediate the cluster.
It can work across different environments like cloud, data centers.

RHACM Components on managed cluster.

Spec sync controller
Policy custom resource
Template Sync Controller.
Gatekeeper
Policy Controller

RHACM Components on Hub cluster

Governance.
Polocy Propagator Controller
Policy customer resource
Policy automation custom resource
Policy binding custom resource
Placement custom resource.

Openscap in ACM.

Openscap checks the compliance of Openhsift resources based on the communicated based compliance content available at https://github.com/ComplianceAsCode/content
In openshift content is distributed to the Openshift nodes as a container image 
