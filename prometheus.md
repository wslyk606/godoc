
client_golang: github.com/prometheus/client_golang/prometheus Index | Examples | Files | Directories

package prometheus
import "github.com/prometheus/client_golang/prometheus"

Package prometheus is the core instrumentation package. It provides metrics primitives to instrument code for monitoring. It also offers a registry for metrics. Sub-packages allow to expose the registered metrics via HTTP (package promhttp) or push them to a Pushgateway (package push). There is also a sub-package promauto, which provides metrics constructors with automatic registration.

All exported functions and methods are safe to be used concurrently unless specified otherwise.

A Basic Example
As a starting point, a very basic usage example:

package main

import (
	"log"
	"net/http"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
	cpuTemp = prometheus.NewGauge(prometheus.GaugeOpts{
		Name: "cpu_temperature_celsius",
		Help: "Current temperature of the CPU.",
	})
	hdFailures = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "hd_errors_total",
			Help: "Number of hard-disk errors.",
		},
		[]string{"device"},
	)
)

func init() {
	// Metrics have to be registered to be exposed:
	prometheus.MustRegister(cpuTemp)
	prometheus.MustRegister(hdFailures)
}

func main() {
	cpuTemp.Set(65.3)
	hdFailures.With(prometheus.Labels{"device":"/dev/sda"}).Inc()

	// The Handler function provides a default handler to expose metrics
	// via an HTTP server. "/metrics" is the usual endpoint for that.
	http.Handle("/metrics", promhttp.Handler())
	log.Fatal(http.ListenAndServe(":8080", nil))
}
This is a complete program that exports two metrics, a Gauge and a Counter, the latter with a label attached to turn it into a (one-dimensional) vector.

Metrics
The number of exported identifiers in this package might appear a bit overwhelming. However, in addition to the basic plumbing shown in the example above, you only need to understand the different metric types and their vector versions for basic usage. Furthermore, if you are not concerned with fine-grained control of when and how to register metrics with the registry, have a look at the promauto package, which will effectively allow you to ignore registration altogether in simple cases.

Above, you have already touched the Counter and the Gauge. There are two more advanced metric types: the Summary and Histogram. A more thorough description of those four metric types can be found in the Prometheus docs: https://prometheus.io/docs/concepts/metric_types/

A fifth "type" of metric is Untyped. It behaves like a Gauge, but signals the Prometheus server not to assume anything about its type.

In addition to the fundamental metric types Gauge, Counter, Summary, Histogram, and Untyped, a very important part of the Prometheus data model is the partitioning of samples along dimensions called labels, which results in metric vectors. The fundamental types are GaugeVec, CounterVec, SummaryVec, HistogramVec, and UntypedVec.

While only the fundamental metric types implement the Metric interface, both the metrics and their vector versions implement the Collector interface. A Collector manages the collection of a number of Metrics, but for convenience, a Metric can also “collect itself”. Note that Gauge, Counter, Summary, Histogram, and Untyped are interfaces themselves while GaugeVec, CounterVec, SummaryVec, HistogramVec, and UntypedVec are not.

To create instances of Metrics and their vector versions, you need a suitable …Opts struct, i.e. GaugeOpts, CounterOpts, SummaryOpts, HistogramOpts, or UntypedOpts.

Custom Collectors and constant Metrics
While you could create your own implementations of Metric, most likely you will only ever implement the Collector interface on your own. At a first glance, a custom Collector seems handy to bundle Metrics for common registration (with the prime example of the different metric vectors above, which bundle all the metrics of the same name but with different labels).

There is a more involved use case, too: If you already have metrics available, created outside of the Prometheus context, you don't need the interface of the various Metric types. You essentially want to mirror the existing numbers into Prometheus Metrics during collection. An own implementation of the Collector interface is perfect for that. You can create Metric instances “on the fly” using NewConstMetric, NewConstHistogram, and NewConstSummary (and their respective Must… versions). That will happen in the Collect method. The Describe method has to return separate Desc instances, representative of the “throw-away” metrics to be created later. NewDesc comes in handy to create those Desc instances.

The Collector example illustrates the use case. You can also look at the source code of the processCollector (mirroring process metrics), the goCollector (mirroring Go metrics), or the expvarCollector (mirroring expvar metrics) as examples that are used in this package itself.

If you just need to call a function to get a single float value to collect as a metric, GaugeFunc, CounterFunc, or UntypedFunc might be interesting shortcuts.

Advanced Uses of the Registry
While MustRegister is the by far most common way of registering a Collector, sometimes you might want to handle the errors the registration might cause. As suggested by the name, MustRegister panics if an error occurs. With the Register function, the error is returned and can be handled.

An error is returned if the registered Collector is incompatible or inconsistent with already registered metrics. The registry aims for consistency of the collected metrics according to the Prometheus data model. Inconsistencies are ideally detected at registration time, not at collect time. The former will usually be detected at start-up time of a program, while the latter will only happen at scrape time, possibly not even on the first scrape if the inconsistency only becomes relevant later. That is the main reason why a Collector and a Metric have to describe themselves to the registry.

So far, everything we did operated on the so-called default registry, as it can be found in the global DefaultRegisterer variable. With NewRegistry, you can create a custom registry, or you can even implement the Registerer or Gatherer interfaces yourself. The methods Register and Unregister work in the same way on a custom registry as the global functions Register and Unregister on the default registry.

There are a number of uses for custom registries: You can use registries with special properties, see NewPedanticRegistry. You can avoid global state, as it is imposed by the DefaultRegisterer. You can use multiple registries at the same time to expose different metrics in different ways. You can use separate registries for testing purposes.

Also note that the DefaultRegisterer comes registered with a Collector for Go runtime metrics (via NewGoCollector) and a Collector for process metrics (via NewProcessCollector). With a custom registry, you are in control and decide yourself about the Collectors to register.

HTTP Exposition
The Registry implements the Gatherer interface. The caller of the Gather method can then expose the gathered metrics in some way. Usually, the metrics are served via HTTP on the /metrics endpoint. That's happening in the example above. The tools to expose metrics via HTTP are in the promhttp sub-package. (The top-level functions in the prometheus package are deprecated.)

Pushing to the Pushgateway
Function for pushing to the Pushgateway can be found in the push sub-package.

Graphite Bridge
Functions and examples to push metrics from a Gatherer to Graphite can be found in the graphite sub-package.

Other Means of Exposition
More ways of exposing metrics can easily be added by following the approaches of the existing implementations.

Index
Constants
Variables
func BuildFQName(namespace, subsystem, name string) string
func ExponentialBuckets(start, factor float64, count int) []float64
func Handler() http.Handler
func InstrumentHandler(handlerName string, handler http.Handler) http.HandlerFunc
func InstrumentHandlerFunc(handlerName string, handlerFunc func(http.ResponseWriter, *http.Request)) http.HandlerFunc
func InstrumentHandlerFuncWithOpts(opts SummaryOpts, handlerFunc func(http.ResponseWriter, *http.Request)) http.HandlerFunc
func InstrumentHandlerWithOpts(opts SummaryOpts, handler http.Handler) http.HandlerFunc
func LinearBuckets(start, width float64, count int) []float64
func MustRegister(cs ...Collector)
func Register(c Collector) error
func UninstrumentedHandler() http.Handler
func Unregister(c Collector) bool
type AlreadyRegisteredError
func (err AlreadyRegisteredError) Error() string
type Collector
func NewExpvarCollector(exports map[string]*Desc) Collector
func NewGoCollector() Collector
func NewProcessCollector(pid int, namespace string) Collector
func NewProcessCollectorPIDFn( pidFn func() (int, error), namespace string, ) Collector
type Counter
func NewCounter(opts CounterOpts) Counter
type CounterFunc
func NewCounterFunc(opts CounterOpts, function func() float64) CounterFunc
type CounterOpts
type CounterVec
func NewCounterVec(opts CounterOpts, labelNames []string) *CounterVec
func (v *CounterVec) CurryWith(labels Labels) (*CounterVec, error)
func (m CounterVec) Delete(labels Labels) bool
func (m CounterVec) DeleteLabelValues(lvs ...string) bool
func (v *CounterVec) GetMetricWith(labels Labels) (Counter, error)
func (v *CounterVec) GetMetricWithLabelValues(lvs ...string) (Counter, error)
func (v *CounterVec) MustCurryWith(labels Labels) *CounterVec
func (v *CounterVec) With(labels Labels) Counter
func (v *CounterVec) WithLabelValues(lvs ...string) Counter
type Desc
func NewDesc(fqName, help string, variableLabels []string, constLabels Labels) *Desc
func NewInvalidDesc(err error) *Desc
func (d *Desc) String() string
type Gatherer
type GathererFunc
func (gf GathererFunc) Gather() ([]*dto.MetricFamily, error)
type Gatherers
func (gs Gatherers) Gather() ([]*dto.MetricFamily, error)
type Gauge
func NewGauge(opts GaugeOpts) Gauge
type GaugeFunc
func NewGaugeFunc(opts GaugeOpts, function func() float64) GaugeFunc
type GaugeOpts
type GaugeVec
func NewGaugeVec(opts GaugeOpts, labelNames []string) *GaugeVec
func (v *GaugeVec) CurryWith(labels Labels) (*GaugeVec, error)
func (m GaugeVec) Delete(labels Labels) bool
func (m GaugeVec) DeleteLabelValues(lvs ...string) bool
func (v *GaugeVec) GetMetricWith(labels Labels) (Gauge, error)
func (v *GaugeVec) GetMetricWithLabelValues(lvs ...string) (Gauge, error)
func (v *GaugeVec) MustCurryWith(labels Labels) *GaugeVec
func (v *GaugeVec) With(labels Labels) Gauge
func (v *GaugeVec) WithLabelValues(lvs ...string) Gauge
type Histogram
func NewHistogram(opts HistogramOpts) Histogram
type HistogramOpts
type HistogramVec
func NewHistogramVec(opts HistogramOpts, labelNames []string) *HistogramVec
func (v *HistogramVec) CurryWith(labels Labels) (ObserverVec, error)
func (m HistogramVec) Delete(labels Labels) bool
func (m HistogramVec) DeleteLabelValues(lvs ...string) bool
func (v *HistogramVec) GetMetricWith(labels Labels) (Observer, error)
func (v *HistogramVec) GetMetricWithLabelValues(lvs ...string) (Observer, error)
func (v *HistogramVec) MustCurryWith(labels Labels) ObserverVec
func (v *HistogramVec) With(labels Labels) Observer
func (v *HistogramVec) WithLabelValues(lvs ...string) Observer
type LabelPairSorter
func (s LabelPairSorter) Len() int
func (s LabelPairSorter) Less(i, j int) bool
func (s LabelPairSorter) Swap(i, j int)
type Labels
type Metric
func MustNewConstHistogram( desc *Desc, count uint64, sum float64, buckets map[float64]uint64, labelValues ...string, ) Metric
func MustNewConstMetric(desc *Desc, valueType ValueType, value float64, labelValues ...string) Metric
func MustNewConstSummary( desc *Desc, count uint64, sum float64, quantiles map[float64]float64, labelValues ...string, ) Metric
func NewConstHistogram( desc *Desc, count uint64, sum float64, buckets map[float64]uint64, labelValues ...string, ) (Metric, error)
func NewConstMetric(desc *Desc, valueType ValueType, value float64, labelValues ...string) (Metric, error)
func NewConstSummary( desc *Desc, count uint64, sum float64, quantiles map[float64]float64, labelValues ...string, ) (Metric, error)
func NewInvalidMetric(desc *Desc, err error) Metric
type MultiError
func (errs *MultiError) Append(err error)
func (errs MultiError) Error() string
func (errs MultiError) MaybeUnwrap() error
type Observer
type ObserverFunc
func (f ObserverFunc) Observe(value float64)
type ObserverVec
type Opts
type Registerer
type Registry
func NewPedanticRegistry() *Registry
func NewRegistry() *Registry
func (r *Registry) Gather() ([]*dto.MetricFamily, error)
func (r *Registry) MustRegister(cs ...Collector)
func (r *Registry) Register(c Collector) error
func (r *Registry) Unregister(c Collector) bool
type Summary
func NewSummary(opts SummaryOpts) Summary
type SummaryOpts
type SummaryVec
func NewSummaryVec(opts SummaryOpts, labelNames []string) *SummaryVec
func (v *SummaryVec) CurryWith(labels Labels) (ObserverVec, error)
func (m SummaryVec) Delete(labels Labels) bool
func (m SummaryVec) DeleteLabelValues(lvs ...string) bool
func (v *SummaryVec) GetMetricWith(labels Labels) (Observer, error)
func (v *SummaryVec) GetMetricWithLabelValues(lvs ...string) (Observer, error)
func (v *SummaryVec) MustCurryWith(labels Labels) ObserverVec
func (v *SummaryVec) With(labels Labels) Observer
func (v *SummaryVec) WithLabelValues(lvs ...string) Observer
type Timer
func NewTimer(o Observer) *Timer
func (t *Timer) ObserveDuration()
type UntypedFunc
func NewUntypedFunc(opts UntypedOpts, function func() float64) UntypedFunc
type UntypedOpts
type ValueType
Examples
AlreadyRegisteredError
Collector
Counter
CounterVec
Gatherers
Gauge
GaugeFunc
GaugeVec
Histogram
InstrumentHandler
LabelPairSorter
NewConstHistogram
NewConstSummary
NewExpvarCollector
Register
Summary
SummaryVec
Timer
Timer (Complex)
Timer (Gauge)
Package Files

collector.go counter.go desc.go doc.go expvar_collector.go fnv.go gauge.go go_collector.go histogram.go http.go labels.go metric.go observer.go process_collector.go registry.go summary.go timer.go untyped.go value.go vec.go

Constants
const (
    // DefMaxAge is the default duration for which observations stay
    // relevant.
    DefMaxAge time.Duration = 10 * time.Minute
    // DefAgeBuckets is the default number of buckets used to calculate the
    // age of observations.
    DefAgeBuckets = 5
    // DefBufCap is the standard buffer size for collecting Summary observations.
    DefBufCap = 500
)
Default values for SummaryOpts.

Variables
var (
    DefaultRegisterer Registerer = defaultRegistry
    DefaultGatherer   Gatherer   = defaultRegistry
)
DefaultRegisterer and DefaultGatherer are the implementations of the Registerer and Gatherer interface a number of convenience functions in this package act on. Initially, both variables point to the same Registry, which has a process collector (currently on Linux only, see NewProcessCollector) and a Go collector (see NewGoCollector, in particular the note about stop-the-world implication with Go versions older than 1.9) already registered. This approach to keep default instances as global state mirrors the approach of other packages in the Go standard library. Note that there are caveats. Change the variables with caution and only if you understand the consequences. Users who want to avoid global state altogether should not use the convenience functions and act on custom instances instead.

var (
    DefBuckets = []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5, 10}
)
DefBuckets are the default Histogram buckets. The default buckets are tailored to broadly measure the response time (in seconds) of a network service. Most likely, however, you will be required to define buckets customized to your use case.

var (
    DefObjectives = map[float64]float64{0.5: 0.05, 0.9: 0.01, 0.99: 0.001}
)
DefObjectives are the default Summary quantile values.

Deprecated: DefObjectives will not be used as the default objectives in v0.10 of the library. The default Summary will have no quantiles then.

func BuildFQName
func BuildFQName(namespace, subsystem, name string) string
BuildFQName joins the given three name components by "_". Empty name components are ignored. If the name parameter itself is empty, an empty string is returned, no matter what. Metric implementations included in this library use this function internally to generate the fully-qualified metric name from the name component in their Opts. Users of the library will only need this function if they implement their own Metric or instantiate a Desc (with NewDesc) directly.

func ExponentialBuckets
func ExponentialBuckets(start, factor float64, count int) []float64
ExponentialBuckets creates 'count' buckets, where the lowest bucket has an upper bound of 'start' and each following bucket's upper bound is 'factor' times the previous bucket's upper bound. The final +Inf bucket is not counted and not included in the returned slice. The returned slice is meant to be used for the Buckets field of HistogramOpts.

The function panics if 'count' is 0 or negative, if 'start' is 0 or negative, or if 'factor' is less than or equal 1.

func Handler
func Handler() http.Handler
Handler returns an HTTP handler for the DefaultGatherer. It is already instrumented with InstrumentHandler (using "prometheus" as handler name).

Deprecated: Please note the issues described in the doc comment of InstrumentHandler. You might want to consider using promhttp.InstrumentedHandler instead.

func InstrumentHandler
func InstrumentHandler(handlerName string, handler http.Handler) http.HandlerFunc
InstrumentHandler wraps the given HTTP handler for instrumentation. It registers four metric collectors (if not already done) and reports HTTP metrics to the (newly or already) registered collectors: http_requests_total (CounterVec), http_request_duration_microseconds (Summary), http_request_size_bytes (Summary), http_response_size_bytes (Summary). Each has a constant label named "handler" with the provided handlerName as value. http_requests_total is a metric vector partitioned by HTTP method (label name "method") and HTTP status code (label name "code").

Deprecated: InstrumentHandler has several issues. Use the tooling provided in package promhttp instead. The issues are the following:

- It uses Summaries rather than Histograms. Summaries are not useful if aggregation across multiple instances is required.

- It uses microseconds as unit, which is deprecated and should be replaced by seconds.

- The size of the request is calculated in a separate goroutine. Since this calculator requires access to the request header, it creates a race with any writes to the header performed during request handling. httputil.ReverseProxy is a prominent example for a handler performing such writes.

- It has additional issues with HTTP/2, cf. https://github.com/prometheus/client_golang/issues/272.

Example
func InstrumentHandlerFunc
func InstrumentHandlerFunc(handlerName string, handlerFunc func(http.ResponseWriter, *http.Request)) http.HandlerFunc
InstrumentHandlerFunc wraps the given function for instrumentation. It otherwise works in the same way as InstrumentHandler (and shares the same issues).

Deprecated: InstrumentHandlerFunc is deprecated for the same reasons as InstrumentHandler is. Use the tooling provided in package promhttp instead.

func InstrumentHandlerFuncWithOpts
func InstrumentHandlerFuncWithOpts(opts SummaryOpts, handlerFunc func(http.ResponseWriter, *http.Request)) http.HandlerFunc
InstrumentHandlerFuncWithOpts works like InstrumentHandlerFunc (and shares the same issues) but provides more flexibility (at the cost of a more complex call syntax). See InstrumentHandlerWithOpts for details how the provided SummaryOpts are used.

Deprecated: InstrumentHandlerFuncWithOpts is deprecated for the same reasons as InstrumentHandler is. Use the tooling provided in package promhttp instead.

func InstrumentHandlerWithOpts
func InstrumentHandlerWithOpts(opts SummaryOpts, handler http.Handler) http.HandlerFunc
InstrumentHandlerWithOpts works like InstrumentHandler (and shares the same issues) but provides more flexibility (at the cost of a more complex call syntax). As InstrumentHandler, this function registers four metric collectors, but it uses the provided SummaryOpts to create them. However, the fields "Name" and "Help" in the SummaryOpts are ignored. "Name" is replaced by "requests_total", "request_duration_microseconds", "request_size_bytes", and "response_size_bytes", respectively. "Help" is replaced by an appropriate help string. The names of the variable labels of the http_requests_total CounterVec are "method" (get, post, etc.), and "code" (HTTP status code).

If InstrumentHandlerWithOpts is called as follows, it mimics exactly the behavior of InstrumentHandler:

prometheus.InstrumentHandlerWithOpts(
    prometheus.SummaryOpts{
         Subsystem:   "http",
         ConstLabels: prometheus.Labels{"handler": handlerName},
    },
    handler,
)
Technical detail: "requests_total" is a CounterVec, not a SummaryVec, so it cannot use SummaryOpts. Instead, a CounterOpts struct is created internally, and all its fields are set to the equally named fields in the provided SummaryOpts.

Deprecated: InstrumentHandlerWithOpts is deprecated for the same reasons as InstrumentHandler is. Use the tooling provided in package promhttp instead.

func LinearBuckets
func LinearBuckets(start, width float64, count int) []float64
LinearBuckets creates 'count' buckets, each 'width' wide, where the lowest bucket has an upper bound of 'start'. The final +Inf bucket is not counted and not included in the returned slice. The returned slice is meant to be used for the Buckets field of HistogramOpts.

The function panics if 'count' is zero or negative.

func MustRegister
func MustRegister(cs ...Collector)
MustRegister registers the provided Collectors with the DefaultRegisterer and panics if any error occurs.

MustRegister is a shortcut for DefaultRegisterer.MustRegister(cs...). See there for more details.

func Register
func Register(c Collector) error
Register registers the provided Collector with the DefaultRegisterer.

Register is a shortcut for DefaultRegisterer.Register(c). See there for more details.

Example
func UninstrumentedHandler
func UninstrumentedHandler() http.Handler
UninstrumentedHandler returns an HTTP handler for the DefaultGatherer.

Deprecated: Use promhttp.Handler instead. See there for further documentation.

func Unregister
func Unregister(c Collector) bool
Unregister removes the registration of the provided Collector from the DefaultRegisterer.

Unregister is a shortcut for DefaultRegisterer.Unregister(c). See there for more details.

type AlreadyRegisteredError
type AlreadyRegisteredError struct {
    ExistingCollector, NewCollector Collector
}
AlreadyRegisteredError is returned by the Register method if the Collector to be registered has already been registered before, or a different Collector that collects the same metrics has been registered before. Registration fails in that case, but you can detect from the kind of error what has happened. The error contains fields for the existing Collector and the (rejected) new Collector that equals the existing one. This can be used to find out if an equal Collector has been registered before and switch over to using the old one, as demonstrated in the example.

Example
func (AlreadyRegisteredError) Error
func (err AlreadyRegisteredError) Error() string
type Collector
type Collector interface {
    // Describe sends the super-set of all possible descriptors of metrics
    // collected by this Collector to the provided channel and returns once
    // the last descriptor has been sent. The sent descriptors fulfill the
    // consistency and uniqueness requirements described in the Desc
    // documentation. (It is valid if one and the same Collector sends
    // duplicate descriptors. Those duplicates are simply ignored. However,
    // two different Collectors must not send duplicate descriptors.) This
    // method idempotently sends the same descriptors throughout the
    // lifetime of the Collector. If a Collector encounters an error while
    // executing this method, it must send an invalid descriptor (created
    // with NewInvalidDesc) to signal the error to the registry.
    Describe(chan<- *Desc)
    // Collect is called by the Prometheus registry when collecting
    // metrics. The implementation sends each collected metric via the
    // provided channel and returns once the last metric has been sent. The
    // descriptor of each sent metric is one of those returned by
    // Describe. Returned metrics that share the same descriptor must differ
    // in their variable label values. This method may be called
    // concurrently and must therefore be implemented in a concurrency safe
    // way. Blocking occurs at the expense of total performance of rendering
    // all registered metrics. Ideally, Collector implementations support
    // concurrent readers.
    Collect(chan<- Metric)
}
Collector is the interface implemented by anything that can be used by Prometheus to collect metrics. A Collector has to be registered for collection. See Registerer.Register.

The stock metrics provided by this package (Gauge, Counter, Summary, Histogram, Untyped) are also Collectors (which only ever collect one metric, namely itself). An implementer of Collector may, however, collect multiple metrics in a coordinated fashion and/or create metrics on the fly. Examples for collectors already implemented in this library are the metric vectors (i.e. collection of multiple instances of the same Metric but with different label values) like GaugeVec or SummaryVec, and the ExpvarCollector.

Example
func NewExpvarCollector
func NewExpvarCollector(exports map[string]*Desc) Collector
NewExpvarCollector returns a newly allocated expvar Collector that still has to be registered with a Prometheus registry.

An expvar Collector collects metrics from the expvar interface. It provides a quick way to expose numeric values that are already exported via expvar as Prometheus metrics. Note that the data models of expvar and Prometheus are fundamentally different, and that the expvar Collector is inherently slower than native Prometheus metrics. Thus, the expvar Collector is probably great for experiments and prototying, but you should seriously consider a more direct implementation of Prometheus metrics for monitoring production systems.

The exports map has the following meaning:

The keys in the map correspond to expvar keys, i.e. for every expvar key you want to export as Prometheus metric, you need an entry in the exports map. The descriptor mapped to each key describes how to export the expvar value. It defines the name and the help string of the Prometheus metric proxying the expvar value. The type will always be Untyped.

For descriptors without variable labels, the expvar value must be a number or a bool. The number is then directly exported as the Prometheus sample value. (For a bool, 'false' translates to 0 and 'true' to 1). Expvar values that are not numbers or bools are silently ignored.

If the descriptor has one variable label, the expvar value must be an expvar map. The keys in the expvar map become the various values of the one Prometheus label. The values in the expvar map must be numbers or bools again as above.

For descriptors with more than one variable label, the expvar must be a nested expvar map, i.e. where the values of the topmost map are maps again etc. until a depth is reached that corresponds to the number of labels. The leaves of that structure must be numbers or bools as above to serve as the sample values.

Anything that does not fit into the scheme above is silently ignored.

Example
func NewGoCollector
func NewGoCollector() Collector
NewGoCollector returns a collector which exports metrics about the current Go process. This includes memory stats. To collect those, runtime.ReadMemStats is called. This causes a stop-the-world, which is very short with Go1.9+ (~25µs). However, with older Go versions, the stop-the-world duration depends on the heap size and can be quite significant (~1.7 ms/GiB as per https://go-review.googlesource.com/c/go/+/34937).

func NewProcessCollector
func NewProcessCollector(pid int, namespace string) Collector
NewProcessCollector returns a collector which exports the current state of process metrics including CPU, memory and file descriptor usage as well as the process start time for the given process ID under the given namespace.

Currently, the collector depends on a Linux-style proc filesystem and therefore only exports metrics for Linux.

func NewProcessCollectorPIDFn
func NewProcessCollectorPIDFn(
    pidFn func() (int, error),
    namespace string,
) Collector
NewProcessCollectorPIDFn works like NewProcessCollector but the process ID is determined on each collect anew by calling the given pidFn function.

type Counter
type Counter interface {
    Metric
    Collector

    // Inc increments the counter by 1. Use Add to increment it by arbitrary
    // non-negative values.
    Inc()
    // Add adds the given value to the counter. It panics if the value is <
    // 0.
    Add(float64)
}
Counter is a Metric that represents a single numerical value that only ever goes up. That implies that it cannot be used to count items whose number can also go down, e.g. the number of currently running goroutines. Those "counters" are represented by Gauges.

A Counter is typically used to count requests served, tasks completed, errors occurred, etc.

To create Counter instances, use NewCounter.

Example
func NewCounter
func NewCounter(opts CounterOpts) Counter
NewCounter creates a new Counter based on the provided CounterOpts.

The returned implementation tracks the counter value in two separate variables, a float64 and a uint64. The latter is used to track calls of the Inc method and calls of the Add method with a value that can be represented as a uint64. This allows atomic increments of the counter with optimal performance. (It is common to have an Inc call in very hot execution paths.) Both internal tracking values are added up in the Write method. This has to be taken into account when it comes to precision and overflow behavior.

type CounterFunc
type CounterFunc interface {
    Metric
    Collector
}
CounterFunc is a Counter whose value is determined at collect time by calling a provided function.

To create CounterFunc instances, use NewCounterFunc.

func NewCounterFunc
func NewCounterFunc(opts CounterOpts, function func() float64) CounterFunc
NewCounterFunc creates a new CounterFunc based on the provided CounterOpts. The value reported is determined by calling the given function from within the Write method. Take into account that metric collection may happen concurrently. If that results in concurrent calls to Write, like in the case where a CounterFunc is directly registered with Prometheus, the provided function must be concurrency-safe. The function should also honor the contract for a Counter (values only go up, not down), but compliance will not be checked.

type CounterOpts
type CounterOpts Opts
CounterOpts is an alias for Opts. See there for doc comments.

type CounterVec
type CounterVec struct {
    // contains filtered or unexported fields
}
CounterVec is a Collector that bundles a set of Counters that all share the same Desc, but have different values for their variable labels. This is used if you want to count the same thing partitioned by various dimensions (e.g. number of HTTP requests, partitioned by response code and method). Create instances with NewCounterVec.

Example
func NewCounterVec
func NewCounterVec(opts CounterOpts, labelNames []string) *CounterVec
NewCounterVec creates a new CounterVec based on the provided CounterOpts and partitioned by the given label names.

func (*CounterVec) CurryWith
func (v *CounterVec) CurryWith(labels Labels) (*CounterVec, error)
CurryWith returns a vector curried with the provided labels, i.e. the returned vector has those labels pre-set for all labeled operations performed on it. The cardinality of the curried vector is reduced accordingly. The order of the remaining labels stays the same (just with the curried labels taken out of the sequence – which is relevant for the (GetMetric)WithLabelValues methods). It is possible to curry a curried vector, but only with labels not yet used for currying before.

The metrics contained in the CounterVec are shared between the curried and uncurried vectors. They are just accessed differently. Curried and uncurried vectors behave identically in terms of collection. Only one must be registered with a given registry (usually the uncurried version). The Reset method deletes all metrics, even if called on a curried vector.

func (CounterVec) Delete
func (m CounterVec) Delete(labels Labels) bool
Delete deletes the metric where the variable labels are the same as those passed in as labels. It returns true if a metric was deleted.

It is not an error if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc. However, such inconsistent Labels can never match an actual metric, so the method will always return false in that case.

This method is used for the same purpose as DeleteLabelValues(...string). See there for pros and cons of the two methods.

func (CounterVec) DeleteLabelValues
func (m CounterVec) DeleteLabelValues(lvs ...string) bool
DeleteLabelValues removes the metric where the variable labels are the same as those passed in as labels (same order as the VariableLabels in Desc). It returns true if a metric was deleted.

It is not an error if the number of label values is not the same as the number of VariableLabels in Desc. However, such inconsistent label count can never match an actual metric, so the method will always return false in that case.

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider Delete(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map). See also the CounterVec example.

func (*CounterVec) GetMetricWith
func (v *CounterVec) GetMetricWith(labels Labels) (Counter, error)
GetMetricWith returns the Counter for the given Labels map (the label names must match those of the VariableLabels in Desc). If that label map is accessed for the first time, a new Counter is created. Implications of creating a Counter without using it and keeping the Counter for later use are the same as for GetMetricWithLabelValues.

An error is returned if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc (minus any curried labels).

This method is used for the same purpose as GetMetricWithLabelValues(...string). See there for pros and cons of the two methods.

func (*CounterVec) GetMetricWithLabelValues
func (v *CounterVec) GetMetricWithLabelValues(lvs ...string) (Counter, error)
GetMetricWithLabelValues returns the Counter for the given slice of label values (same order as the VariableLabels in Desc). If that combination of label values is accessed for the first time, a new Counter is created.

It is possible to call this method without using the returned Counter to only create the new Counter but leave it at its starting value 0. See also the SummaryVec example.

Keeping the Counter for later use is possible (and should be considered if performance is critical), but keep in mind that Reset, DeleteLabelValues and Delete can be used to delete the Counter from the CounterVec. In that case, the Counter will still exist, but it will not be exported anymore, even if a Counter with the same label values is created later.

An error is returned if the number of label values is not the same as the number of VariableLabels in Desc (minus any curried labels).

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider GetMetricWith(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map). See also the GaugeVec example.

func (*CounterVec) MustCurryWith
func (v *CounterVec) MustCurryWith(labels Labels) *CounterVec
MustCurryWith works as CurryWith but panics where CurryWith would have returned an error.

func (*CounterVec) With
func (v *CounterVec) With(labels Labels) Counter
With works as GetMetricWith, but panics where GetMetricWithLabels would have returned an error. Not returning an error allows shortcuts like

myVec.With(prometheus.Labels{"code": "404", "method": "GET"}).Add(42)
func (*CounterVec) WithLabelValues
func (v *CounterVec) WithLabelValues(lvs ...string) Counter
WithLabelValues works as GetMetricWithLabelValues, but panics where GetMetricWithLabelValues would have returned an error. Not returning an error allows shortcuts like

myVec.WithLabelValues("404", "GET").Add(42)
type Desc
type Desc struct {
    // contains filtered or unexported fields
}
Desc is the descriptor used by every Prometheus Metric. It is essentially the immutable meta-data of a Metric. The normal Metric implementations included in this package manage their Desc under the hood. Users only have to deal with Desc if they use advanced features like the ExpvarCollector or custom Collectors and Metrics.

Descriptors registered with the same registry have to fulfill certain consistency and uniqueness criteria if they share the same fully-qualified name: They must have the same help string and the same label names (aka label dimensions) in each, constLabels and variableLabels, but they must differ in the values of the constLabels.

Descriptors that share the same fully-qualified names and the same label values of their constLabels are considered equal.

Use NewDesc to create new Desc instances.

func NewDesc
func NewDesc(fqName, help string, variableLabels []string, constLabels Labels) *Desc
NewDesc allocates and initializes a new Desc. Errors are recorded in the Desc and will be reported on registration time. variableLabels and constLabels can be nil if no such labels should be set. fqName and help must not be empty.

variableLabels only contain the label names. Their label values are variable and therefore not part of the Desc. (They are managed within the Metric.)

For constLabels, the label values are constant. Therefore, they are fully specified in the Desc. See the Collector example for a usage pattern.

func NewInvalidDesc
func NewInvalidDesc(err error) *Desc
NewInvalidDesc returns an invalid descriptor, i.e. a descriptor with the provided error set. If a collector returning such a descriptor is registered, registration will fail with the provided error. NewInvalidDesc can be used by a Collector to signal inability to describe itself.

func (*Desc) String
func (d *Desc) String() string
type Gatherer
type Gatherer interface {
    // Gather calls the Collect method of the registered Collectors and then
    // gathers the collected metrics into a lexicographically sorted slice
    // of uniquely named MetricFamily protobufs. Gather ensures that the
    // returned slice is valid and self-consistent so that it can be used
    // for valid exposition. As an exception to the strict consistency
    // requirements described for metric.Desc, Gather will tolerate
    // different sets of label names for metrics of the same metric family.
    //
    // Even if an error occurs, Gather attempts to gather as many metrics as
    // possible. Hence, if a non-nil error is returned, the returned
    // MetricFamily slice could be nil (in case of a fatal error that
    // prevented any meaningful metric collection) or contain a number of
    // MetricFamily protobufs, some of which might be incomplete, and some
    // might be missing altogether. The returned error (which might be a
    // MultiError) explains the details. Note that this is mostly useful for
    // debugging purposes. If the gathered protobufs are to be used for
    // exposition in actual monitoring, it is almost always better to not
    // expose an incomplete result and instead disregard the returned
    // MetricFamily protobufs in case the returned error is non-nil.
    Gather() ([]*dto.MetricFamily, error)
}
Gatherer is the interface for the part of a registry in charge of gathering the collected metrics into a number of MetricFamilies. The Gatherer interface comes with the same general implication as described for the Registerer interface.

type GathererFunc
type GathererFunc func() ([]*dto.MetricFamily, error)
GathererFunc turns a function into a Gatherer.

func (GathererFunc) Gather
func (gf GathererFunc) Gather() ([]*dto.MetricFamily, error)
Gather implements Gatherer.

type Gatherers
type Gatherers []Gatherer
Gatherers is a slice of Gatherer instances that implements the Gatherer interface itself. Its Gather method calls Gather on all Gatherers in the slice in order and returns the merged results. Errors returned from the Gather calles are all returned in a flattened MultiError. Duplicate and inconsistent Metrics are skipped (first occurrence in slice order wins) and reported in the returned error.

Gatherers can be used to merge the Gather results from multiple Registries. It also provides a way to directly inject existing MetricFamily protobufs into the gathering by creating a custom Gatherer with a Gather method that simply returns the existing MetricFamily protobufs. Note that no registration is involved (in contrast to Collector registration), so obviously registration-time checks cannot happen. Any inconsistencies between the gathered MetricFamilies are reported as errors by the Gather method, and inconsistent Metrics are dropped. Invalid parts of the MetricFamilies (e.g. syntactically invalid metric or label names) will go undetected.

Example
func (Gatherers) Gather
func (gs Gatherers) Gather() ([]*dto.MetricFamily, error)
Gather implements Gatherer.

type Gauge
type Gauge interface {
    Metric
    Collector

    // Set sets the Gauge to an arbitrary value.
    Set(float64)
    // Inc increments the Gauge by 1. Use Add to increment it by arbitrary
    // values.
    Inc()
    // Dec decrements the Gauge by 1. Use Sub to decrement it by arbitrary
    // values.
    Dec()
    // Add adds the given value to the Gauge. (The value can be negative,
    // resulting in a decrease of the Gauge.)
    Add(float64)
    // Sub subtracts the given value from the Gauge. (The value can be
    // negative, resulting in an increase of the Gauge.)
    Sub(float64)

    // SetToCurrentTime sets the Gauge to the current Unix time in seconds.
    SetToCurrentTime()
}
Gauge is a Metric that represents a single numerical value that can arbitrarily go up and down.

A Gauge is typically used for measured values like temperatures or current memory usage, but also "counts" that can go up and down, like the number of running goroutines.

To create Gauge instances, use NewGauge.

Example
func NewGauge
func NewGauge(opts GaugeOpts) Gauge
NewGauge creates a new Gauge based on the provided GaugeOpts.

The returned implementation is optimized for a fast Set method. If you have a choice for managing the value of a Gauge via Set vs. Inc/Dec/Add/Sub, pick the former. For example, the Inc method of the returned Gauge is slower than the Inc method of a Counter returned by NewCounter. This matches the typical scenarios for Gauges and Counters, where the former tends to be Set-heavy and the latter Inc-heavy.

type GaugeFunc
type GaugeFunc interface {
    Metric
    Collector
}
GaugeFunc is a Gauge whose value is determined at collect time by calling a provided function.

To create GaugeFunc instances, use NewGaugeFunc.

Example
func NewGaugeFunc
func NewGaugeFunc(opts GaugeOpts, function func() float64) GaugeFunc
NewGaugeFunc creates a new GaugeFunc based on the provided GaugeOpts. The value reported is determined by calling the given function from within the Write method. Take into account that metric collection may happen concurrently. If that results in concurrent calls to Write, like in the case where a GaugeFunc is directly registered with Prometheus, the provided function must be concurrency-safe.

type GaugeOpts
type GaugeOpts Opts
GaugeOpts is an alias for Opts. See there for doc comments.

type GaugeVec
type GaugeVec struct {
    // contains filtered or unexported fields
}
GaugeVec is a Collector that bundles a set of Gauges that all share the same Desc, but have different values for their variable labels. This is used if you want to count the same thing partitioned by various dimensions (e.g. number of operations queued, partitioned by user and operation type). Create instances with NewGaugeVec.

Example
func NewGaugeVec
func NewGaugeVec(opts GaugeOpts, labelNames []string) *GaugeVec
NewGaugeVec creates a new GaugeVec based on the provided GaugeOpts and partitioned by the given label names.

func (*GaugeVec) CurryWith
func (v *GaugeVec) CurryWith(labels Labels) (*GaugeVec, error)
CurryWith returns a vector curried with the provided labels, i.e. the returned vector has those labels pre-set for all labeled operations performed on it. The cardinality of the curried vector is reduced accordingly. The order of the remaining labels stays the same (just with the curried labels taken out of the sequence – which is relevant for the (GetMetric)WithLabelValues methods). It is possible to curry a curried vector, but only with labels not yet used for currying before.

The metrics contained in the GaugeVec are shared between the curried and uncurried vectors. They are just accessed differently. Curried and uncurried vectors behave identically in terms of collection. Only one must be registered with a given registry (usually the uncurried version). The Reset method deletes all metrics, even if called on a curried vector.

func (GaugeVec) Delete
func (m GaugeVec) Delete(labels Labels) bool
Delete deletes the metric where the variable labels are the same as those passed in as labels. It returns true if a metric was deleted.

It is not an error if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc. However, such inconsistent Labels can never match an actual metric, so the method will always return false in that case.

This method is used for the same purpose as DeleteLabelValues(...string). See there for pros and cons of the two methods.

func (GaugeVec) DeleteLabelValues
func (m GaugeVec) DeleteLabelValues(lvs ...string) bool
DeleteLabelValues removes the metric where the variable labels are the same as those passed in as labels (same order as the VariableLabels in Desc). It returns true if a metric was deleted.

It is not an error if the number of label values is not the same as the number of VariableLabels in Desc. However, such inconsistent label count can never match an actual metric, so the method will always return false in that case.

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider Delete(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map). See also the CounterVec example.

func (*GaugeVec) GetMetricWith
func (v *GaugeVec) GetMetricWith(labels Labels) (Gauge, error)
GetMetricWith returns the Gauge for the given Labels map (the label names must match those of the VariableLabels in Desc). If that label map is accessed for the first time, a new Gauge is created. Implications of creating a Gauge without using it and keeping the Gauge for later use are the same as for GetMetricWithLabelValues.

An error is returned if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc (minus any curried labels).

This method is used for the same purpose as GetMetricWithLabelValues(...string). See there for pros and cons of the two methods.

func (*GaugeVec) GetMetricWithLabelValues
func (v *GaugeVec) GetMetricWithLabelValues(lvs ...string) (Gauge, error)
GetMetricWithLabelValues returns the Gauge for the given slice of label values (same order as the VariableLabels in Desc). If that combination of label values is accessed for the first time, a new Gauge is created.

It is possible to call this method without using the returned Gauge to only create the new Gauge but leave it at its starting value 0. See also the SummaryVec example.

Keeping the Gauge for later use is possible (and should be considered if performance is critical), but keep in mind that Reset, DeleteLabelValues and Delete can be used to delete the Gauge from the GaugeVec. In that case, the Gauge will still exist, but it will not be exported anymore, even if a Gauge with the same label values is created later. See also the CounterVec example.

An error is returned if the number of label values is not the same as the number of VariableLabels in Desc (minus any curried labels).

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider GetMetricWith(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map).

func (*GaugeVec) MustCurryWith
func (v *GaugeVec) MustCurryWith(labels Labels) *GaugeVec
MustCurryWith works as CurryWith but panics where CurryWith would have returned an error.

func (*GaugeVec) With
func (v *GaugeVec) With(labels Labels) Gauge
With works as GetMetricWith, but panics where GetMetricWithLabels would have returned an error. Not returning an error allows shortcuts like

myVec.With(prometheus.Labels{"code": "404", "method": "GET"}).Add(42)
func (*GaugeVec) WithLabelValues
func (v *GaugeVec) WithLabelValues(lvs ...string) Gauge
WithLabelValues works as GetMetricWithLabelValues, but panics where GetMetricWithLabelValues would have returned an error. Not returning an error allows shortcuts like

myVec.WithLabelValues("404", "GET").Add(42)
type Histogram
type Histogram interface {
    Metric
    Collector

    // Observe adds a single observation to the histogram.
    Observe(float64)
}
A Histogram counts individual observations from an event or sample stream in configurable buckets. Similar to a summary, it also provides a sum of observations and an observation count.

On the Prometheus server, quantiles can be calculated from a Histogram using the histogram_quantile function in the query language.

Note that Histograms, in contrast to Summaries, can be aggregated with the Prometheus query language (see the documentation for detailed procedures). However, Histograms require the user to pre-define suitable buckets, and they are in general less accurate. The Observe method of a Histogram has a very low performance overhead in comparison with the Observe method of a Summary.

To create Histogram instances, use NewHistogram.

Example
func NewHistogram
func NewHistogram(opts HistogramOpts) Histogram
NewHistogram creates a new Histogram based on the provided HistogramOpts. It panics if the buckets in HistogramOpts are not in strictly increasing order.

type HistogramOpts
type HistogramOpts struct {
    // Namespace, Subsystem, and Name are components of the fully-qualified
    // name of the Histogram (created by joining these components with
    // "_"). Only Name is mandatory, the others merely help structuring the
    // name. Note that the fully-qualified name of the Histogram must be a
    // valid Prometheus metric name.
    Namespace string
    Subsystem string
    Name      string

    // Help provides information about this Histogram. Mandatory!
    //
    // Metrics with the same fully-qualified name must have the same Help
    // string.
    Help string

    // ConstLabels are used to attach fixed labels to this metric. Metrics
    // with the same fully-qualified name must have the same label names in
    // their ConstLabels.
    //
    // ConstLabels are only used rarely. In particular, do not use them to
    // attach the same labels to all your metrics. Those use cases are
    // better covered by target labels set by the scraping Prometheus
    // server, or by one specific metric (e.g. a build_info or a
    // machine_role metric). See also
    // https://prometheus.io/docs/instrumenting/writing_exporters/#target-labels,-not-static-scraped-labels
    ConstLabels Labels

    // Buckets defines the buckets into which observations are counted. Each
    // element in the slice is the upper inclusive bound of a bucket. The
    // values must be sorted in strictly increasing order. There is no need
    // to add a highest bucket with +Inf bound, it will be added
    // implicitly. The default value is DefBuckets.
    Buckets []float64
}
HistogramOpts bundles the options for creating a Histogram metric. It is mandatory to set Name and Help to a non-empty string. All other fields are optional and can safely be left at their zero value.

type HistogramVec
type HistogramVec struct {
    // contains filtered or unexported fields
}
HistogramVec is a Collector that bundles a set of Histograms that all share the same Desc, but have different values for their variable labels. This is used if you want to count the same thing partitioned by various dimensions (e.g. HTTP request latencies, partitioned by status code and method). Create instances with NewHistogramVec.

func NewHistogramVec
func NewHistogramVec(opts HistogramOpts, labelNames []string) *HistogramVec
NewHistogramVec creates a new HistogramVec based on the provided HistogramOpts and partitioned by the given label names.

func (*HistogramVec) CurryWith
func (v *HistogramVec) CurryWith(labels Labels) (ObserverVec, error)
CurryWith returns a vector curried with the provided labels, i.e. the returned vector has those labels pre-set for all labeled operations performed on it. The cardinality of the curried vector is reduced accordingly. The order of the remaining labels stays the same (just with the curried labels taken out of the sequence – which is relevant for the (GetMetric)WithLabelValues methods). It is possible to curry a curried vector, but only with labels not yet used for currying before.

The metrics contained in the HistogramVec are shared between the curried and uncurried vectors. They are just accessed differently. Curried and uncurried vectors behave identically in terms of collection. Only one must be registered with a given registry (usually the uncurried version). The Reset method deletes all metrics, even if called on a curried vector.

func (HistogramVec) Delete
func (m HistogramVec) Delete(labels Labels) bool
Delete deletes the metric where the variable labels are the same as those passed in as labels. It returns true if a metric was deleted.

It is not an error if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc. However, such inconsistent Labels can never match an actual metric, so the method will always return false in that case.

This method is used for the same purpose as DeleteLabelValues(...string). See there for pros and cons of the two methods.

func (HistogramVec) DeleteLabelValues
func (m HistogramVec) DeleteLabelValues(lvs ...string) bool
DeleteLabelValues removes the metric where the variable labels are the same as those passed in as labels (same order as the VariableLabels in Desc). It returns true if a metric was deleted.

It is not an error if the number of label values is not the same as the number of VariableLabels in Desc. However, such inconsistent label count can never match an actual metric, so the method will always return false in that case.

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider Delete(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map). See also the CounterVec example.

func (*HistogramVec) GetMetricWith
func (v *HistogramVec) GetMetricWith(labels Labels) (Observer, error)
GetMetricWith returns the Histogram for the given Labels map (the label names must match those of the VariableLabels in Desc). If that label map is accessed for the first time, a new Histogram is created. Implications of creating a Histogram without using it and keeping the Histogram for later use are the same as for GetMetricWithLabelValues.

An error is returned if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc (minus any curried labels).

This method is used for the same purpose as GetMetricWithLabelValues(...string). See there for pros and cons of the two methods.

func (*HistogramVec) GetMetricWithLabelValues
func (v *HistogramVec) GetMetricWithLabelValues(lvs ...string) (Observer, error)
GetMetricWithLabelValues returns the Histogram for the given slice of label values (same order as the VariableLabels in Desc). If that combination of label values is accessed for the first time, a new Histogram is created.

It is possible to call this method without using the returned Histogram to only create the new Histogram but leave it at its starting value, a Histogram without any observations.

Keeping the Histogram for later use is possible (and should be considered if performance is critical), but keep in mind that Reset, DeleteLabelValues and Delete can be used to delete the Histogram from the HistogramVec. In that case, the Histogram will still exist, but it will not be exported anymore, even if a Histogram with the same label values is created later. See also the CounterVec example.

An error is returned if the number of label values is not the same as the number of VariableLabels in Desc (minus any curried labels).

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider GetMetricWith(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map). See also the GaugeVec example.

func (*HistogramVec) MustCurryWith
func (v *HistogramVec) MustCurryWith(labels Labels) ObserverVec
MustCurryWith works as CurryWith but panics where CurryWith would have returned an error.

func (*HistogramVec) With
func (v *HistogramVec) With(labels Labels) Observer
With works as GetMetricWith but panics where GetMetricWithLabels would have returned an error. Not returning an error allows shortcuts like

myVec.With(prometheus.Labels{"code": "404", "method": "GET"}).Observe(42.21)
func (*HistogramVec) WithLabelValues
func (v *HistogramVec) WithLabelValues(lvs ...string) Observer
WithLabelValues works as GetMetricWithLabelValues, but panics where GetMetricWithLabelValues would have returned an error. Not returning an error allows shortcuts like

myVec.WithLabelValues("404", "GET").Observe(42.21)
type LabelPairSorter
type LabelPairSorter []*dto.LabelPair
LabelPairSorter implements sort.Interface. It is used to sort a slice of dto.LabelPair pointers. This is useful for implementing the Write method of custom metrics.

Example
func (LabelPairSorter) Len
func (s LabelPairSorter) Len() int
func (LabelPairSorter) Less
func (s LabelPairSorter) Less(i, j int) bool
func (LabelPairSorter) Swap
func (s LabelPairSorter) Swap(i, j int)
type Labels
type Labels map[string]string
Labels represents a collection of label name -> value mappings. This type is commonly used with the With(Labels) and GetMetricWith(Labels) methods of metric vector Collectors, e.g.:

myVec.With(Labels{"code": "404", "method": "GET"}).Add(42)
The other use-case is the specification of constant label pairs in Opts or to create a Desc.

type Metric
type Metric interface {
    // Desc returns the descriptor for the Metric. This method idempotently
    // returns the same descriptor throughout the lifetime of the
    // Metric. The returned descriptor is immutable by contract. A Metric
    // unable to describe itself must return an invalid descriptor (created
    // with NewInvalidDesc).
    Desc() *Desc
    // Write encodes the Metric into a "Metric" Protocol Buffer data
    // transmission object.
    //
    // Metric implementations must observe concurrency safety as reads of
    // this metric may occur at any time, and any blocking occurs at the
    // expense of total performance of rendering all registered
    // metrics. Ideally, Metric implementations should support concurrent
    // readers.
    //
    // While populating dto.Metric, it is the responsibility of the
    // implementation to ensure validity of the Metric protobuf (like valid
    // UTF-8 strings or syntactically valid metric and label names). It is
    // recommended to sort labels lexicographically. (Implementers may find
    // LabelPairSorter useful for that.) Callers of Write should still make
    // sure of sorting if they depend on it.
    Write(*dto.Metric) error
}
A Metric models a single sample value with its meta data being exported to Prometheus. Implementations of Metric in this package are Gauge, Counter, Histogram, Summary, and Untyped.

func MustNewConstHistogram
func MustNewConstHistogram(
    desc *Desc,
    count uint64,
    sum float64,
    buckets map[float64]uint64,
    labelValues ...string,
) Metric
MustNewConstHistogram is a version of NewConstHistogram that panics where NewConstMetric would have returned an error.

func MustNewConstMetric
func MustNewConstMetric(desc *Desc, valueType ValueType, value float64, labelValues ...string) Metric
MustNewConstMetric is a version of NewConstMetric that panics where NewConstMetric would have returned an error.

func MustNewConstSummary
func MustNewConstSummary(
    desc *Desc,
    count uint64,
    sum float64,
    quantiles map[float64]float64,
    labelValues ...string,
) Metric
MustNewConstSummary is a version of NewConstSummary that panics where NewConstMetric would have returned an error.

func NewConstHistogram
func NewConstHistogram(
    desc *Desc,
    count uint64,
    sum float64,
    buckets map[float64]uint64,
    labelValues ...string,
) (Metric, error)
NewConstHistogram returns a metric representing a Prometheus histogram with fixed values for the count, sum, and bucket counts. As those parameters cannot be changed, the returned value does not implement the Histogram interface (but only the Metric interface). Users of this package will not have much use for it in regular operations. However, when implementing custom Collectors, it is useful as a throw-away metric that is generated on the fly to send it to Prometheus in the Collect method.

buckets is a map of upper bounds to cumulative counts, excluding the +Inf bucket.

NewConstHistogram returns an error if the length of labelValues is not consistent with the variable labels in Desc.

Example
func NewConstMetric
func NewConstMetric(desc *Desc, valueType ValueType, value float64, labelValues ...string) (Metric, error)
NewConstMetric returns a metric with one fixed value that cannot be changed. Users of this package will not have much use for it in regular operations. However, when implementing custom Collectors, it is useful as a throw-away metric that is generated on the fly to send it to Prometheus in the Collect method. NewConstMetric returns an error if the length of labelValues is not consistent with the variable labels in Desc.

func NewConstSummary
func NewConstSummary(
    desc *Desc,
    count uint64,
    sum float64,
    quantiles map[float64]float64,
    labelValues ...string,
) (Metric, error)
NewConstSummary returns a metric representing a Prometheus summary with fixed values for the count, sum, and quantiles. As those parameters cannot be changed, the returned value does not implement the Summary interface (but only the Metric interface). Users of this package will not have much use for it in regular operations. However, when implementing custom Collectors, it is useful as a throw-away metric that is generated on the fly to send it to Prometheus in the Collect method.

quantiles maps ranks to quantile values. For example, a median latency of 0.23s and a 99th percentile latency of 0.56s would be expressed as:

map[float64]float64{0.5: 0.23, 0.99: 0.56}
NewConstSummary returns an error if the length of labelValues is not consistent with the variable labels in Desc.

Example
func NewInvalidMetric
func NewInvalidMetric(desc *Desc, err error) Metric
NewInvalidMetric returns a metric whose Write method always returns the provided error. It is useful if a Collector finds itself unable to collect a metric and wishes to report an error to the registry.

type MultiError
type MultiError []error
MultiError is a slice of errors implementing the error interface. It is used by a Gatherer to report multiple errors during MetricFamily gathering.

func (*MultiError) Append
func (errs *MultiError) Append(err error)
Append appends the provided error if it is not nil.

func (MultiError) Error
func (errs MultiError) Error() string
func (MultiError) MaybeUnwrap
func (errs MultiError) MaybeUnwrap() error
MaybeUnwrap returns nil if len(errs) is 0. It returns the first and only contained error as error if len(errs is 1). In all other cases, it returns the MultiError directly. This is helpful for returning a MultiError in a way that only uses the MultiError if needed.

type Observer
type Observer interface {
    Observe(float64)
}
Observer is the interface that wraps the Observe method, which is used by Histogram and Summary to add observations.

type ObserverFunc
type ObserverFunc func(float64)
The ObserverFunc type is an adapter to allow the use of ordinary functions as Observers. If f is a function with the appropriate signature, ObserverFunc(f) is an Observer that calls f.

This adapter is usually used in connection with the Timer type, and there are two general use cases:

The most common one is to use a Gauge as the Observer for a Timer. See the "Gauge" Timer example.

The more advanced use case is to create a function that dynamically decides which Observer to use for observing the duration. See the "Complex" Timer example.

func (ObserverFunc) Observe
func (f ObserverFunc) Observe(value float64)
Observe calls f(value). It implements Observer.

type ObserverVec
type ObserverVec interface {
    GetMetricWith(Labels) (Observer, error)
    GetMetricWithLabelValues(lvs ...string) (Observer, error)
    With(Labels) Observer
    WithLabelValues(...string) Observer
    CurryWith(Labels) (ObserverVec, error)
    MustCurryWith(Labels) ObserverVec

    Collector
}
ObserverVec is an interface implemented by `HistogramVec` and `SummaryVec`.

type Opts
type Opts struct {
    // Namespace, Subsystem, and Name are components of the fully-qualified
    // name of the Metric (created by joining these components with
    // "_"). Only Name is mandatory, the others merely help structuring the
    // name. Note that the fully-qualified name of the metric must be a
    // valid Prometheus metric name.
    Namespace string
    Subsystem string
    Name      string

    // Help provides information about this metric. Mandatory!
    //
    // Metrics with the same fully-qualified name must have the same Help
    // string.
    Help string

    // ConstLabels are used to attach fixed labels to this metric. Metrics
    // with the same fully-qualified name must have the same label names in
    // their ConstLabels.
    //
    // ConstLabels are only used rarely. In particular, do not use them to
    // attach the same labels to all your metrics. Those use cases are
    // better covered by target labels set by the scraping Prometheus
    // server, or by one specific metric (e.g. a build_info or a
    // machine_role metric). See also
    // https://prometheus.io/docs/instrumenting/writing_exporters/#target-labels,-not-static-scraped-labels
    ConstLabels Labels
}
Opts bundles the options for creating most Metric types. Each metric implementation XXX has its own XXXOpts type, but in most cases, it is just be an alias of this type (which might change when the requirement arises.)

It is mandatory to set Name and Help to a non-empty string. All other fields are optional and can safely be left at their zero value.

type Registerer
type Registerer interface {
    // Register registers a new Collector to be included in metrics
    // collection. It returns an error if the descriptors provided by the
    // Collector are invalid or if they — in combination with descriptors of
    // already registered Collectors — do not fulfill the consistency and
    // uniqueness criteria described in the documentation of metric.Desc.
    //
    // If the provided Collector is equal to a Collector already registered
    // (which includes the case of re-registering the same Collector), the
    // returned error is an instance of AlreadyRegisteredError, which
    // contains the previously registered Collector.
    //
    // It is in general not safe to register the same Collector multiple
    // times concurrently.
    Register(Collector) error
    // MustRegister works like Register but registers any number of
    // Collectors and panics upon the first registration that causes an
    // error.
    MustRegister(...Collector)
    // Unregister unregisters the Collector that equals the Collector passed
    // in as an argument.  (Two Collectors are considered equal if their
    // Describe method yields the same set of descriptors.) The function
    // returns whether a Collector was unregistered.
    //
    // Note that even after unregistering, it will not be possible to
    // register a new Collector that is inconsistent with the unregistered
    // Collector, e.g. a Collector collecting metrics with the same name but
    // a different help string. The rationale here is that the same registry
    // instance must only collect consistent metrics throughout its
    // lifetime.
    Unregister(Collector) bool
}
Registerer is the interface for the part of a registry in charge of registering and unregistering. Users of custom registries should use Registerer as type for registration purposes (rather than the Registry type directly). In that way, they are free to use custom Registerer implementation (e.g. for testing purposes).

type Registry
type Registry struct {
    // contains filtered or unexported fields
}
Registry registers Prometheus collectors, collects their metrics, and gathers them into MetricFamilies for exposition. It implements both Registerer and Gatherer. The zero value is not usable. Create instances with NewRegistry or NewPedanticRegistry.

func NewPedanticRegistry
func NewPedanticRegistry() *Registry
NewPedanticRegistry returns a registry that checks during collection if each collected Metric is consistent with its reported Desc, and if the Desc has actually been registered with the registry.

Usually, a Registry will be happy as long as the union of all collected Metrics is consistent and valid even if some metrics are not consistent with their own Desc or a Desc provided by their registered Collector. Well-behaved Collectors and Metrics will only provide consistent Descs. This Registry is useful to test the implementation of Collectors and Metrics.

func NewRegistry
func NewRegistry() *Registry
NewRegistry creates a new vanilla Registry without any Collectors pre-registered.

func (*Registry) Gather
func (r *Registry) Gather() ([]*dto.MetricFamily, error)
Gather implements Gatherer.

func (*Registry) MustRegister
func (r *Registry) MustRegister(cs ...Collector)
MustRegister implements Registerer.

func (*Registry) Register
func (r *Registry) Register(c Collector) error
Register implements Registerer.

func (*Registry) Unregister
func (r *Registry) Unregister(c Collector) bool
Unregister implements Registerer.

type Summary
type Summary interface {
    Metric
    Collector

    // Observe adds a single observation to the summary.
    Observe(float64)
}
A Summary captures individual observations from an event or sample stream and summarizes them in a manner similar to traditional summary statistics: 1. sum of observations, 2. observation count, 3. rank estimations.

A typical use-case is the observation of request latencies. By default, a Summary provides the median, the 90th and the 99th percentile of the latency as rank estimations. However, the default behavior will change in the upcoming v0.10 of the library. There will be no rank estiamtions at all by default. For a sane transition, it is recommended to set the desired rank estimations explicitly.

Note that the rank estimations cannot be aggregated in a meaningful way with the Prometheus query language (i.e. you cannot average or add them). If you need aggregatable quantiles (e.g. you want the 99th percentile latency of all queries served across all instances of a service), consider the Histogram metric type. See the Prometheus documentation for more details.

To create Summary instances, use NewSummary.

Example
func NewSummary
func NewSummary(opts SummaryOpts) Summary
NewSummary creates a new Summary based on the provided SummaryOpts.

type SummaryOpts
type SummaryOpts struct {
    // Namespace, Subsystem, and Name are components of the fully-qualified
    // name of the Summary (created by joining these components with
    // "_"). Only Name is mandatory, the others merely help structuring the
    // name. Note that the fully-qualified name of the Summary must be a
    // valid Prometheus metric name.
    Namespace string
    Subsystem string
    Name      string

    // Help provides information about this Summary. Mandatory!
    //
    // Metrics with the same fully-qualified name must have the same Help
    // string.
    Help string

    // ConstLabels are used to attach fixed labels to this metric. Metrics
    // with the same fully-qualified name must have the same label names in
    // their ConstLabels.
    //
    // ConstLabels are only used rarely. In particular, do not use them to
    // attach the same labels to all your metrics. Those use cases are
    // better covered by target labels set by the scraping Prometheus
    // server, or by one specific metric (e.g. a build_info or a
    // machine_role metric). See also
    // https://prometheus.io/docs/instrumenting/writing_exporters/#target-labels,-not-static-scraped-labels
    ConstLabels Labels

    // Objectives defines the quantile rank estimates with their respective
    // absolute error. If Objectives[q] = e, then the value reported for q
    // will be the φ-quantile value for some φ between q-e and q+e.  The
    // default value is DefObjectives. It is used if Objectives is left at
    // its zero value (i.e. nil). To create a Summary without Objectives,
    // set it to an empty map (i.e. map[float64]float64{}).
    //
    // Deprecated: Note that the current value of DefObjectives is
    // deprecated. It will be replaced by an empty map in v0.10 of the
    // library. Please explicitly set Objectives to the desired value.
    Objectives map[float64]float64

    // MaxAge defines the duration for which an observation stays relevant
    // for the summary. Must be positive. The default value is DefMaxAge.
    MaxAge time.Duration

    // AgeBuckets is the number of buckets used to exclude observations that
    // are older than MaxAge from the summary. A higher number has a
    // resource penalty, so only increase it if the higher resolution is
    // really required. For very high observation rates, you might want to
    // reduce the number of age buckets. With only one age bucket, you will
    // effectively see a complete reset of the summary each time MaxAge has
    // passed. The default value is DefAgeBuckets.
    AgeBuckets uint32

    // BufCap defines the default sample stream buffer size.  The default
    // value of DefBufCap should suffice for most uses. If there is a need
    // to increase the value, a multiple of 500 is recommended (because that
    // is the internal buffer size of the underlying package
    // "github.com/bmizerany/perks/quantile").
    BufCap uint32
}
SummaryOpts bundles the options for creating a Summary metric. It is mandatory to set Name and Help to a non-empty string. While all other fields are optional and can safely be left at their zero value, it is recommended to explicitly set the Objectives field to the desired value as the default value will change in the upcoming v0.10 of the library.

type SummaryVec
type SummaryVec struct {
    // contains filtered or unexported fields
}
SummaryVec is a Collector that bundles a set of Summaries that all share the same Desc, but have different values for their variable labels. This is used if you want to count the same thing partitioned by various dimensions (e.g. HTTP request latencies, partitioned by status code and method). Create instances with NewSummaryVec.

Example
func NewSummaryVec
func NewSummaryVec(opts SummaryOpts, labelNames []string) *SummaryVec
NewSummaryVec creates a new SummaryVec based on the provided SummaryOpts and partitioned by the given label names.

func (*SummaryVec) CurryWith
func (v *SummaryVec) CurryWith(labels Labels) (ObserverVec, error)
CurryWith returns a vector curried with the provided labels, i.e. the returned vector has those labels pre-set for all labeled operations performed on it. The cardinality of the curried vector is reduced accordingly. The order of the remaining labels stays the same (just with the curried labels taken out of the sequence – which is relevant for the (GetMetric)WithLabelValues methods). It is possible to curry a curried vector, but only with labels not yet used for currying before.

The metrics contained in the SummaryVec are shared between the curried and uncurried vectors. They are just accessed differently. Curried and uncurried vectors behave identically in terms of collection. Only one must be registered with a given registry (usually the uncurried version). The Reset method deletes all metrics, even if called on a curried vector.

func (SummaryVec) Delete
func (m SummaryVec) Delete(labels Labels) bool
Delete deletes the metric where the variable labels are the same as those passed in as labels. It returns true if a metric was deleted.

It is not an error if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc. However, such inconsistent Labels can never match an actual metric, so the method will always return false in that case.

This method is used for the same purpose as DeleteLabelValues(...string). See there for pros and cons of the two methods.

func (SummaryVec) DeleteLabelValues
func (m SummaryVec) DeleteLabelValues(lvs ...string) bool
DeleteLabelValues removes the metric where the variable labels are the same as those passed in as labels (same order as the VariableLabels in Desc). It returns true if a metric was deleted.

It is not an error if the number of label values is not the same as the number of VariableLabels in Desc. However, such inconsistent label count can never match an actual metric, so the method will always return false in that case.

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider Delete(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map). See also the CounterVec example.

func (*SummaryVec) GetMetricWith
func (v *SummaryVec) GetMetricWith(labels Labels) (Observer, error)
GetMetricWith returns the Summary for the given Labels map (the label names must match those of the VariableLabels in Desc). If that label map is accessed for the first time, a new Summary is created. Implications of creating a Summary without using it and keeping the Summary for later use are the same as for GetMetricWithLabelValues.

An error is returned if the number and names of the Labels are inconsistent with those of the VariableLabels in Desc (minus any curried labels).

This method is used for the same purpose as GetMetricWithLabelValues(...string). See there for pros and cons of the two methods.

func (*SummaryVec) GetMetricWithLabelValues
func (v *SummaryVec) GetMetricWithLabelValues(lvs ...string) (Observer, error)
GetMetricWithLabelValues returns the Summary for the given slice of label values (same order as the VariableLabels in Desc). If that combination of label values is accessed for the first time, a new Summary is created.

It is possible to call this method without using the returned Summary to only create the new Summary but leave it at its starting value, a Summary without any observations.

Keeping the Summary for later use is possible (and should be considered if performance is critical), but keep in mind that Reset, DeleteLabelValues and Delete can be used to delete the Summary from the SummaryVec. In that case, the Summary will still exist, but it will not be exported anymore, even if a Summary with the same label values is created later. See also the CounterVec example.

An error is returned if the number of label values is not the same as the number of VariableLabels in Desc (minus any curried labels).

Note that for more than one label value, this method is prone to mistakes caused by an incorrect order of arguments. Consider GetMetricWith(Labels) as an alternative to avoid that type of mistake. For higher label numbers, the latter has a much more readable (albeit more verbose) syntax, but it comes with a performance overhead (for creating and processing the Labels map). See also the GaugeVec example.

func (*SummaryVec) MustCurryWith
func (v *SummaryVec) MustCurryWith(labels Labels) ObserverVec
MustCurryWith works as CurryWith but panics where CurryWith would have returned an error.

func (*SummaryVec) With
func (v *SummaryVec) With(labels Labels) Observer
With works as GetMetricWith, but panics where GetMetricWithLabels would have returned an error. Not returning an error allows shortcuts like

myVec.With(prometheus.Labels{"code": "404", "method": "GET"}).Observe(42.21)
func (*SummaryVec) WithLabelValues
func (v *SummaryVec) WithLabelValues(lvs ...string) Observer
WithLabelValues works as GetMetricWithLabelValues, but panics where GetMetricWithLabelValues would have returned an error. Not returning an error allows shortcuts like

myVec.WithLabelValues("404", "GET").Observe(42.21)
type Timer
type Timer struct {
    // contains filtered or unexported fields
}
Timer is a helper type to time functions. Use NewTimer to create new instances.

Example
Example (Complex)
Example (Gauge)
func NewTimer
func NewTimer(o Observer) *Timer
NewTimer creates a new Timer. The provided Observer is used to observe a duration in seconds. Timer is usually used to time a function call in the following way:

func TimeMe() {
    timer := NewTimer(myHistogram)
    defer timer.ObserveDuration()
    // Do actual work.
}
func (*Timer) ObserveDuration
func (t *Timer) ObserveDuration()
ObserveDuration records the duration passed since the Timer was created with NewTimer. It calls the Observe method of the Observer provided during construction with the duration in seconds as an argument. ObserveDuration is usually called with a defer statement.

Note that this method is only guaranteed to never observe negative durations if used with Go1.9+.

type UntypedFunc
type UntypedFunc interface {
    Metric
    Collector
}
UntypedFunc works like GaugeFunc but the collected metric is of type "Untyped". UntypedFunc is useful to mirror an external metric of unknown type.

To create UntypedFunc instances, use NewUntypedFunc.

func NewUntypedFunc
func NewUntypedFunc(opts UntypedOpts, function func() float64) UntypedFunc
NewUntypedFunc creates a new UntypedFunc based on the provided UntypedOpts. The value reported is determined by calling the given function from within the Write method. Take into account that metric collection may happen concurrently. If that results in concurrent calls to Write, like in the case where an UntypedFunc is directly registered with Prometheus, the provided function must be concurrency-safe.

type UntypedOpts
type UntypedOpts Opts
UntypedOpts is an alias for Opts. See there for doc comments.

type ValueType
type ValueType int
ValueType is an enumeration of metric types that represent a simple value.

const (
    CounterValue ValueType
    GaugeValue
    UntypedValue
)
Possible values for the ValueType enum.

Directories
Path	Synopsis
graphite	Package graphite provides a bridge to push Prometheus metrics to a Graphite server.
promauto	Package promauto provides constructors for the usual Prometheus metrics that return them already registered with the global registry (prometheus.DefaultRegisterer).
promhttp	Package promhttp provides tooling around HTTP servers and clients.
push	Package push provides functions to push metrics to a Pushgateway.
Package prometheus imports 27 packages (graph) and is imported by 5781 packages. Updated 13 days ago. Refresh now. Tools for package owners.

Website Issues | Go Language Back to top
