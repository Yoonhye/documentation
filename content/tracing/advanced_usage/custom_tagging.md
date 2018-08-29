---
title: Custom Tagging
kind: documentation
further_reading:
- link: "tracing/advanced_usage/manual_tracing"
  tag: "Manual Tracing"
  text: Programmatic creation of traces to send to Datadog.
---

Custom Tagging allows adding key-value pairs to specific spans. These key-value pairs (tags) are used to correlate traces with other Datadog products and to provide more details about specific spans.

{{< tabs >}}
{{% tab "Java" %}}
Tags are key-value pairs attached to spans. All tags share a single namespace.

The Datadog UI uses specific tags to set UI properties, such as an application's service name. A full list of these tags can be found in the [Datadog][java api doc] and [OpenTracing][opentracing java doc] APIs.

### Custom Tags

Custom tags are set using the OpenTracing API.

Custom tags may be set for auto-instrumentation by grabbing the active span out of the global tracer.

```java
import io.opentracing.Tracer;
import io.opentracing.util.GlobalTracer;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet
class ServletImpl extends AbstractHttpServlet {
  @Override
  void doGet(HttpServletRequest req, HttpServletResponse resp) {
    final Tracer tracer = GlobalTracer.get();
    if (tracer != null && tracer.activeSpan() != null) {
      tracer.activeSpan().setTag("org.id", 12345);
      tracer.activeSpan().setTag("http.url", "/login");
    }
    // servlet impl
  }
}
```

[java api doc]: https://github.com/DataDog/dd-trace-java/blob/master/dd-trace-api/src/main/java/datadog/trace/api/DDTags.java
[opentracing java doc]: https://github.com/opentracing/opentracing-java/blob/master/opentracing-api/src/main/java/io/opentracing/tag/Tags.java

{{% /tab %}}
{{% tab "Python" %}}

**Adding tags to a span**

You can add tags directly to a span by calling `set_tag`. For example with
the following route handler:

```python
from ddtrace import tracer

@app.route('/post/<int:post_id>')
def handle_post(post_id):
  with tracer.trace('web.request') as span:
    span.set_tag('post.id', post_id)
```

**Adding tags to a current active span**

The current span can be retrieved from the context in order to set tags. This
way, if a span was started by our instrumentation, you can retrieve the span
and add custom tags. Note that if a span does not exist, `None` will be
returned:

```python
from ddtrace import tracer

@app.route('/post/<int:post_id>')
@tracer.wrap()
def handle_post(post_id):
  # get the active span in the context, put there by tracer.wrap()
  current_span = tracer.current_span()
  if current_span:
    current_span.set_tag('post.id', post_id)
```

**Adding tags globally to all spans**

You can also add tags to all spans by configuring the tracer with the `tracer.set_tags` method:

```python
from ddtrace import tracer

tracer.set_tags({ 'env': 'prod' })
```

{{% /tab %}}
{{% tab "Ruby" %}}
**Adding tags to a span**

You can add tags directly to `Datadog::Span` objects by calling `#set_tag`:

```ruby
# An example of a Sinatra endpoint,
# with Datadog tracing around the request.
get '/posts' do
  Datadog.tracer.trace('web.request') do |span|
    span.set_tag('http.url', request.path)
  end
end
```

**Adding tags to a current active span**

You can also access the current active span from any method within your code. Note, however, that if the method is called and there is no span currently active, `active_span` will be nil.

```ruby
# e.g. adding tag to active span

current_span = Datadog.tracer.active_span
current_span.set_tag('my_tag', 'my_value') unless current_span.nil?
```

**Adding tags globally to all spans**

You can also add tags to all spans by configuring the tracer with the `tags` option:

```ruby
Datadog.configure do |c|
  c.tracer tags: { 'env' => 'prod' }
end
```

See the [API documentation][ruby api doc] for more details.

[ruby api doc]: https://github.com/DataDog/dd-trace-rb/blob/master/docs/GettingStarted.md#environment-and-tags

{{% /tab %}}
{{% tab "Go" %}}
**Adding tags to a span**

You can add tags directly to a `Span` interface by calling `SetTag`:

```go
package main

import (
    "log"
    "net/http"

    "gopkg.in/DataDog/dd-trace-go.v1/ddtrace/tracer"
)

func handler(w http.ResponseWriter, r *http.Request) {
    // Create a span for a web request at the /posts URL.
    span := tracer.StartSpan("web.request", tracer.ResourceName("/posts"))
    defer span.Finish()

    // Set tag
    span.SetTag("http.url", r.URL.Path)
}

func main() {
    tracer.Start(tracer.WithServiceName("my-blog"))
    defer tracer.Stop()
    http.HandleFunc("/posts", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**Adding tags to a Span attached to a Context**

Our integrations make use of `Context` type to propagate the current active span. If you want to add
a tag to a span attached to a `Context` via automatic instrumentation, you can call the `SpanFromContext`
function:

```go
package main

import (
    "net/http"

    "gopkg.in/DataDog/dd-trace-go.v1/ddtrace/tracer"
)

func handler(w http.ResponseWriter, r *http.Request) {
    // Retrieve a span for a web request attached to a Go Context.
    if span, ok := tracer.SpanFromContext(r.Context()); ok {
        // Set tag
        span.SetTag("http.url", r.URL.Path)
    }
}
```

**Adding tags globally to all spans**

You can also add tags to all spans by configuring the tracer with the `tags` option:

```go
package main

import "gopkg.in/DataDog/dd-trace-go.v1/ddtrace/tracer"

func main() {
    tracer.Start(
        tracer.WithGlobalTag("datacenter", "us-1"),
        tracer.WithGlobalTag("env", "prod"),
    )
    defer tracer.Stop()
}
```

{{% /tab %}}
{{% tab "Node.js" %}}
**Adding tags to a span**

You can add tags directly to the span objects directly by calling `setTag` or `addTags`:

```javascript
// An example of an Express endpoint,
// with Datadog tracing around the request.
app.get('/posts', (req, res) => {
  const span = tracer.startSpan('web.request')

  span.setTag('http.url', req.url)
  span.addTags({
    'http.method': req.method
  })
})
```

**Adding tags to a current active span**

You can also access the current active span from any method within your code. Note however that if the method is called and there is no span currently active `tracer.scopeManager().active()` will return `null`.

```javascript
// e.g. adding tag to active span

const scope = tracer.scopeManager().active()
const span = scope.span()

span.setTag('my_tag', 'my_value')
```

**Adding tags globally to all spans**

You can also add tags to all spans by configuring the tracer with the `tags` option:

```javascript
tracer.init({
  tags: {
    env: 'prod'
  }
})
```

See the [API documentation][nodejs api doc] for more details.

[nodejs api doc]: https://datadog.github.io/dd-trace-js/
{{% /tab %}}
{{< /tabs >}}

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}