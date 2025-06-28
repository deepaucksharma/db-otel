# New Relic Database Intelligence MVP

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-enabled-orange)](https://opentelemetry.io)
[![Production Ready](https://img.shields.io/badge/Production-Ready-green)](DEPLOYMENT.md)

## What This Is

A production-ready OpenTelemetry Collector configuration that safely collects database execution plans and performance metrics, sending them to New Relic for analysis. Built on the principle of "Configure, Don't Build" - leveraging standard OTEL components with minimal custom code.

### Key Features

- 🛡️ **Production-Safe**: Read-replica only, with timeouts and circuit breakers
- 🚀 **High Availability**: Leader election support for multi-instance deployment
- 🔒 **Security First**: PII sanitization, credential management, network policies
- 📊 **Observable**: Comprehensive metrics, alerts, and SLO tracking
- 🎯 **Easy Setup**: One-click quickstart with interactive configuration
- 🔍 **Silent Failure Detection**: Monitors NrIntegrationError events for data loss
- 📈 **Cardinality Management**: Multi-stage query normalization and fingerprinting
- 🔗 **Entity Correlation**: Automatic database entity synthesis in New Relic

### Current Capabilities

| Feature | PostgreSQL | MySQL | MongoDB |
|---------|:----------:|:-----:|:-------:|
| Query Metadata | ✅ | ✅ | 🔄 |
| Execution Plans | 🔄 | ❌ | ❌ |
| Performance Metrics | ✅ | ✅ | 🔄 |
| Zero-Impact Collection | ✅ | ✅ | ✅ |

✅ = Supported, 🔄 = Metadata only, ❌ = Not available

## What This Isn't (Yet)

- **Not** automatic APM-to-database correlation (requires manual correlation)
- **Not** real-time query analysis (batch collection only, 5-min intervals)
- **Not** zero-configuration (requires database prerequisites)

## Quick Start

### One-Command Setup

```bash
# Clone and run quickstart
git clone https://github.com/newrelic/database-intelligence-mvp
cd database-intelligence-mvp
./quickstart.sh all
```

### Manual Setup Overview

1. **Check Prerequisites**: Your database needs specific extensions enabled (see [PREREQUISITES.md](PREREQUISITES.md))
2. **Choose Deployment**: 
   - Local: Docker Compose with HA support
   - Kubernetes: StatefulSet or HA Deployment with leader election
3. **Configure Safety**: Read-replica endpoints, read-only users required
4. **Start Collecting**: See your first data in New Relic within 5 minutes

### Critical Safety Warning

⚠️ **This collector MUST connect to read-replicas only**. Never point it at your primary database. All configurations include safety timeouts, but replica targeting is your first line of defense.

## Architecture Philosophy

We follow three core principles:

1. **Safety Over Features**: Every query has a timeout, every collection has a limit
2. **Honest Limitations**: We clearly document what doesn't work
3. **Incremental Value**: Start simple, enhance gradually

## What's Inside

### Components

- **Receivers**: 
  - `sqlquery`: Safe metadata collection from PostgreSQL/MySQL
  - `filelog`: Zero-impact log parsing for auto_explain
- **Processors**: 
  - `memory_limiter`: OOM protection
  - `transform/sanitize_pii`: Security hardening
  - `transform/query_normalization`: Cardinality reduction
  - `resource/entity_synthesis`: New Relic entity creation
  - `circuitbreaker`: Per-database failure isolation
  - `adaptivesampler`: Priority-based intelligent sampling
- **Exporters**: 
  - `otlp/newrelic`: HTTP/protobuf with zstd compression

### Implementation Status

| Component | Status | Details |
|-----------|--------|---------|  
| Core Collector Config | ✅ Complete | Production-ready YAML |
| High Availability | ✅ Complete | Leader election support |
| Deployment Automation | ✅ Complete | Docker, K8s, Helm |
| Safety Testing | ✅ Complete | Comprehensive test suite |
| Monitoring Setup | ✅ Complete | Prometheus rules & alerts |
| Documentation | ✅ Complete | 10+ guides available |

## Production Deployment

### Prerequisites

- PostgreSQL: `pg_stat_statements` extension
- MySQL: Performance Schema enabled
- New Relic: License key and OTLP endpoint
- Infrastructure: Kubernetes 1.19+ or Docker 20.10+

### Deployment Options

#### Option 1: Kubernetes HA (Recommended)
```bash
kubectl apply -f deploy/k8s/ha-deployment.yaml
```

#### Option 2: Docker Compose
```bash
cd deploy/docker
docker-compose up -d
```

#### Option 3: Helm (Coming Soon)
```bash
helm install db-intelligence ./deploy/helm
```

## Resource Requirements

| Component | CPU | Memory | Storage | Network |
|-----------|-----|--------|---------|----------|
| Collector (each) | 500m-1000m | 512Mi-1Gi | 10Gi | <10Mbps |
| Database Impact | <1% | - | - | 1 connection |

## New Relic Integration Monitoring

### Critical Validation Query
```sql
-- Monitor for silent failures (run every 5 minutes)
SELECT count(*) FROM NrIntegrationError 
WHERE newRelicFeature = 'Metrics' 
  AND message LIKE '%database%'
SINCE 5 minutes ago
```

### Key Metrics to Monitor
- **NrIntegrationError Events**: Silent failure detection
- **Cardinality Warnings**: Query pattern explosion
- **Entity Synthesis**: Database entity creation
- **Circuit Breaker State**: Per-database health
- **Sampling Effectiveness**: Data reduction rates

Import `monitoring/dashboard-config.json` for comprehensive monitoring.

## Documentation

### Essential Guides

- 📋 [PREREQUISITES.md](PREREQUISITES.md) - Database setup requirements
- 🏗️ [ARCHITECTURE.md](ARCHITECTURE.md) - Technical design decisions
- 🚀 [DEPLOYMENT.md](DEPLOYMENT.md) - Production deployment guide
- 🔧 [CONFIGURATION.md](CONFIGURATION.md) - Detailed configuration reference
- 📊 [OPERATIONS.md](OPERATIONS.md) - Daily operations & monitoring
- ⚠️ [LIMITATIONS.md](LIMITATIONS.md) - Known limitations & workarounds
- 🔍 [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues & solutions
- 🔗 [NEWRELIC_INTEGRATION.md](docs/NEWRELIC_INTEGRATION.md) - New Relic integration guide

### Additional Resources

- 🎯 [EVOLUTION.md](EVOLUTION.md) - Roadmap and future enhancements
- 🤝 [CONTRIBUTING.md](CONTRIBUTING.md) - How to contribute
- 📝 [CHANGELOG.md](CHANGELOG.md) - Release history and fixes

## Support & Community

- **Issues**: [GitHub Issues](https://github.com/newrelic/database-intelligence-mvp/issues)
- **Discussions**: [GitHub Discussions](https://github.com/newrelic/database-intelligence-mvp/discussions)
- **Slack**: #database-observability channel
- **Email**: database-intelligence@newrelic.com

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- OpenTelemetry Community for the collector framework
- PostgreSQL and MySQL communities for observability features
- New Relic customers for feedback and use cases