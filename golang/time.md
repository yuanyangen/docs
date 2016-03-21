Internally, the Go runtime uses a single priority queue and goroutine to manage timers. This supports (but is not limited to) the entirety of the time package. It normally obviates the need for a user-land time-ordered priority queue but it’s important to keep in mind that it’s a single data structure with a single lock, potentially impacting GOMAXPROCS > 1 performance. See runtime/time.goc.



http://golang.org/src/pkg/runtime/time.goc?s=1684:1787#L83
