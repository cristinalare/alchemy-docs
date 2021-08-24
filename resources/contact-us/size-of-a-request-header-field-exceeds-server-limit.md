---
description: >-
  Large request headers will return errors even on methods that should be very
  small.
---

# Size of a request header field exceeds server limit

You may not always know exactly what headers you are sending in an API call. If you have a global wrapper or a wrapper that is very far up your import tree, then you may not even know that it is adding headers to your requests.

As an example, the Honeycomb Trace wrapper will add extremely large `X-Honeycomb-Trace` headers to every request you send. This will show up on our end as `Request header exceeds LimitRequestFieldSize: X-Honeycomb-Trace` and will be returned to you as a `Size of a request header field exceeds server limits` error.

In general you should avoid such global wrappers. If you want to trace a request then add the headers as-needed.

