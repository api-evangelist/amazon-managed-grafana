# Amazon Managed Grafana (amazon-managed-grafana)
Amazon Managed Grafana is a fully managed service for open source Grafana developed in collaboration with Grafana Labs. It enables interactive data visualizations and dashboards for operational metrics, logs, and traces from multiple sources including AWS services, third-party ISVs, and on-premises data. The service handles provisioning, setup, scaling, and maintenance of Grafana, allowing teams to focus on creating dashboards and analyzing data.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-managed-grafana/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Dashboards, Monitoring, Observability, Visualization

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### Amazon Managed Grafana API
The Amazon Managed Grafana API provides programmatic access to create and manage Grafana workspaces, users, SAML configurations, and workspace API keys for managed Grafana deployments. Covers workspace lifecycle management, authentication configuration, license association, and access control across all managed Grafana resources.

**Human URL:** [https://aws.amazon.com/grafana/](https://aws.amazon.com/grafana/)

#### Tags:

 - Dashboards, Monitoring, Observability, Visualization

#### Properties

- [Documentation](https://docs.aws.amazon.com/grafana/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-managed-grafana-openapi-original.yaml)
- [GettingStarted](https://aws.amazon.com/grafana/getting-started/)
- [Pricing](https://aws.amazon.com/grafana/pricing/)
- [FAQ](https://aws.amazon.com/grafana/faqs/)

## Common Properties

- [Portal](https://aws.amazon.com/grafana/)
- [Documentation](https://docs.aws.amazon.com/grafana/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/mt/tag/amazon-managed-grafana/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/grafana/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)
- [SpectralRules](rules/amazon-managed-grafana-spectral-rules.yml)
- [Vocabulary](vocabulary/amazon-managed-grafana-vocabulary.yaml)
- [NaftikoCapability](capabilities/observability-dashboard-workflow.yaml)

## Features

| Name | Description |
|------|-------------|
| Fully Managed Grafana | Provision and manage Grafana workspaces without infrastructure setup, patching, or scaling. |
| SSO and SAML Integration | Configure SAML-based single sign-on for workspace authentication and user management. |
| Multi-Source Data Visualization | Connect to AWS services, third-party ISVs, and on-premises data sources in a single dashboard. |
| Workspace API Keys | Create and manage API keys for programmatic access to Grafana workspace resources. |
| License Management | Associate and manage Grafana Enterprise licenses for advanced features. |
| VPC Integration | Deploy workspaces within a VPC for secure private access to data sources. |
| Role-Based Access Control | Manage user and group permissions within Grafana workspaces using role assignments. |

## Use Cases

| Name | Description |
|------|-------------|
| Infrastructure Monitoring | Visualize AWS infrastructure metrics from CloudWatch, EC2, RDS, and other services in unified dashboards. |
| Container Observability | Monitor Kubernetes and ECS workloads using Prometheus and CloudWatch Container Insights data sources. |
| Application Performance Monitoring | Track application latency, error rates, and throughput with custom dashboards and alerting. |
| Business Metrics Dashboards | Build executive dashboards combining operational and business metrics from multiple data sources. |
| Security and Compliance Monitoring | Visualize security findings and compliance metrics from AWS Security Hub and GuardDuty. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon CloudWatch | Visualize CloudWatch metrics and logs natively in Grafana dashboards. |
| Amazon Managed Service for Prometheus | Query Prometheus metrics from AMP workspaces as a Grafana data source. |
| AWS X-Ray | Trace application requests and visualize distributed tracing data in Grafana. |
| Amazon OpenSearch Service | Query OpenSearch indices for log analytics and visualization. |
| Amazon Timestream | Visualize time-series data stored in Amazon Timestream. |
| AWS IAM Identity Center | Integrate with IAM Identity Center for centralized user authentication. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Amazon Managed Grafana OpenAPI](openapi/amazon-managed-grafana-openapi-original.yaml)

### JSON Schema

125 schema files available in the [json-schema/](json-schema/) directory.

### JSON Structure

125 structure files available in the [json-structure/](json-structure/) directory.

### JSON-LD

- [Amazon Managed Grafana Context](json-ld/amazon-managed-grafana-context.jsonld)

### Examples

125 example files available in the [examples/](examples/) directory.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Amazon Managed Grafana](capabilities/shared/managed-grafana.yaml) — 21 operations for workspace management, authentication, and access control

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Observability Dashboard Workflow](capabilities/observability-dashboard-workflow.yaml) | Amazon Managed Grafana | 5 | Platform Engineer, Operations Engineer |

## Vocabulary

- [Amazon Managed Grafana Vocabulary](vocabulary/amazon-managed-grafana-vocabulary.yaml) — Unified taxonomy mapping 5 resources, 7 actions, 1 workflow, and 2 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon Managed Grafana Spectral Rules](rules/amazon-managed-grafana-spectral-rules.yml) — 18 rules across 7 categories enforcing Amazon Managed Grafana API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
