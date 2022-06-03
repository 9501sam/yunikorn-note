## placement manager
* [```(cc *ClusterContext) handleRMUpdateApplicationEvent(event *rmevent.RMUpdateApplicationEvent)```](https://github.com/apache/yunikorn-core/blob/a590b7d0059cc875bc9ba5c81451a3db14c54326/pkg/scheduler/context.go#L484)
    *  [```(pc *PartitionContext) AddApplication(app *objects.Application)```](https://github.com/apache/yunikorn-core/blob/a590b7d0059cc875bc9ba5c81451a3db14c54326/pkg/scheduler/partition.go#L307)
        *  [```(m *AppPlacementManager) PlaceApplication(app *objects.Application) error```](https://github.com/apache/yunikorn-core/blob/a590b7d0059cc875bc9ba5c81451a3db14c54326/pkg/scheduler/placement/placement.go#L135)

https://github.com/apache/yunikorn-core/blob/a590b7d0059cc875bc9ba5c81451a3db14c54326/pkg/scheduler/placement/placement.go#L33
```go
type AppPlacementManager struct {
	name        string
	rules       []rule
	initialised bool
	queueFn     func(string) *objects.Queue

	sync.RWMutex
}
```

https://github.com/apache/yunikorn-core/blob/a590b7d0059cc875bc9ba5c81451a3db14c54326/pkg/scheduler/placement/rule.go#L33
```go
// Interface that all placement rules need to implement.
type rule interface {
	// Initialise the rule from the configuration.
	// An error may only be returned if the configuration is not correct.
	initialise(conf configs.PlacementRule) error

	// Execute the rule and return the queue getName the application is placed in.
	// Returns the fully qualified queue getName if the rule finds a queue or an empty string if the rule did not match.
	// The error must only be set if there is a failure while executing the rule not if the rule did not match.
	placeApplication(app *objects.Application, queueFn func(string) *objects.Queue) (string, error)

	// Return the getName of the rule which is defined in the rule.
	// The basicRule provides a "unnamed rule" implementation.
	getName() string

	// Return the parent rule.
	// This method is implemented in the basicRule which each rule must be based on.
	getParent() rule
}
```

### logs about rules
```
### shim -> rm -> core, 2. handleRMUpdateApplicationEvent()
2022-05-31T09:03:23.314Z	INFO	scheduler/context.go:486	enter handleRMUpdateApplicationEvent()
2022-05-31T09:03:23.314Z	INFO	scheduler/context.go:487	&{Request:new:<applicationID:"nginx_2019_01_22_00001" queueName:"root.sandbox" partitionName:"[mycluster]default" ugi:<user:"nobody" > tags:<key:"namespace" value:"yunikorn" > tags:<key:"yunikorn.apache.org/schedulingPolicyParameters" value:"gangSchedulingStyle=Hard" > tags:<key:"yunikorn.apache.org/task-groups" value:"" > gangSchedulingStyle:"Hard" > rmID:"mycluster" }
2022-05-31T09:03:23.314Z	INFO	scheduler/partition.go:309	enter AddApplication()
2022-05-31T09:03:23.315Z	INFO	placement/placement.go:137	enter PlaceApplication()
2022-05-31T09:03:23.315Z	INFO	placement/placement.go:138	&{name: rules:[0xc003bdf950] initialised:true queueFn:0x993ce0 RWMutex:{w:{state:0 sema:0} writerSem:0 readerSem:0 readerCount:0 readerWait:0}}
2022-05-31T09:03:23.315Z	INFO	placement/placement.go:149	Executing rule for placing application	{"ruleName": "tag", "application": "nginx_2019_01_22_00001"}
2022-05-31T09:03:23.315Z	INFO	placement/tag_rule.go:61	enter placeApplication()
2022-05-31T09:03:23.315Z	INFO	placement/tag_rule.go:108	Tag rule intermediate result	{"application": "nginx_2019_01_22_00001", "queue": "root.yunikorn"}
2022-05-31T09:03:23.315Z	INFO	placement/tag_rule.go:117	Tag rule application placed	{"application": "nginx_2019_01_22_00001", "queue": "root.yunikorn"}
2022-05-31T09:03:23.315Z	INFO	placement/placement.go:208	Rule result for placing application	{"application": "nginx_2019_01_22_00001", "queueName": "root.yunikorn"}
2022-05-31T09:03:23.315Z	INFO	objects/queue.go:150	dynamic queue added to scheduler	{"queueName": "root.yunikorn"}
2022-05-31T09:03:23.315Z	INFO	scheduler/context.go:541	Added application to partition	{"applicationID": "nginx_2019_01_22_00001", "partitionName": "[mycluster]default", "requested queue": "root.sandbox", "placed queue": "root.yunikorn"}
```
