---
title: Distributed Tracing
kind: documentation
---

Distributed Tracing links traces across multiple hosts. Linking is implemented by injecting Datadog Metadata into the request headers.

Distributed Tracing headers are language agnostic. A trace started in one language may propagate to another (for example, from Python to Java).


## NodeJS

Distributed tracing allows you to propagate a single trace across multiple services, so you can see performance end-to-end.

Distributed tracing is enabled by default for all supported integrations.

## Ruby 

Distributed tracing allows you to propagate a single trace across multiple services, so you can see performance end-to-end.

Distributed tracing is disabled by default. For more details about how to activate and configure distributed tracing, check out the [API documentation][8].

## Java

Create a distributed trace in custom instrumentation using OpenTracing:

```java
// Step 1: Inject the Datadog headers in the client code
try (Scope scope = tracer.buildSpan("httpClientSpan").startActive(true)) {
    final Span span = scope.span();
    HttpRequest request = /* your code here */;

    tracer.inject(span.context(),
                  Format.Builtin.HTTP_HEADERS,
                  new MyHttpHeadersInjectAdapter(request));

    // http request impl...
}

public static class MyHttpHeadersInjectAdapter implements TextMap {
  private final HttpRequest httpRequest;

  public HttpHeadersInjectAdapter(final HttpRequest httpRequest) {
    this.httpRequest = httpRequest;
  }

  @Override
  public void put(final String key, final String value) {
    httpRequest.addHeader(key, value);
  }

  @Override
  public Iterator<Map.Entry<String, String>> iterator() {
    throw new UnsupportedOperationException("This class should be used only with tracer#inject()");
  }
}

// Step 2: Extract the Datadog headers in the server code
HttpRequest request = /* your code here */;

final SpanContext extractedContext =
  GlobalTracer.get().extract(Format.Builtin.HTTP_HEADERS,
                             new MyHttpRequestExtractAdapter(request));

try (Scope scope = tracer.buildSpan("httpServerSpan").asChildOf(extractedContext).startActive(true)) {
    final Span span = scope.span(); // will be a child of http client span in step 1
    // http server impl...
}

public class MyHttpRequestExtractAdapter implements TextMap {
  private final HttpRequest request;

  public HttpRequestExtractAdapter(final HttpRequest request) {
    this.request = request;
  }

  @Override
  public Iterator<Map.Entry<String, String>> iterator() {
    return request.headers().iterator();
  }

  @Override
  public void put(final String key, final String value) {
    throw new UnsupportedOperationException("This class should be used only with Tracer.extract()!");
  }
}
```

## GO

Propagate a single trace across multiple services with distributed tracing. For more details about how to use and configure distributed tracing, check out the [godoc page][tracer godoc].

Make use of priority sampling to ensure that distributed traces are complete. Set the sampling priority of a trace by adding the `sampling.priority` tag to its root span. This is then propagated throughout the entire stack. For example:

```go
span.SetTag(ext.SamplingPriority, ext.PriorityUserKeep)
```

Possible values for the sampling priority tag are:

| Sampling Value             | Effect                                                                                                      |
| -------------------------- | :---------------------------------------------------------------------------------------------------------- |
| ext.PriorityAutoReject     | The sampler automatically decided to not keep the trace. The Agent will drop it.                            |
| ext.PriorityAutoKeep       | The sampler automatically decided to keep the trace. The Agent will keep it. Might be sampled server-side.  |
| ext.PriorityUserReject     | The user asked to not keep the trace. The Agent will drop it.                                               |
| ext.PriorityUserKeep       | The user asked to keep the trace. The Agent will keep it. The server will keep it too.                      |

[8]: https://github.com/DataDog/dd-trace-rb/blob/master/docs/GettingStarted.md#distributed-tracing