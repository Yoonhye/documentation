---
title: Logging
kind: documentation
further_reading:
- link: "tracing/advanced_usage/debugging"
  tag: "Debugging"
  text: Diagnose issues or audit trace data.
---

Datadog's logging APIs allows for accessing active tracing identifiers which can be used to correlate APM traces with specific log events.

{{< tabs >}}
{{% tab "Java" %}}
The Java tracer exposes two API calls to allow printing trace and span identifiers along with log statements, `CorrelationIdentifier#getTraceId()`, and `CorrelationIdentifier#getSpanId()`.

These identifiers can be injected into application logs using MDC frameworks.

log4j2:

```java
import org.apache.logging.log4j.ThreadContext;
import datadog.trace.api.CorrelationIdentifier;

// there must be spans started and active before this block.
try {
    ThreadContext.put("ddTraceID", "ddTraceID:" + String.valueOf(CorrelationIdentifier.getTraceId()));
    ThreadContext.put("ddSpanID", "ddSpanID:" + String.valueOf(CorrelationIdentifier.getSpanId()));
} finally {
    ThreadContext.remove("ddTraceID");
    ThreadContext.remove("ddSpanID");
}
```

slf4j/logback:

```java
import org.slf4j.MDC;
import datadog.trace.api.CorrelationIdentifier;

// there must be spans started and active before this block.
try {
    MDC.put("ddTraceID", "ddTraceID:" + String.valueOf(CorrelationIdentifier.getTraceId()));
    MDC.put("ddSpanID", "ddSpanID:" + String.valueOf(CorrelationIdentifier.getSpanId()));
} finally {
    MDC.remove("ddTraceID");
    MDC.remove("ddSpanID");
}
```

log4j2 XML Pattern:

```
<PatternLayout pattern="%clr{%d{yyyy-MM-dd HH:mm:ss.SSS}}{faint} %clr{%5p} %clr{${sys:PID}}{magenta} %clr{---}{faint} %X{ddTraceID} %X{ddSpanID} %m%n%xwEx" />
```

Logback XML Pattern:

```
<Pattern>
    %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} %X{ddTraceID} %X{ddSpanID} - %msg%n
</Pattern>
```

{{% /tab %}}
{{% tab "Python" %}}
Getting the required information needed for logging is easy:

```python
from ddtrace import helpers

trace_id, span_id = helpers.get_correlation_ids()
```
{{% /tab %}}
{{% tab "Ruby" %}}
Coming Soon.

{{% /tab %}}
{{% tab "Go" %}}
The Go tracer exposes two API calls to allow printing trace and span identifiers along with log statements using exported methods from `SpanContext` type:

```go
package main

import (
	"net/http"

	"gopkg.in/DataDog/dd-trace-go.v1/ddtrace/tracer"
)

func handler(w http.ResponseWriter, r *http.Request) {
	// Create a span for a web request at the /posts URL.
	span := tracer.StartSpan("web.request", tracer.ResourceName("/posts"))
	defer span.Finish()

	// Retrieve Trace ID and Span ID
	traceID := span.Context().TraceID()
	spanID := span.Context().SpanID()
}
```

{{% /tab %}}
{{% tab "Node.js" %}}
Coming Soon.
{{% /tab %}}
{{< /tabs >}}

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}