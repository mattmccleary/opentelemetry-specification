# OpenTelemetry Environment Variable Specification

The goal of this specification is to unify the environment variable names between different OpenTelemetry SDK implementations. SDKs MAY choose to allow configuration via the environment variables in this specification, but are not required to. If they do, they SHOULD use the names listed in this document.

For variables which accept a known value out of a set, i.e., an enum value, SDK implementations MAY support additional values not listed here.
For variables accepting an enum value, if the user provides a value the SDK does not recognize, the SDK MUST generate a warning and gracefully ignore the setting.

## General SDK Configuration

| Name                     | Description                                       | Default                           | Notes                               |
| ------------------------ | ------------------------------------------------- | --------------------------------- | ----------------------------------- |
| OTEL_RESOURCE_ATTRIBUTES | Key-value pairs to be used as resource attributes |                                   | See [Resource SDK](./resource/sdk.md#specifying-resource-information-via-an-environment-variable) for more details. |
| OTEL_LOG_LEVEL           | Log level used by the SDK logger                  | "info"                            |                                     |
| OTEL_PROPAGATORS         | Propagators to be used as a comma separated list  | "tracecontext,baggage"            | Values MUST be deduplicated in order to register a `Propagator` only once. |
| OTEL_TRACE_SAMPLER       | Sampler to be used for traces                     | "parentbased_always_on"                       | See [Sampling](./trace/sdk.md#sampling) |
| OTEL_TRACE_SAMPLER_ARG   | String value to be used as the sampler argument   |                                   | The specified value will only be used if OTEL_TRACE_SAMPLER is set. Each Sampler type defines its own expected input, if any. Invalid or unrecognized input MUST be logged and MUST be otherwise ignored, i.e. the SDK MUST behave as if OTEL_TRACE_SAMPLER_ARG is not set.  |

Known values for OTEL_PROPAGATORS are:

- `"tracecontext"`: [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- `"baggage"`: [W3C Baggage](https://www.w3.org/TR/baggage/)
- `"b3"`: [B3 Single](https://github.com/openzipkin/b3-propagation#single-header)
- `"b3multi"`: [B3 Multi](https://github.com/openzipkin/b3-propagation#multiple-headers)
- `"jaeger"`: [Jaeger](https://www.jaegertracing.io/docs/1.21/client-libraries/#propagation-format)
- `"xray"`: [AWS X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html#xray-concepts-tracingheader) (_third party_)
- `"ottracer"`: [Lightstep](https://github.com/lightstep/lightstep-tracer-java-common/blob/master/common/src/main/java/com/lightstep/tracer/shared/TextMapPropagator.java) (_third party_)

Known values for `OTEL_TRACE_SAMPLER` are:

- `"always_on"`: `AlwaysOnSampler`
- `"always_off"`: `AlwaysOffSampler`
- `"traceidratio"`: `TraceIdRatioBased`
- `"parentbased_always_on"`: `ParentBased(root=AlwaysOnSampler)`
- `"parentbased_always_off"`: `ParentBased(root=AlwaysOffSampler)`
- `"parentbased_traceidratio"`: `ParentBased(root=TraceIdRatioBased)`

Depending on the value of `OTEL_TRACE_SAMPLER`, `OTEL_TRACE_SAMPLER_ARG` may be set as follows:

- For `traceidratio` and `parentbased_traceidratio` samplers: Sampling probability, a number in the [0..1] range, e.g. "0.25". Default is 1.0 if unset.

## Batch Span Processor

| Name                           | Description                                    | Default | Notes                                                 |
| ------------------------------ | ---------------------------------------------- | ------- | ----------------------------------------------------- |
| OTEL_BSP_SCHEDULE_DELAY_MILLIS | Delay interval between two consecutive exports | 5000    |                                                       |
| OTEL_BSP_EXPORT_TIMEOUT_MILLIS | Maximum allowed time to export data            | 30000   |                                                       |
| OTEL_BSP_MAX_QUEUE_SIZE        | Maximum queue size                             | 2048    |                                                       |
| OTEL_BSP_MAX_EXPORT_BATCH_SIZE | Maximum batch size                             | 512     | Must be less than or equal to OTEL_BSP_MAX_QUEUE_SIZE |

## Span Collection Limits

| Name                            | Description                          | Default | Notes |
| ------------------------------- | ------------------------------------ | ------- | ----- |
| OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT | Maximum allowed span attribute count | 1000    |       |
| OTEL_SPAN_EVENT_COUNT_LIMIT     | Maximum allowed span event count     | 1000    |       |
| OTEL_SPAN_LINK_COUNT_LIMIT      | Maximum allowed span link count      | 1000    |       |

## OTLP Exporter

See [OpenTelemetry Protocol Exporter Configuration Options](./protocol/exporter.md).

## Jaeger Exporter

| Name                            | Description                                       | Default                                                                                          |
| ------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| OTEL_EXPORTER_JAEGER_AGENT_HOST | Hostname for the Jaeger agent                     | "localhost"                                                                                      |
| OTEL_EXPORTER_JAEGER_AGENT_PORT | Port for the Jaeger agent                         | 6832                                                                                             |
| OTEL_EXPORTER_JAEGER_ENDPOINT   | HTTP endpoint for Jaeger traces                   | <!-- markdown-link-check-disable --> "http://localhost:14250"<!-- markdown-link-check-enable --> |
| OTEL_EXPORTER_JAEGER_USER       | Username to be used for HTTP basic authentication | -                                                                                                |
| OTEL_EXPORTER_JAEGER_PASSWORD   | Password to be used for HTTP basic authentication | -                                                                                                |

## Zipkin Exporter

| Name                          | Description                | Default                                                                                                      |
| ----------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| OTEL_EXPORTER_ZIPKIN_ENDPOINT | Endpoint for Zipkin traces | <!-- markdown-link-check-disable --> "http://localhost:9411/api/v2/spans"<!-- markdown-link-check-enable --> |

## Prometheus Exporter

| Name                          | Description                     | Default                      |
| ----------------------------- | --------------------------------| ---------------------------- |
| OTEL_EXPORTER_PROMETHEUS_HOST | Host used by the Prometheus exporter | All addresses: "0.0.0.0"|
| OTEL_EXPORTER_PROMETHEUS_PORT | Port used by the Prometheus exporter | 9464                    |

## Exporter Selection

We define environment variables for setting a single exporter per signal.

| Name          | Description                                                                  | Default |
| ------------- | ---------------------------------------------------------------------------- | ------- |
| OTEL_TRACE_EXPORTER | Trace exporter to be used | "otlp"  |
| OTEL_METRICS_EXPORTER | Metrics exporter to be used | "otlp"  |

Known values for OTEL_TRACE_EXPORTER are:

- `"otlp"`: [OTLP](./protocol/otlp.md)
- `"jaeger"`: [Jaeger gRPC](https://www.jaegertracing.io/docs/1.21/apis/#protobuf-via-grpc-stable)
- `"zipkin"`: [Zipkin](https://zipkin.io/zipkin-api/) (Defaults to [protobuf](https://github.com/openzipkin/zipkin-api/blob/master/zipkin.proto) format)

Known values for OTEL_METRICS_EXPORTER are:

- `"otlp"`: [OTLP](./protocol/otlp.md)
- `"prometheus"`: [Prometheus](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md)

## Language Specific Environment Variables

To ensure consistent naming across projects, this specification recommends that language specific environment variables are formed using the following convention:

```
OTEL_{LANGUAGE}_{FEATURE}
```
