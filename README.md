# Fluent Bit + HyperDX Integration Demo

This demo shows how to run Fluent Bit v4.x with HyperDX integration using Docker Compose. The setup includes log collection, processing with OpenTelemetry formatting, and forwarding to HyperDX for observability.

For other Fluent Bit versions:
- **Fluent Bit 2.x**: See [HyperDX Fluent Bit 2.x documentation](https://www.hyperdx.io/docs/install/fluentbit#fluentbit-2x)
- **Other versions**: This configuration is compatible with Fluent Bit 3.1.x-4.x series

## What This Demo Does

- Runs Fluent Bit in a Docker container
- Collects logs from multiple sources:
  - Dummy logs (for testing)
  - HTTP endpoint (for real log ingestion)
- Processes logs with OpenTelemetry envelope format
- Forwards processed logs to HyperDX
- Provides monitoring endpoints

## Prerequisites

- Docker and Docker Compose installed
- A HyperDX account and API key
- Basic familiarity with Docker

## Quick Start

### 1. Get Your HyperDX API Key

1. Log into your HyperDX account
2. Navigate to your team settings
3. Copy your Ingestion API key

### 2. Set Up Environment Variable

You have two options to provide your API key:

**Option A: Export environment variable**
```bash
export HYPERDX_API_KEY=your_actual_api_key_here
```

**Option B: Create a .env file (recommended for demos)**
```bash
echo "HYPERDX_API_KEY=your_actual_api_key_here" > .env
```

### 3. Run the Demo

```bash
docker-compose -f docker-compose.fluentbit-hyperdx.yml up
```

To run in the background:
```bash
docker-compose -f docker-compose.fluentbit-hyperdx.yml up -d
```

## Testing the Setup

### 1. Check if Fluent Bit is Running

The container should start and you'll see logs indicating Fluent Bit is running. Look for messages like:
```
[info] [fluent bit] version=X.X.X
[info] [input:dummy:dummy.0] initializing
[info] [input:http:http.1] listening on 0.0.0.0:9880
```

### 2. Send Test Logs via HTTP

You can send logs to the HTTP input endpoint:

```bash
curl -X POST http://localhost:9880/test \
  -H "Content-Type: application/json" \
  -d '{"timestamp":"2024-01-01T12:00:00Z","level":"info","message":"Test log from HTTP endpoint"}'
```

### 3. Check HyperDX Dashboard

1. Log into HyperDX
2. Navigate to the search page
3. You should see logs from the `my-service` service appearing

## Available Ports

- **24224**: Forward input port (for Docker log driver)
- **9880**: HTTP input port (for sending logs via HTTP)

## Configuration Details

### Log Processing Pipeline

The setup includes:
1. **OpenTelemetry Envelope**: Wraps logs in OpenTelemetry format
2. **Content Modifier**: Adds service name metadata
3. **Dual Output**: Logs go to both stdout (for debugging) and HyperDX

### Service Attribution

All logs are automatically tagged with `service.name: my-service`. You can modify this in the `fluent-bit-4.yaml` configuration file.

## Troubleshooting

### Common Issues

**API Key Not Set**
```
Error: Authorization header missing or invalid
```
- Ensure `HYPERDX_API_KEY` environment variable is set
- Verify the API key is correct and active

**Connection Issues**
```
Error: failed to connect to in-otel.hyperdx.io
```
- Check internet connectivity
- Verify firewall allows outbound HTTPS on port 443

**Container Won't Start**
```
Error: port already in use
```
- Check if ports 24224 or 9880 are already in use
- Stop other services using these ports or modify the docker-compose file

### Viewing Logs

To see Fluent Bit logs:
```bash
docker-compose -f docker-compose.fluentbit-hyperdx.yml logs -f
```

To see only errors:
```bash
docker-compose -f docker-compose.fluentbit-hyperdx.yml logs | grep -i error
```

## Stopping the Demo

```bash
docker-compose -f docker-compose.fluentbit-hyperdx.yml down
```

To also remove the built image:
```bash
docker-compose -f docker-compose.fluentbit-hyperdx.yml down --rmi local
```

## Customization

### Changing the Service Name

Edit `fluent-bit-4.yaml` and modify the `value` under `content_modifier`:
```yaml
- name: content_modifier
  context: otel_resource_attributes
  action: upsert
  key: service.name
  value: your-custom-service-name  # Change this
```

### Adding More Log Sources

You can extend the configuration by adding more inputs to the `fluent-bit-4.yaml` file. See the [Fluent Bit documentation](https://docs.fluentbit.io/manual/pipeline/inputs) for available input plugins.

## Support

For issues with:
- **Fluent Bit configuration**: Check the [Fluent Bit documentation](https://docs.fluentbit.io/)
- **HyperDX integration**: Contact HyperDX support
- **This demo setup**: Review the troubleshooting section above or open a GitHub issue

## Next Steps

Once you've verified the demo works:
1. Integrate with your actual applications
2. Customize the log processing pipeline for your needs
3. Set up proper monitoring and alerting in HyperDX
4. Consider scaling the setup for production use
