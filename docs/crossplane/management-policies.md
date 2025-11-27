managementPolicies 
 Note
The managed resource managementPolicies option is a beta feature. Crossplane enables beta features by default.

The Provider determines support for management policies.
Refer to the Provider’s documentation to see if the Provider supports management policies.

Crossplane managementPolicies determine which actions Crossplane can take on a managed resource and its corresponding external resource.
Apply one or more managementPolicies to a managed resource to determine what permissions Crossplane has over the resource.

For example, give Crossplane permission to create and delete an external resource, but not make any changes, set the policies to ["Create", "Delete", "Observe"].

apiVersion: ec2.aws.m.upbound.io/v1beta1
kind: Subnet
spec:
  managementPolicies: ["Create", "Delete", "Observe"]
  forProvider:
    # Removed for brevity

The default policy grants Crossplane full control over the resources.
Defining the managementPolicies field with an empty array pauses the resource.

 Important
The Provider determines support for management policies.
Refer to the Provider’s documentation to see if the Provider supports management policies.
Crossplane supports the following policies:

Policy	Description
*	Default policy. Crossplane has full control over a resource.
Create	If the external resource doesn’t exist, Crossplane creates it based on the managed resource settings.
Delete	Crossplane can delete the external resource when deleting the managed resource.
LateInitialize	Crossplane initializes some external resource settings not defined in the spec.forProvider of the managed resource. See the late initialization section for more details.
Observe	Crossplane only observes the resource and doesn’t make any changes. Used for observe only resources.
Update	Crossplane changes the external resource when changing the managed resource.
The following is a list of common policy combinations:

Create	Delete	LateInitialize	Observe	Update	Description
✔️
✔️
✔️
✔️
✔️
Default policy. Crossplane has full control over the resource.
✔️
✔️
✔️
✔️
After creation any changes made to the managed resource aren’t passed to the external resource. Useful for immutable external resources.
✔️
✔️
✔️
✔️
Prevent Crossplane from managing any settings not defined in the managed resource. Useful for immutable fields in an external resource.
✔️
✔️
✔️
Crossplane doesn’t import any settings from the external resource and doesn’t push changes to the managed resource. Crossplane recreates the external resource if it’s deleted.
✔️
✔️
✔️
✔️
Crossplane doesn’t delete the external resource when deleting the managed resource.
✔️
✔️
✔️
Crossplane doesn’t delete the external resource when deleting the managed resource. Crossplane doesn’t apply changes to the external resource after creation.
✔️
✔️
✔️
Crossplane doesn’t delete the external resource when deleting the managed resource. Crossplane doesn’t import any settings from the external resource.
✔️
✔️
Crossplane creates the external resource but doesn’t apply any changes to the external resource or managed resource. Crossplane can’t delete the resource.
✔️
Crossplane only observes a resource.
No policy set. An alternative method for pausing a resource.
providerConfigRef 