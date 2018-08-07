# Tracer: Distributed system tracing

[![Highway at Night](docs/highway.jpg)](https://pixabay.com/en/highway-at-night-long-long-exposure-371009/)

[![Stability: Active](https://masterminds.github.io/stability/active.svg)](https://masterminds.github.io/stability/active.html)
[![Build Status](https://img.shields.io/travis/zalando/tracer/master.svg)](https://travis-ci.org/zalando/tracer)
[![Coverage Status](https://img.shields.io/coveralls/zalando/tracer/master.svg)](https://coveralls.io/r/zalando/tracer)
[![Code Quality](https://img.shields.io/codacy/grade/213bb62c41b34a32951929e37a2d20ac/master.svg)](https://www.codacy.com/app/whiskeysierra/tracer)
[![Javadoc](http://javadoc.io/badge/org.zalando/tracer-core.svg)](http://www.javadoc.io/doc/org.zalando/tracer-core)
[![Release](https://img.shields.io/github/release/zalando/tracer.svg)](https://github.com/zalando/tracer/releases)
[![Maven Central](https://img.shields.io/maven-central/v/org.zalando/tracer-parent.svg)](https://maven-badges.herokuapp.com/maven-central/org.zalando/tracer-parent)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/zalando/tracer/master/LICENSE)

> **Tracer** noun, /ˈtɹeɪsɚ/: A round of ammunition that contains a flammable substance that produces a visible trail when fired in the dark.

*Tracer* is a library that manages custom trace identifiers and carries them through distributed systems. Traditionally traces are transported as custom HTTP headers, usually generated by clients on the very first request. Traces are added  to any subsequent requests and responses, especially to transitive dependencies. Having a consistent trace across different services in a system allows to correlate requests and responses beyond the traditional single client-server communication.

This library historically originates from a closed-source implementation called *Flow-ID*. The goal was to create a clean open source version in which we could get rid of all the drawbacks of the old implementation, e.g. strong-coupling to internal libraries, single hard-coded header and limited testability.

- **Status**: Under development and used in production

## Features

-  **Tracing** of HTTP requests and responses
-  **Customization** by having a pluggable trace format and lifecycle listeners for easy integration
-  **Support** for Servlet containers, Apache’s HTTP client, Square's OkHttp, Hystrix, JUnit, AspectJ and (via its elegant API) several other frameworks
-  Convenient [Spring Boot](http://projects.spring.io/spring-boot/) Auto Configuration
-  Sensible defaults

## Dependencies

- Java 8
- Any build tool using Maven Central, or direct download
- Servlet Container (optional)
- Apache HTTP Client (optional)
- OkHttp (optional)
- Hystrix (optional)
- AspectJ (optional)
- JUnit (optional)
- Spring 4.x **or 5.x** (optional)
- Spring Boot 1.x **or 2.x** (optional)

## Installation

Add the following dependency to your project:

```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-core</artifactId>
    <version>${tracer.version}</version>
</dependency>
```

Additional modules/artifacts of Tracer always share the same version number.

Alternatively, you can import our *bill of materials*...

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.zalando</groupId>
      <artifactId>tracer-bom</artifactId>
      <version>${tracer.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

... which allows you to omit versions and scopes:

```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-core</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-servlet</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-httpclient</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-okhttp</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-aspectj</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-junit</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>tracer-spring-boot-starter</artifactId>
</dependency>
```

## Usage

After adding the dependency, create a `Tracer` and specify the name of the traces you want to manage:

```java
Tracer tracer = Tracer.create("X-Trace-ID");
```

If you need access to the current trace's value, call `getValue()` on it:

```java
Trace trace = tracer.get("X-Trace-ID"); // this is a live-view that can be a shared as a singleton
entity.setLastModifiedBy(trace.getValue());
```

By default there can only be one trace active at a time. Sometimes it can be useful to *stack* traces:

```java
Tracer tracer = Tracer.builder()
    .stacked(true)
    .trace("X-Trace-ID")
    .build();
```

### Generators

When starting a new trace, *Tracer* will create trace value by means of pre-configured generator.
You can override this on a per-trace level by adding the following to your setup:

```java
Tracer tracer = Tracer.builder()
        .trace("X-Trace-ID", new CustomGenerator())
        .build();
```

There are several generator implementations included.

#### UUID

The uuid generator creates random-based UUID values.

    5cdc0690-536b-11e8-8b88-6b0631b44b96
    
The generated values have a length of 36 characters.

#### Flow ID

The flow id generator creates 128-bit random integer values encoded in base64:

    REcCvlqMSReeo7adheiYFA

The generated values have a length of 22 characters.

#### Phrase

The phrase generator creates over 10^9 different phrases like:

    tender_goodall_likes_evil_panini
    nostalgic_boyd_helps_agitated_noyce
    pensive_allen_tells_fervent_einstein

The generates values have a length between 22 and 61 characters.

#### Random64 and Random128

The random generators create random hexadecimal integers of length 64- and 128-bit, respectively.

    c94af1541b3e4f3a
    1ff222514858965f0960bdbb78fe550e
    
The generated values have a length of 16 or 32 characters.

### Listeners

For some use cases, e.g. integration with other frameworks and libraries, it might be useful to register a listener
that gets notified every time a trace is either started or stopped.

```java
Tracer tracer = Tracer.builder()
        .trace("X-Trace-ID")
        .listener(new CustomTraceListener())
        .build();
```

*Tracer* comes with a very useful listener by default, the `MDCTraceListener`:

```java
Tracer tracer = Tracer.builder()
        .trace("X-Trace-ID")
        .listener(new MDCTraceListener())
        .build();
```

It allows you to add the trace id to every log line:

```xml
<PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} [%X{X-Trace-ID}] - %msg%n"/>
```

Another built-in listener is the `LoggingTraceListener` which logs the start and end of every trace.

## Servlet

On the server side is a single filter that you must be register in your filter chain. Make sure it runs very early — otherwise you might miss some crucial information when debugging.

You have to register the `TracerFilter` as a `Filter` in your filter chain:

```java
context.addFilter("TracerFilter", new TracerFilter(tracer))
    .addMappingForUrlPatterns(EnumSet.of(REQUEST, ASYNC, ERROR), true, "/*");
```

## Apache HTTP Client

Many client-side HTTP libraries on the JVM use the Apache HTTPClient, which is why *Tracer* comes with a request interceptor:

```java
DefaultHttpClient client = new DefaultHttpClient();
client.addRequestInterceptor(new TracerHttpRequestInterceptor(tracer));
```

### OkHttp

The `tracer-okhttp` module contains an `Interceptor` to use with the `OkHttpClient`:

```java
OkHttpClient client = new OkHttpClient.Builder()
        .addNetworkInterceptor(new TracerInterceptor(tracer))
        .build();
```


## Hystrix

*Tracer* comes with built-in Hystrix support in form of a custom `HystrixConcurrencyStrategy`:

```java
final HystrixPlugins plugins = HystrixPlugins.getInstance();
final HystrixConcurrencyStrategy delegate = HystrixConcurrencyStrategyDefault.getInstance(); // or another
plugins.registerConcurrencyStrategy(new TracerConcurrencyStrategy(tracer, delegate));
```

## AspectJ

For background jobs and tests, you can use the built-in aspect:

```java
@Traced
public void performBackgroundJob() {
    // do work
}
```

or you can manage the lifecycle yourself:

```java
tracer.start();

try {
    // do work
} finally {
    tracer.stop();
}
```

## JUnit

*Tracer* comes with a special `TestRule` that manages traces for every test run for you:

```java
@Rule
public final TracerRule tracing = new TracerRule(tracer);
```

It also supports JSR-330-compliant dependency injection frameworks: 

```java
@Inject
@Rule
public final TracerRule tracing;
```

## Spring Boot Starter

*Tracer* comes with a convenient auto configuration for Spring Boot users that sets up aspect, servlet filter and MDC support automatically with sensible defaults:

| Configuration                 | Description                                                                                                          | Default                     |
|-------------------------------|----------------------------------------------------------------------------------------------------------------------|-----------------------------|
| `tracer.stacked`              | Enables stacking of traces                                                                                           | `false`                     |
| `tracer.aspect.enabled`       | Enables the [`TracedAspect`](#aspect)                                                                                | `true`                      |
| `tracer.async.enabled`        | Enables for asynchronous tasks, i.e. `@Async`                                                                        | `true`                      |
| `tracer.filter.enabled`       | Enables the [`TracerFilter`](#servlet)                                                                               | `true`                      |
| `tracer.logging.enabled`      | Enables the [`LoggingTraceListener`](#logging)                                                                       | `false`                     |
| `tracer.logging.category`     | Changes the category of the [`LoggingTraceListener`](#logging)                                                       | `org.zalando.tracer.Tracer` |
| `tracer.mdc.enabled`          | Enables the [`MdcTraceListener`](#logging)                                                                           | `true`                      |
| `tracer.scheduling.enabled`   | Enables support for Task Scheduling, i.e. `@Scheduled`                                                               | `true`                      |
| `tracer.scheduling.pool-size` | Configures the thread pool size, i.e. the number of scheduled tasks                                                  | # of CPUs                   |
| `tracer.traces`               | Configures actual traces, mapping from name to generator type (`uuid`, `flow-id`, `phrase`, `random64`, `random128`) |                             |

```yaml
tracer:
    aspect.enabled: true
    async.enabled: true
    filter.enabled: true
    logging:
        enabled: false
        category: org.zalando.tracer.Tracer
    mdc.enabled: true
    scheduling.enabled: true
    traces:
        X-Trace-ID: uuid
        X-Flow-ID: flow-id
```

The `TracerAutoConfiguration` will automatically pick up any `TraceListener` bound in the application context.

## Getting Help with Tracer

If you have questions, concerns, bug reports, etc., please file an issue in this repository's [Issue Tracker](../../issues).

## Getting Involved/Contributing

To contribute, simply make a pull request and add a brief description (1-2 sentences) of your addition or change. For
more details, check the [contribution guidelines](.github/CONTRIBUTING.md).

## Alternatives

Tracer, by design, does not provide sampling, metrics or annotations. Neither does it use the semantics of spans as
most of the following projects do. If you require any of these, you're highly encouraged to try them.

- [The OpenTracing Project](http://opentracing.io/)
- [Apache HTrace](http://htrace.incubator.apache.org/)
- [Spring Cloud Sleuth](http://cloud.spring.io/spring-cloud-sleuth/)
- [Zipkin](http://zipkin.io/)
