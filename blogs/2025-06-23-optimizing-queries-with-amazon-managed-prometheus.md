---
title: "Optimizing Queries with Amazon Managed Prometheus"
url: "https://aws.amazon.com/blogs/mt/optimizing-queries-with-amazon-managed-prometheus/"
date: "Mon, 23 Jun 2025 14:58:40 +0000"
author: "Rodrigue Koffi"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-grafana/feed/"
---
<h1>Introduction</h1> 
<p>In today’s cloud-native environments, organizations rely on metrics monitoring to maintain application reliability and performance. <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/what-is-Amazon-Managed-Service-Prometheus.html">Amazon Managed Service for Prometheus</a> serves as a tool for storing and analyzing application and infrastructure metrics. As applications and platforms evolve, teams often discover opportunities to optimize their metrics querying patterns. Common scenarios like expanding service deployments, growing infrastructure footprint, or increasingly complex <a href="https://prometheus.io/docs/prometheus/latest/querying/basics/">Prometheus Query Language (PromQL)</a> queries can lead to processing more samples than intended. Even seemingly straightforward queries might scan billions of samples in a single execution, making it valuable to understand query optimization techniques. By implementing best practices and leveraging built-in governance controls, teams can ensure efficient and cost-effective metric analysis while maintaining comprehensive monitoring coverage.</p> 
<p>This is a blog series on governance controls with Amazon Managed Service for Prometheus. In this post, we’ll focus on features that provide PromQL query insights and control capabilities. Specifically, we’ll cover the <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/query-insights-control.html">Query Logging and Query Thresholds</a> features, which help you monitor, manage, and optimize your PromQL queries.</p> 
<h1>Overview</h1> 
<p>For this blog series, the scenario is a centralized observability architecture. Example Corp, a multinational company, is collecting all platform and application metrics in Prometheus format with a centralized view from multiple AWS accounts.</p> 
<p>The central observability team at Example Corp provides Amazon Managed Service for Prometheus as the metrics platform to various application teams. However, they lack visibility into how teams are querying their metrics. Without visibility into tenant-specific querying patterns, unexpected spikes can overwhelm workspace capacity, causing workspace-wide disruptions. While isolating high-volume tenants to dedicated workspaces prevents these issues, it breaks critical cross-service visibility needed for troubleshooting distributed systems. Additionally, the platform team needs granular cost attribution capabilities to enable team-specific budget management. The solution requires maintaining shared infrastructure while providing tenant isolation, unified data access, and precise usage tracking.</p> 
<p>Let’s explore how the Query Logging and Query Thresholds features in Amazon Managed Service for Prometheus can help Example Corp address these challenges and optimize their PromQL querying.</p> 
<h1>PromQL refresher</h1> 
<p>Before diving into the topic, let’s have a quick look at the Prometheus Query Language (PromQL). PromQL is the powerful querying language that serves as the backbone for data retrieval and analysis in Prometheus and Amazon Managed Service for Prometheus (AMP). At its core, PromQL works with time series data, where each data point combines a metric name, a set of labels (key-value pairs), and timestamped values. This unique structure allows for highly flexible and precise metric querying.</p> 
<p>When working with PromQL, you’ll encounter three primary query types. Instant queries provide the latest values for your metrics, range queries fetch data over a specified time window, and subqueries allow for complex nested operations. This versatility makes PromQL particularly powerful for both real-time monitoring and historical analysis.</p> 
<p>Here’s a few sample PromQL queries:</p> 
<pre><code class="lang-bash">
# 1. INSTANT QUERIES
# These return the most recent value for each time series
# Simple metric selection
node_cpu_seconds_total
# With label matching
node_cpu_seconds_total{cpu="0", mode="idle" }

# 2. RANGE QUERIES
# These return a set of values over a time range, denoted by '[time_range]'
# Basic range query - last 5 minutes of data
node_cpu_seconds_total[5m]
# Calculating rate over time
rate(node_cpu_seconds_total[5m])

# 3. FILTERING QUERIES
# Various ways to filter metrics
# Exact match
http_requests_total{status="200"}
# Regex match (matches 200, 201, etc)
http_requests_total{status=~"2.."}

# 4. AGGREGATION QUERIES
# Combine multiple time series
# Sum of values grouped by instance
sum by(instance) (node_cpu_seconds_total)
# Count of time series grouped by status
count by(status) (http_requests_total)

# 5. SUBQUERIES
# Queries operating on results of range vectors
# Rate of CPU usage over 5m, sampled every 1m for the last hour
rate(node_cpu_seconds_total[5m])[1h:1m]
# Average CPU usage with subquery
avg_over_time(rate(node_cpu_seconds_total[5m])[30m:1m])
</code></pre> 
<p>As of May 2025, Amazon Managed Service for Prometheus <a href="https://aws.amazon.com/about-aws/whats-new/2025/05/amazon-managed-service-prometheus-95-day-queries/">supports queries spanning up to 95 days</a>, a significant expansion from the previous 32 days limit. This extended query range capability makes it even more critical to understand and optimize query patterns, as longer time ranges can exponentially increase the number of samples processed. For instance, a subquery like <code>rate(node_cpu_seconds_total[5m])[95d:1m]</code> could potentially process billions of samples across a three-month period.</p> 
<p>When issuing these types of PromQL queries without proper consideration, they can inadvertently trigger significant cost implications. For instance, a simple instant query like <code>node_cpu_seconds_total</code> without any label constraints could match thousands of time series across all nodes. The situation becomes even more problematic with regex-based filtering queries like <code>status=~"2.."</code>, that might match more series than intended, and aggregation queries <code>sum by(instance)</code> that need to process all matched series before producing results. The most costly scenarios often involve subqueries, where operations like <code>rate(node_cpu_seconds_total[5m])[1h:1m]</code> create a compound effect – first processing the inner range vector across all matched series, then performing additional calculations at each step interval over an hour, potentially scanning many samples in a single execution. Without proper visibility into how these queries translate to processed samples and associated costs, teams often discover these issues only after seeing unexpected spikes in their billing statements.</p> 
<h1>Configuring query logging</h1> 
<p>AWS has introduced query logging capabilities for Amazon Managed Service for Prometheus, enabling customers to gain enhanced visibility into their PromQL query execution patterns and associated costs. This new feature helps organizations like Example Corp optimize their PromQL querying by providing detailed logs of queries that exceed specified thresholds. Operators can enable the query logging feature for an Amazon Managed Service for Prometheus workspace using the following <a href="https://docs.aws.amazon.com/cli/">AWS CLI</a> command:</p> 
<pre><code class="lang-bash">
aws amp create-query-logging-configuration \
  --destinations '[{"cloudWatchLogs":{"logGroupArn":"arn:aws:logs:$AWS_REGION:123456789012:log-group:/amp/query-logs/ws-12345678:*"},"filters":{"qspThreshold":1000000}}]' \
  --workspace-id ws-12345678
</code></pre> 
<p>After configuration, operators can verify their setup using the describe command:</p> 
<pre><code class="lang-bash">
aws amp describe-query-logging-configuration —workspace-id ws-12345678
</code></pre> 
<p>Alternatively, the Example Corp team can also configure the query insights thresholds through the <a href="https://console.aws.amazon.com/prometheus/home">AWS Management Console</a></p> 
<div class="wp-caption alignnone" id="attachment_61106" style="width: 1440px;">
 <img alt="Figure 1. Figure showing screenshot of query insights" class="wp-image-61106 size-full" height="632" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/06/05/Picture1-3.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-61106">Figure 1. Query insights configuration using AWS management console</p>
</div> 
<p>This returns the active configuration details, confirming parameters such as the workspace ID, logging status, and QSP threshold settings. Once enabled, the feature begins logging queries to CloudWatch, where operators can examine detailed execution metrics. For instance, a typical query log entry provides insights:</p> 
<pre><code class="lang-bash">
{
	"workspaceId": "ws-12345678",
	"message": {
		"query": "rate(node_cpu_seconds_total[5m])",
		"queryType": "range",
		"time": null,
		"start": "1747581840",
		"end": "1747668240",
		"step": "120",
		"userAgent": "Grafana/10.4.1",
		"dashboardUid": "rYdddlPWk",
		"panelId": "191",
		"samplesProcessed":400292
	},
	"component": "query-frontend"
}
</code></pre> 
<p>These logs reveal crucial information about each query’s execution, including the number of samples processed, data transfer size, and execution time. Example Corp can use this information to identify expensive queries and optimize their monitoring costs. The configuration can be adjusted using the update command, or the AWS Management Console:</p> 
<pre><code class="lang-bash">
aws amp update-query-logging-configuration \
  --workspace-id ws-12345678 \
  --destinations '[{...}]'
</code></pre> 
<h1>Deriving insights from query logs</h1> 
<p>With <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CloudWatchLogs-Field-Indexing.html">Amazon CloudWatch Logs fields indexes</a>, Example Corp can create indexes on the “workspaceId” and “query” fields. With those fields indexed, they will be able to run fast and efficient CloudWatch Logs Insights queries across multiple log groups. CloudWatch Logs indexes only the log events ingested after an index policy is created. It doesn’t index log events ingested before the policy was created. After you create a field index, each matching log event remains indexed for 30 days from the log event’s ingestion time.</p> 
<h2>Creating field indexes using AWS CLI</h2> 
<p>Run the below AWS CLI&nbsp;command to create a field index policy for the log group which stores your query logs. You can also create the indexes from AWS Console using Create a log-group level field index policy</p> 
<pre><code class="lang-bash">
aws logs put-index-policy \
  --log-group-identifier /amp/query-logs \
  --policy-document "{\"Fields\": [\"workspaceId\", \"query\"]}" \
  --region $AWS_REGION
</code></pre> 
<p>They can also setup indexes at an account level if they have multiple log groups concerned. All the log events matching these keys will be matched regardless of their log groups source.</p> 
<h2>Querying CloudWatch logs examples</h2> 
<p>Below are sample CloudWatch Logs Insights queries with their corresponding output. These queries can be extended with more complex conditions to match your use case.</p> 
<ul> 
 <li>Find PromQL queries processing large amounts of samples.</li> 
</ul> 
<pre><code>
fields message.dashboardUid as DashboardUid, message.panelId as PaneId, message.query as Query
| filter message.samplesProcessed &gt; 10000
| stats count(*) as frequency,
avg(message.samplesProcessed) as avg_samples,
max(message.samplesProcessed) as max_samples,
avg(message.step) as avg_step
by DashboardUid, PaneId, Query
| sort avg_samples desc
| limit 10
</code></pre> 
<div class="wp-caption alignnone" id="attachment_61150" style="width: 1440px;">
 <img alt="Figure 2: CloudWatch Logs Insights query result" class="size-full wp-image-61150" height="583" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/06/09/Picture2-7.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-61150">Figure 2: CloudWatch Logs Insights query result</p>
</div> 
<ul> 
 <li>Query to identify most frequently executed queries and their types</li> 
</ul> 
<pre><code>
stats count(*) as queryCount by message.query, message.queryType
| sort queryCount desc
| limit 20
</code></pre> 
<ul> 
 <li>Query to analyze which clients are making queries</li> 
</ul> 
<pre><code>
stats count(*) as requests by message.userAgent
| sort requests desc
</code></pre> 
<ul> 
 <li>Query to track Grafana Dashboard specific usage</li> 
</ul> 
<pre><code>
stats count(*) as queries, avg(message.samplesProcessed) as avgSamples
by message.dashboardUid, message.panelId
| sort queries desc
</code></pre> 
<p>Note that all these queries can be issued from an <a href="https://docs.aws.amazon.com/grafana/latest/userguide/what-is-Amazon-Managed-Service-Grafana.html">Amazon Managed Grafana</a> workspace. Storing and querying logs will incur charges at <a href="https://aws.amazon.com/cloudwatch/pricing/">CloudWatch rates</a>.</p> 
<h1>Implementing query controls</h1> 
<p>Beyond getting visibility, Example Corp can also take preventive measures to control their usage. Amazon Managed Service for Prometheus offers the ability to limit query cost by providing limits on the amount of Query Samples Processed (QSP) that can be used by a single query. You can configure two types of thresholds for QSP – warning and error – to help manage and control query costs effectively.<br /> When a query hits the warning threshold, a warning message appears in the API query response. For queries viewed through Amazon Managed Grafana, the warning will be visible in the UI, helping users identify expensive queries. Queries that hit the error threshold are not charged and will be rejected with an error.</p> 
<p>To enable this feature in Amazon Managed Grafana,</p> 
<ul> 
 <li>Navigate to the Home page, choose “<strong>Data Sources</strong>” under “<strong>Connections</strong>” section. Next, choose your AMP workspace to open the settings page.</li> 
</ul> 
<div class="wp-caption alignnone" id="attachment_61110" style="width: 1440px;">
 <img alt="Figure 3. Figure showing screenshot of Grafana configuration" class="wp-image-61110 size-full" height="1100" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/06/05/Picture3-2.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-61110">Figure 3. Figure showing screenshot of Grafana configuration</p>
</div> 
<ul> 
 <li>Scroll down to “<strong>Other</strong>” section under the “<strong>Settings</strong>” tab. Next under “<strong>Custom query parameters</strong>”, set two custom query parameters with their values as<br /> <code>max_samples_processed_warning_threshold=100000</code> and<br /> <code>max_samples_processed_error_threshold=1000000</code> Here multiple parameters should be concatenated together with an <code>&amp;</code>.</li> 
 <li>Save the data source configuration</li> 
</ul> 
<div class="wp-caption alignnone" id="attachment_61147" style="width: 1440px;">
 <img alt="Figure 4: Configuring query parameters for warning and error threshold" class="size-full wp-image-61147" height="1100" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/06/09/Picture4-4.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-61147">Figure 4: Configuring query parameters for warning and error threshold</p>
</div> 
<p>With this feature, any queries that are running against the Amazon Managed Service for Prometheus data source from Grafana will be subject to a warning threshold of 100,000 Query Samples Processed (QSP) and an error threshold of 1,000,000 QSP.<br /> Example Corp can now proactively manage query costs by receiving a warning when a query exceeds the warning threshold and prevent running queries that would exceed the error threshold.</p> 
<h2>Implementing Query Controls: A Practical Example</h2> 
<p>Let’s see how this works with a <a href="https://grafana.com/docs/grafana/latest/dashboards/">Grafana dashboard</a> example. Imagine two <a href="https://grafana.com/docs/grafana/latest/panels-visualizations/panel-overview/">panels</a> each using a different PromQL query:</p> 
<p><strong>Panel 1 (Not Optimized Query)</strong>: <code>rate(node_cpu_seconds_total[7d:1m])[5m:1s]</code></p> 
<p>This query calculates the maximum rate of CPU usage for each instance, CPU, and mode over a 7-day period with 1-minute resolution. It first computes the per-second rate of increase for node_cpu_seconds_total over 5-minute windows, then finds the maximum value for each of these rates over the entire 7-day period. This query is extremely expensive as it processes a huge amount of data, potentially millions of samples for a multi-node cluster with multiple CPUs, leading to very high samples processed.</p> 
<p><strong>Panel 2 (Optimized Query)</strong>: <code>max by (instance)(rate(node_cpu_seconds_total{mode="user"}[5m]))</code></p> 
<p>This query calculates the maximum rate of CPU usage in user mode for each instance over the last 5 minutes. It filters the data to include only CPU time spent in user mode, then calculates the per-second rate. The max by(instance)aggregation combines the rates across all CPU cores for each instance, resulting in a single value per node for maximum user CPU utilization. This significantly reduces the number of samples processed and focuses on recent, relevant data, making the query much more efficient.<br /> When loading the Grafana dashboard, the unoptimized query in Panel 1 immediately triggers an error message as it attempts to process over 1 million samples, exceeding the error threshold. The query is rejected with an error, and importantly, no charges are incurred as the query is blocked before processing.</p> 
<div class="wp-caption alignnone" id="attachment_61148" style="width: 1391px;">
 <img alt="Figure 5: Grafana dashboard showing unoptimized and optimized PromQL query examples" class="size-full wp-image-61148" height="727" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/06/09/Picture5-7.png" width="1381" />
 <p class="wp-caption-text" id="caption-attachment-61148">Figure 5: Grafana dashboard showing unoptimized and optimized PromQL query examples</p>
</div> 
<p>In contrast, the optimized query in Panel 2 processes fewer than 100,000 samples, running well below the warning threshold and loads successfully.</p> 
<h1>Conclusion</h1> 
<p>In this blog, we explored how Example Corp leverages query governance controls in Amazon Managed Service for Prometheus to optimize their PromQL querying patterns and costs. We demonstrated how organizations can implement comprehensive query monitoring through query logging, utilize CloudWatch Logs for detailed analytics, and enforce QSP thresholds to prevent costly query execution. We showed how teams can significantly reduce resource consumption while maintaining effective monitoring capabilities. These governance controls provide deeper visibility into query patterns and enable proactive cost management, ensuring a sustainable and cost-effective monitoring strategy that scales with your organization’s needs. As cloud-native environments grow in complexity, these governance controls become increasingly crucial for maintaining reliable and efficient observability solutions.</p> 
<p>To learn more about AWS Observability, checkout the <a href="https://aws-observability.github.io/observability-best-practices/">AWS Observability Best Practices Guide</a>.<br /> To get hands-on experience with AWS Observability services, check out the <a href="https://catalog.workshops.aws/observability/en-US">One Observability Workshop</a>.</p>
