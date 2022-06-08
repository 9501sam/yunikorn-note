```main.go```
```go
var (
	version string
	date    string
)

func main() {
	log.Logger().Info("Build info", zap.String("version", version), zap.String("date", date))
	log.Logger().Info("starting scheduler",
		zap.String("name", constants.SchedulerName))

	conf.BuildVersion = version
	conf.BuildDate = date
	conf.IsPluginVersion = false

	serviceContext := entrypoint.StartAllServicesWithLogger(log.Logger(), log.GetZapConfigs())

	if sa, ok := serviceContext.RMProxy.(api.SchedulerAPI); ok {
		ss := shim.NewShimScheduler(sa, conf.GetSchedulerConf())      // here ^U^, is a KubernetesShim
		ss.Run()

		signalChan := make(chan os.Signal, 1)
		signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)
		for range signalChan {
			log.Logger().Info("Shutdown signal received, exiting...")
			ss.Stop()
			os.Exit(0)
		}
	}
}
```

[```KubernetesShim```](https://github.com/apache/yunikorn-k8shim/blob/6a2d0142732f05131c5c34abd9ec96143326bcc0/pkg/shim/scheduler.go#L46)
```go
type KubernetesShim struct {
	apiFactory           client.APIProvider
	context              *cache.Context
	appManager           *appmgmt.AppManagementService
	phManager            *cache.PlaceholderManager
	callback             api.ResourceManagerCallback // AsyncRMCallback: implement ResourceManagerCallback
	stateMachine         *fsm.FSM
	stopChan             chan struct{}
	lock                 *sync.RWMutex
	outstandingAppsFound bool
}
```

### logs about ```AsyncRMCallback```

```
2022-06-03T09:54:34.527Z	DEBUG	rmproxy/rmproxy.go:64	enqueue event	{"eventType": "*rmevent.RMApplicationUpdateEvent", "event": {"RmID":"mycluster","AcceptedApplications":[{"applicationID":"nginx_2019_01_22_00001"}],"RejectedApplications":[],"UpdatedApplications":null}, "currentQueueSize": 0}
2022-06-03T09:54:34.527Z	DEBUG	callback/scheduler_callback.go:104	UpdateApplication callback received	{"UpdateApplicationResponse": "accepted:<applicationID:\"nginx_2019_01_22_00001\" > "}
2022-06-03T09:54:34.527Z	DEBUG	callback/scheduler_callback.go:110	callback: response to accepted application	{"appID": "nginx_2019_01_22_00001"}
2022-06-03T09:54:34.527Z	INFO	callback/scheduler_callback.go:114	Accepting app	{"appID": "nginx_2019_01_22_00001"}
2022-06-03T09:54:34.527Z	DEBUG	cache/application.go:734	shim app state transition	{"app": "nginx_2019_01_22_00001", "source": "Submitted", "destination": "Accepted", "event": "AcceptApplication"}
2022-06-03T09:54:34.528Z	DEBUG	configs/configvalidator.go:331	checking partition queue config	{"partitionName": "default"}
2022-06-03T09:54:34.528Z	DEBUG	configs/configvalidator.go:133	checking placement rule config	{"partitionName": "default"}
2022-06-03T09:54:35.508Z	DEBUG	scheduler/scheduler.go:167	inspect outstanding requests
2022-06-03T09:54:35.527Z	DEBUG	cache/application.go:551	postAppAccepted on cached app	{"appID": "nginx_2019_01_22_00001", "numTaskGroups": 0, "numAllocatedTasks": 0}
2022-06-03T09:54:35.527Z	DEBUG	cache/application.go:523	Skip reservation stage: no task groups defined	{"appID": "nginx_2019_01_22_00001"}
2022-06-03T09:54:35.527Z	INFO	cache/application.go:557	Skip the reservation stage	{"appID": "nginx_2019_01_22_00001"}
2022-06-03T09:54:35.527Z	DEBUG	cache/application.go:734	shim app state transition	{"app": "nginx_2019_01_22_00001", "source": "Accepted", "destination": "Running", "event": "RunApplication"}
2022-06-03T09:54:35.528Z	DEBUG	configs/configvalidator.go:331	checking partition queue config	{"partitionName": "default"}
2022-06-03T09:54:35.528Z	DEBUG	configs/configvalidator.go:133	checking placement rule config	{"partitionName": "default"}
2022-06-03T09:54:36.508Z	DEBUG	scheduler/scheduler.go:167	inspect outstanding requests
2022-06-03T09:54:36.527Z	DEBUG	cache/task.go:600	shim task state transition	{"app": "nginx_2019_01_22_00001", "task": "221351f9-d40c-4c38-af4d-bbb7ade174fc", "taskAlias": "yunikorn/nginx-758d456f6b-bzvff", "source": "New", "destination": "Pending", "event": "InitTask"}
2022-06-03T09:54:36.527Z	DEBUG	cache/task.go:600	shim task state transition	{"app": "nginx_2019_01_22_00001", "task": "221351f9-d40c-4c38-af4d-bbb7ade174fc", "taskAlias": "yunikorn/nginx-758d456f6b-bzvff", "source": "Pending", "destination": "Scheduling", "event": "SubmitTask"}
2022-06-03T09:54:36.527Z	DEBUG	cache/task.go:310	scheduling pod	{"podName": "nginx-758d456f6b-bzvff"}
2022-06-03T09:54:36.527Z	DEBUG	cache/task.go:320	send update request	{"request": "asks:<allocationKey:\"221351f9-d40c-4c38-af4d-bbb7ade174fc\" applicationID:\"nginx_2019_01_22_00001\" resourceAsk:<resources:<key:\"memory\" value:<value:1024000000 > > resources:<key:\"vcore\" value:<value:500 > > > maxAllocations:1 tags:<key:\"kubernetes.io/label/app\" value:\"nginx\" > tags:<key:\"kubernetes.io/label/applicationId\" value:\"nginx_2019_01_22_00001\" > tags:<key:\"kubernetes.io/label/pod-template-hash\" value:\"758d456f6b\" > tags:<key:\"kubernetes.io/label/queue\" value:\"root.sandbox\" > tags:<key:\"kubernetes.io/meta/namespace\" value:\"yunikorn\" > tags:<key:\"kubernetes.io/meta/podName\" value:\"nginx-758d456f6b-bzvff\" > > rmID:\"mycluster\" "}
2022-06-03T09:54:36.527Z	INFO	rmproxy/rmproxy.go:309	enter UpdateAllocation()
2022-06-03T09:54:36.527Z	INFO	rmproxy/rmproxy.go:310	asks:<allocationKey:"221351f9-d40c-4c38-af4d-bbb7ade174fc" applicationID:"nginx_2019_01_22_00001" resourceAsk:<resources:<key:"memory" value:<value:1024000000 > > resources:<key:"vcore" value:<value:500 > > > maxAllocations:1 tags:<key:"kubernetes.io/label/app" value:"nginx" > tags:<key:"kubernetes.io/label/applicationId" value:"nginx_2019_01_22_00001" > tags:<key:"kubernetes.io/label/pod-template-hash" value:"758d456f6b" > tags:<key:"kubernetes.io/label/queue" value:"root.sandbox" > tags:<key:"kubernetes.io/meta/namespace" value:"yunikorn" > tags:<key:"kubernetes.io/meta/podName" value:"nginx-758d456f6b-bzvff" > > rmID:"mycluster" 
```

