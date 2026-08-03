# Amazon Managed Grafana (amazon-managed-grafana)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
