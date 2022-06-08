[ServiceContext](https://github.com/apache/yunikorn-core/blob/master/pkg/entrypoint/service_context.go#L30)
```go
type ServiceContext struct {
	RMProxy   api.SchedulerAPI        // implement SchedulerAPI and EventHandler
	Scheduler *scheduler.Scheduler    // implement EventHandler
	WebApp    *webservice.WebService
}
```
