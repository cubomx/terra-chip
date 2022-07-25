# User Adoption

## Terraform Adoption Milestones
- Discovering the value of IaC (using a universal provisioning language)
- Evaluate Terra OS against an existing set of infra
- Consumption of modules
- Exploration of Enterprise features
- Module registry 
- Producer/consumer model (push modules to registry and others that use them )
- Self service infra with little to no oversight from a centralized provisioning team
- Fully autonomous infra with little to no input. **At the press of button**

## The Challenge
- **Increased costs** from over-provisioning and unused/orphaned
- **Reduced productivity**: devs waiting for provisioning/scale
- **Increased risk** manual, error-prone processes

## The Solution
Library of versioned & validad infra to be consumed.
- Provision without an operator bottleneck
- Serve + infra req with predefined modules
- Reduce risk

## Service Now Integration
End-user can req infra from ServiceNow Service Catalog & TFE can provide an automated way to 
service those reqs.

### Service Catalog
Makes it simple to use TFE for provisioning, policy enforcement, logging of all infra reqs. "Cookie cutter" envs, not ongoing updates.

## Advanced Use Cases
- **Code Promotion**: running a deployment after *deploying & testing changes in a workspace*
- **Blue/Green App Deployment**: deploy test versions of the app infra & andto various beta envs, 
auto validate & promote releases.