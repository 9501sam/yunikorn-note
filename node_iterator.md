* [```(sa *Application) tryNodes(ask *AllocationAsk, iterator NodeIterator) *Allocation```](https://github.com/apache/yunikorn-core/blob/branch-1.0/pkg/scheduler/objects/application.go#L1086)
    * [```(bi *baseIterator) Next() *Node```](https://github.com/apache/yunikorn-core/blob/a6558d4b7ecf4ca393954cf1ea3f354a2f9fb397/pkg/scheduler/objects/node_iterator.go#L56)

### where thie iterator come from ?

* [```(pc *PartitionContext) tryAllocate() *objects.Allocation```](https://github.com/apache/yunikorn-core/blob/a6558d4b7ecf4ca393954cf1ea3f354a2f9fb397/pkg/scheduler/partition.go#L825)
    * [```func (sq *Queue) TryAllocate(iterator func() NodeIterator) *Allocation```](https://github.com/apache/yunikorn-core/blob/a6558d4b7ecf4ca393954cf1ea3f354a2f9fb397/pkg/scheduler/objects/queue.go#L1064)
        * [```(sa *Application) tryAllocate(headRoom *resources.Resource, nodeIterator func() NodeIterator) *Allocation```](https://github.com/apache/yunikorn-core/blob/a6558d4b7ecf4ca393954cf1ea3f354a2f9fb397/pkg/scheduler/objects/application.go#L819)
            * [```(sa *Application) tryNodes(ask *AllocationAsk, iterator NodeIterator) *Allocation```](https://github.com/apache/yunikorn-core/blob/branch-1.0/pkg/scheduler/objects/application.go#L1086)
                * [```(bi *baseIterator) Next() *Node```](https://github.com/apache/yunikorn-core/blob/a6558d4b7ecf4ca393954cf1ea3f354a2f9fb397/pkg/scheduler/objects/node_iterator.go#L56)

```context.go```:
```go
func (pc *PartitionContext) tryAllocate() *objects.Allocation {
	if !resources.StrictlyGreaterThanZero(pc.root.GetPendingResource()) {
		// nothing to do just return
		return nil
	}
	// try allocating from the root down
	alloc := pc.root.TryAllocate(pc.GetNodeIterator)
	if alloc != nil {
		return pc.allocate(alloc)
	}
	return nil
}
```
