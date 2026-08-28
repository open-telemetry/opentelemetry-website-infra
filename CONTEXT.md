# Site Observability

The public OpenTelemetry pipeline that observes the OpenTelemetry project's own
websites. Telemetry flows from Sources through the Edge into the Collector,
which exports to Backends whose Query UIs are open to anyone.

## Language

### Pipeline

**Collector**:
The single OpenTelemetry Collector service that receives all site telemetry. The
only component exposed for writes.
_Avoid_: gateway, proxy, agent

**Edge**:
The Worker in front of the Collector, responsible for rate limiting, body-size
capping, and Origin blocking. Cloudflare Containers require it; it is the
provider-specific layer. When referring to the telemetry source, always write
"Netlify Edge Functions" in full.
_Avoid_: perimeter, front door, ingress, bare "Edge Functions"

**Source**:
A system that emits telemetry into the Collector. Three exist: browser
instrumentation (on opentelemetry.io and explorer.opentelemetry.io), Netlify
Edge Functions, and the Netlify log drain.
_Avoid_: client, producer, emitter

**Backend**:
A store-and-query system the Collector exports to. Never receives writes from a
Source directly.
_Avoid_: sink, datastore, destination

### Surfaces

**Ingest endpoint**:
The public, secretless, write-only OTLP endpoint that Sources send to.
_Avoid_: collector endpoint, OTLP URL, ingress

**Query UI**:
A Backend's public read-only interface. Never accepts writes.
_Avoid_: dashboard, console

### Privacy

**Scrubbing**:
Removal of sensitive fields in the Collector before export. Because reads are
public, anything not scrubbed is world-visible.
_Avoid_: anonymization, sanitization, filtering
