# Controller介绍
## 接口定义
```go
type Controller interface {
	/*
	 译：做了两件事：
	（1）启动一个Reflector从Config的ListerWatcher中获取资源对象，并推送到Config的Queue中，可能会执行Resync操作来同步这个Queue   
	（2）反复的从Queue中获取资源对象并通过Config的ProcessFunc函数处理
	 一直做这两件事知道stopCh被调用关闭退出
	 */
	Run(stopCh <-chan struct{})
	HasSynced() bool
	LastSyncResourceVersion() string
}
```
## 结构体
```go
type controller struct {
	config         Config
	reflector      *Reflector
	reflectorMutex sync.RWMutex
	clock          clock.Clock
}
func New(c *Config) Controller {
    ctlr := &controller{
    config: *c,
    clock:  &clock.RealClock{},
    }
    return ctlr
}
```
可以看到这个controller是由config、reflector、reflectorMutex和clock组成，构造函数New返回了一个接口，需要传入config
### config
```go
type Config struct {
	Queue
	ListerWatcher
	Process ProcessFunc
	ObjectType runtime.Object
	FullResyncPeriod time.Duration
	ShouldResync ShouldResyncFunc
	RetryOnError bool
	WatchErrorHandler WatchErrorHandler
	WatchListPageSize int64
}
```
这个config中有几个比较重要的对象，分别看下：
- Queue 缓存队列，实现了Store,后面再看
- ListerWatcher 
```go
type ListerWatcher interface {
	Lister
	Watcher
}
type Lister interface {
    List(options metav1.ListOptions) (runtime.Object, error)
}
type Watcher interface {
    Watch(options metav1.ListOptions) (watch.Interface, error)
}
```
ListerWatcher 分别继承了Lister和Watcher接口
- ProcessFunc
### reflector
### reflectorMutex
### clock