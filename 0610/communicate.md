## shim
## interface
``` go
// api
type SchedulerAPI interface {
	// Register a new RM, if it is a reconnect from previous RM, cleanup
	// all in-memory data and resync with RM.
	RegisterResourceManager(request *si.RegisterResourceManagerRequest, callback ResourceManagerCallback) (*si.RegisterResourceManagerResponse, error)

	// Update allocation request
	UpdateAllocation(request *si.AllocationRequest) error

	// Update application request
	UpdateApplication(request *si.ApplicationRequest) error

	// Update node info
	UpdateNode(request *si.NodeRequest) error

	// Notify scheduler to reload configuration and hot-refresh in-memory state based on configuration changes
	UpdateConfiguration(clusterID string) error
}

// RM side needs to implement this API
type ResourceManagerCallback interface {

	//Receive Allocation Update Response
	UpdateAllocation(response *si.AllocationResponse) error

	//Receive Application Update Response
	UpdateApplication(response *si.ApplicationResponse) error

	//Receive Node Update Response
	UpdateNode(response *si.NodeResponse) error

	// Run a certain set of predicate functions to determine if a proposed allocation
	// can be allocated onto a node.
	Predicates(args *si.PredicatesArgs) error

	// This plugin is responsible for transmitting events to the shim side.
	// Events can be further exposed from the shim.
	SendEvent(events []*si.EventRecord)

	// Scheduler core can update container scheduling state to the RM,
	// the shim side can determine what to do incorporate with the scheduling state

	// update container scheduling state to the shim side
	// this might be called even the container scheduling state is unchanged
	// the shim side cannot assume to only receive updates on state changes
	// the shim side implementation must be thread safe
	UpdateContainerSchedulingState(request *si.UpdateContainerSchedulingStateRequest)

	// Update configuration
	UpdateConfiguration(args *si.UpdateConfigurationRequest) *si.UpdateConfigurationResponse
}
```

## core
