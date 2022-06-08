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
	callback             api.ResourceManagerCallback // implement 
	stateMachine         *fsm.FSM
	stopChan             chan struct{}
	lock                 *sync.RWMutex
	outstandingAppsFound bool
}
```
