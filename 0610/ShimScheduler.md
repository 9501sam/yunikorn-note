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
505 2022-06-03T09:54:34.527Z    DEBUG   rmproxy/rmproxy.go:64   enqueue event   {"eventType": "*rmevent.RMApplicationUpdateEvent", "event": {"RmID":"mycluster","AcceptedApplications":[{"applicationID":"nginx_2019_01_22_00001"}],"RejectedAp    plications":[],"UpdatedApplications":null}, "currentQueueSize": 0}
506 2022-06-03T09:54:34.527Z    DEBUG   callback/scheduler_callback.go:104  UpdateApplication callback received {"UpdateApplicationResponse": "accepted:<applicationID:\"nginx_2019_01_22_00001\" > "}
507 2022-06-03T09:54:34.527Z    DEBUG   callback/scheduler_callback.go:110  callback: response to accepted application  {"appID": "nginx_2019_01_22_00001"}
508 2022-06-03T09:54:34.527Z    INFO    callback/scheduler_callback.go:114  Accepting app   {"appID": "nginx_2019_01_22_00001"}
509 2022-06-03T09:54:34.527Z    DEBUG   cache/application.go:734    shim app state transition   {"app": "nginx_2019_01_22_00001", "source": "Submitted", "destination": "Accepted", "event": "AcceptApplication"}
```

