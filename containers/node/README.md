# Node

Node-based image for our organization. This vesion satisfies our requirements for observability/telemetry.

## Opentelemetry

The platform MUST be [auto-instrumented](https://opentelemetry.io/docs/zero-code/js/#setup). Because of this two packages are added for opentelemetry.

- [@opentelemetry/api](https://www.npmjs.com/package/@opentelemetry/api) on version: 1.9.0
- [@opentelemetry/auto-instrumentations-node](https://www.npmjs.com/package/@opentelemetry/auto-instrumentations-node) on version: 0.52.0
- [@opentelemetry/winston-transport](https://www.npmjs.com/package/@opentelemetry/winston-transport) on version: 0.7.0

## Technical debts

- [ ] Automate build process.
