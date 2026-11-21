# Node.js base image

The [Dockerfile](Dockerfile) starts from Node.js 20.18.0 on Alpine 3.20 and runs as the non-root user `company`. It configures a user-local npm prefix and sets `NODE_PATH` for modules installed there.

## OpenTelemetry

| Package | Version declared in the Dockerfile |
| --- | --- |
| `@opentelemetry/api` | `1.9.0` |
| `@opentelemetry/auto-instrumentations-node` | `0.52.0` |
| `@opentelemetry/winston-transport` | `0.7.0` |

The npm install step is missing a shell separator before the Winston transport installation. Fix that command before validating the image build. From the repository root, use:

```sh
docker build -t borneo-node:local ./containers/node
```

To enable automatic instrumentation in a consuming application, preload `@opentelemetry/auto-instrumentations-node/register` through `NODE_OPTIONS` and configure the service name and exporter in the application environment. [OpenTelemetry zero-code instrumentation for JavaScript](https://opentelemetry.io/docs/zero-code/js/)

## Maintenance

- [ ] Automate the image build.
