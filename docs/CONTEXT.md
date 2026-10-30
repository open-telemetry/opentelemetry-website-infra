# Site Observability

The observability pipeline planned for the OpenTelemetry project's websites. Telemetry flows from Sources through the Edge into the Collector, which exports to Backends whose Query UIs are open to anyone.

## Language

### Websites

**Site**: One of the OpenTelemetry project's websites observed by this pipeline: opentelemetry.io or explorer.opentelemetry.io.

**Deploy preview**: A temporary version of a Site built for a pull request. Its telemetry is identified separately from production traffic. _Avoid_: staging environment

### Pipeline

**Collector**: The single OpenTelemetry Collector service that receives all site telemetry. The only component exposed for writes. _Avoid_: gateway, proxy, agent

**Edge**: The layer in front of the Collector that enforces request limits and the Origin policy. Use "Netlify Edge Functions" in full when referring to the telemetry Source. _Avoid_: perimeter, front door, ingress, bare "Edge Functions"

**Source**: A system that emits telemetry into the Collector. Planned Sources include browser instrumentation on the Sites, Netlify Edge Functions, and the Netlify log drain. _Avoid_: client, producer, emitter

**Synthetic telemetry**: Test traces and metrics created to exercise the pipeline without collecting data from Site visitors.

**Backend**: A store-and-query system the Collector exports to. Never receives writes from a Source directly. _Avoid_: sink, datastore, destination

**Stack**: A Backend and its supporting services, connected to the shared Collector.

### Surfaces

**Ingest endpoint**: The public, secretless, write-only OTLP endpoint that Sources send to. _Avoid_: collector endpoint, OTLP URL, ingress

**Query UI**: A Backend's public read-only interface. Never accepts writes. _Avoid_: dashboard, console

### Privacy

**Scrubbing**: Removal of sensitive fields in the Collector before export. Because reads are public, anything not scrubbed is world-visible. _Avoid_: anonymization, sanitization, filtering
