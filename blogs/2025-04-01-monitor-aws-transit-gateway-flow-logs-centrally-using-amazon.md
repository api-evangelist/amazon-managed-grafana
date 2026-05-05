---
title: "Monitor AWS Transit Gateway Flow Logs centrally using Amazon Managed Grafana"
url: "https://aws.amazon.com/blogs/mt/monitor-aws-transit-gateway-flow-logs-centrally-using-amazon-managed-grafana/"
date: "Tue, 01 Apr 2025 13:13:18 +0000"
author: "Harsh Chheda"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-grafana/feed/"
---
<p>As organizations continue to expand their cloud infrastructure by connecting multiple <a href="https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html">Amazon Virtual Private Clouds</a>&nbsp;(Amazon VPC) across accounts and regions, the complexity of managing their network environment increases. <a href="https://aws.amazon.com/transit-gateway/">AWS Transit Gateway</a> has emerged as a powerful solution to simplify this complexity by providing a centralized hub for secure communication between Amazon VPCs, on-premises systems, and other transit gateways.</p> 
<p><a href="https://docs.aws.amazon.com/vpc/latest/tgw/tgw-flow-logs.html">Amazon VPC Transit Gateway Flow Logs</a> enables you to gain visibility and insights into network traffic going through your transit gateways. These logs capture detailed information on the transit gateways, such as source/destination IPs, ports, protocol, traffic counters, timestamps, and other metadata. Flow logs can help you with your use-cases such as network troubleshooting, network capacity planning and compliance and security. You can leverage <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a> to visualize and monitor the Transit Gateway Flow Logs, unlocking a wealth of operational benefits. This centralized monitoring can empower organizations to analyze network performance and ensure compliance. By analyzing traffic patterns, teams can proactively identify anomalies, plan capacity, and detect suspicious activities. This comprehensive visibility allows organizations to troubleshoot issues, scale infrastructure, and maintain the overall health of their distributed cloud environments.</p> 
<p>In this post, we will set up an Amazon Managed Grafana dashboard to visualize and centrally monitor Transit Gateway Flow Logs stored in an <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service</a> (Amazon S3) bucket, leveraging <a href="https://aws.amazon.com/glue/">AWS Glue</a> and <a href="https://aws.amazon.com/athena/">Amazon Athena</a> for data cataloging and querying respectively.</p> 
<h2>Architecture Overview</h2> 
<p>The following architecture diagram illustrates the delivery of flow logs generated traffic from multiple Amazon VPCs, traversing through your Transit Gateways and stored into a centralized Amazon S3 bucket. <a href="https://aws.amazon.com/glue/">AWS Glue</a> accesses this S3 bucket and crawls the logs data using <a href="https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html">AWS Glue crawler</a> to create table definitions in <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/serverless-etl-aws-glue/aws-glue-data-catalog.html">AWS Glue Data Catalog</a>. Next, Amazon Athena is used to create a tabular view for an effective data cataloging and querying. Finally, we leverage&nbsp;<a href="https://docs.aws.amazon.com/athena/latest/ug/work-with-data-stores.html">Amazon Managed Grafana Athena data source</a> to create dashboards and visualize the <a href="https://docs.aws.amazon.com/vpc/latest/tgw/tgw-flow-logs.html">AWS Transit Gateway Flow Logs</a>.</p> 
<p><img alt="Diagram illustrating VPCs with EC2 instances using Transit Gateway, generating flow logs to S3. Athena and Glue process data for visualization on Amazon Managed Grafana dashboard." class="alignnone size-full wp-image-59025" height="1008" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/10/Screenshot-2025-01-10-at-11.52.24 AM.png" width="2416" /><em>Figure 1 – Architecture overview</em></p> 
<p><strong><u>Prerequisites</u></strong></p> 
<ol> 
 <li> 
  <ol> 
   <li>Existing AWS Transit Gateways. If you don’t have a Transit Gateway set up in your account, please refer to the AWS <a href="https://docs.aws.amazon.com/vpc/latest/tgw/tgw-getting-started.html">documentation</a> to create one.</li> 
   <li>Create <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html">Amazon S3 bucket</a> for storing Transit Gateway Flow Logs.</li> 
   <li>Set up <a href="https://docs.aws.amazon.com/athena/latest/ug/workgroups-manage-queries-control-costs.html">Athena workgroups</a> with <a href="https://docs.aws.amazon.com/grafana/latest/userguide/Athena-prereq.html">Amazon Managed Grafana prerequisites</a>.</li> 
   <li>Configure Amazon Managed Grafana workspace by following the steps in the <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-create-workspace.html">Creating&nbsp;a workspace</a> This will help you set up the workspace and assign user as administrator, enabling them to access the Grafana dashboard using the workspace URL with full administrative capabilities. 
    <ol> 
     <li>In this post, we’re using the <a href="https://aws.amazon.com/single-sign-on/">AWS IAM Identity Center</a> option with Amazon Managed Grafana. To set up Authentication and Authorization, follow the instructions in the <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-manage-users-and-groups-AMG.html">Amazon Managed Grafana User Guide</a> to enable AWS IAM Identity Center.</li> 
     <li>To use AWS data source configuration, first use the Amazon Managed Grafana console to enable service-managed AWS <a href="https://aws.amazon.com/iam/">Identity and Access Management (IAM)</a> roles that grants the workspace with AWS IAM policies necessary to access resources in your AWS Account or AWS Organization. Then, use the Amazon Managed Grafana workspace console to add Amazon<a href="https://docs.aws.amazon.com/grafana/latest/userguide/AWS-Athena.html"> Athena data source</a>.<br /> <strong>Security/IAM Note:</strong> Configure IAM permissions following the Principle of Least Privilege (PoLP) as this setup is for demonstration purposes only. Refer to <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html">Security best practices in IAM</a></li> 
    </ol> </li> 
  </ol> 
  <ol start="5"> 
   <li>The Amazon S3 permissions for accessing the underlying data source of an Athena query are not included in this managed policy. You must add the necessary permissions for the Amazon S3 buckets manually, on a case-by-case basis.&nbsp;Refer to <a href="https://docs.aws.amazon.com/grafana/latest/userguide/Athena-prereq.html">Athena prerequisites documentation</a>.</li> 
  </ol> </li> 
</ol> 
<h2>Step 1: Launch the AWS CloudFormation template</h2> 
<p>We are using AWS CloudFormation (CFN) template to dynamically build the infrastructure, which will create:</p> 
<ul> 
 <li>An Amazon S3 bucket to store Transit Gateway Flow Logs in the primary AWS Account</li> 
 <li>AWS Glue crawler and database configuration</li> 
 <li>Amazon Athena workgroup setup</li> 
 <li>Athena view deployment as a Named Query</li> 
</ul> 
<p><u>Note</u>: Some of the resources that this stack deploys incur costs when in use.</p> 
<p>After you have confirmed that you meet all prerequisites, deploy the CloudFormation template: <a href="https://d2908q01vomqb2.cloudfront.net/artifacts/MTBlog/cloudops-1228/CFN-TGW-AMG.yaml">BlogCFN.Yaml</a></p> 
<ul> 
 <li>Refer to&nbsp;<a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html">Creating a stack on the AWS CloudFormation console</a>.</li> 
</ul> 
<h2>Step 2: Configure AWS Transit Gateway Flow Logs and store them to Amazon S3 bucket</h2> 
<p>Flow logs can publish the logs data to Amazon S3 using AWS Management Console or AWS Command Line Interface (AWS CLI). These can be published to an existing Amazon S3 bucket that you specify.</p> 
<ol> 
 <li>Launch the <a href="https://console.aws.amazon.com/vpc/">Amazon VPC console</a>.</li> 
 <li>From the navigation pane choose <strong>Transit gateways</strong> or <strong>Transit gateway attachments</strong>.</li> 
 <li>Choose the checkbox for one or more transit gateways or transit gateway attachments.</li> 
 <li>Choose <strong>Actions</strong> &gt; <strong>Create flow log</strong>. 
  <ol> 
   <li>For <strong>Destination</strong>, choose <strong>Send to an S3 bucket</strong>.</li> 
   <li>For <strong>S3 bucket ARN</strong>, you can either use the automatically created S3 bucket (created by AWS CloudFormation template in the step above) or specify the Amazon Resource Name (ARN) of an existing Amazon S3 bucket. When configuring the flow logs, you can optionally specify a subfolder within the S3 bucket, like “my-bucket/my-logs/”, with the S3 bucket ARN. Note that “AWSLogs” cannot be used as a subfolder name, as it is a reserved term. If you own the bucket, AWS automatically creates a resource policy and attaches it to the bucket. For more information, see <a href="https://docs.aws.amazon.com/vpc/latest/tgw/flow-logs-s3.html#flow-logs-s3-permissions">Amazon S3 bucket permissions for flow logs</a>.</li> 
   <li>For <strong>Log record format</strong>, specify the format for the flow log record. 
    <ol> 
     <li>To use the default flow log record format, choose <strong>AWS default format</strong>.</li> 
     <li>To create a custom format, choose <strong>Custom format</strong>. For <strong>Log format</strong>, choose the fields to include in the flow log record.</li> 
    </ol> </li> 
   <li>For <strong>Log file format</strong>, specify the format for the log file. 
    <ol> 
     <li> 
      <ol> 
       <li><strong>Text</strong> – Plain text. This is the default format.</li> 
       <li><strong>Parquet</strong> – Apache Parquet is a columnar data format. Queries on data in Parquet format are 10 to 100 times faster compared to queries on data in plain text. Data in Parquet format with Gzip compression takes 20 percent less storage space than plain text with Gzip compression.5. (Optional) To use Hive-compatible S3 prefixes, choose <strong>Hive-compatible S3 prefix</strong>, <strong>Enable</strong>.</li> 
      </ol> </li> 
    </ol> </li> 
   <li>(Optional) To partition your flow logs per hour, choose <strong>Every 1 hour (60 mins)</strong>.</li> 
   <li>(Optional) To add a tag to the flow log, choose <strong>Add new tag</strong> and specify the tag key and value.</li> 
   <li>Choose <strong>Create flow log</strong>.</li> 
  </ol> </li> 
</ol> 
<ol> 
 <li></li> 
</ol> 
<ol> 
 <li> 
  <ol> 
   <li></li> 
  </ol> </li> 
</ol> 
<p><img alt="Includes steps to configure the S3 destination for the logs." class="alignnone wp-image-59261" height="586" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/30/TGW-Flowlog1.jpg" width="1055" /></p> 
<p><em><img alt="Includes steps to configure the S3 destination for the logs." class="alignnone wp-image-59262" height="605" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/30/TGW-Flowlog2.png" width="1028" /><br /> Figure 2 – Create AWS Transit Gateway Flow logs</em></p> 
<p>The IP traffic going to and from the AWS Transit Gateway is captured and stored in the S3 bucket specified when creating the flow log. Just for this blog post, we have configured AWS Glue crawler schedule as one hour. You can modify this schedule based on your requirements by following the AWS Glue documentation on&nbsp;<a href="https://docs.aws.amazon.com/glue/latest/dg/Update-crawler-schedule.html">updating crawler schedules</a>.</p> 
<p>Once the flow log file is generated and stored in S3 bucket, AWS Glue crawler will scan and catalog the data from the bucket and automatically create or update metadata in the AWS Glue database and tables.</p> 
<h2>Step 3: Create an Amazon Athena view using the saved queries created as part of the AWS CloudFormation stack</h2> 
<ol> 
 <li>Go to Amazon Athena &gt; Query editor &gt; Saved queries tab and choose the query named “<strong>aws_tgw_centralized_logging</strong>”.</li> 
</ol> 
<p><u>Note</u>: Workgroup created is named “<strong>tgw-logs-athena”</strong></p> 
<p><img alt="Create an Amazon Athena view using the saved queries created as part of the AWS CloudFormation stack for Flow and Alert logs" class="alignnone size-full wp-image-59274" height="380" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/30/Screenshot-2025-01-31-at-3.53.21 AM.png" width="1906" /></p> 
<p><em>Figure 3 – Athena saved query</em></p> 
<ol start="2"> 
 <li>On the Query editor, verify the Data source, Database and Table names while running the query. Upon successful execution, the query creates a view named “<strong>tgwlogs</strong>”.</li> 
</ol> 
<p><img alt="Execute the Create Athena View SQL query" class="alignnone size-full wp-image-59270" height="652" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/30/Screenshot-2025-01-31-at-3.45.47 AM.png" width="1505" /></p> 
<p><em>Figure 4 – Run Athena saved query</em></p> 
<h2>Step 4: Configure Amazon Athena data source in Amazon Managed Grafana</h2> 
<ol> 
 <li>After creating the Amazon Managed Grafana workspace and making the user as admin as mentioned in the pre-requisite. Login into the Amazon Managed Grafana dashboard using the workspace URL.</li> 
 <li>Navigate to Data sources and select Amazon Athena from the options.</li> 
 <li>Adjust the Amazon Athena settings by specifying the Default Region, Data source, Database, Workgroup and set the Amazon S3 Output Location for your Amazon Athena query.</li> 
 <li>Choose “Save &amp; test” to confirm that the data source is functioning properly. You can now begin querying and visualizing metrics from the AWS environment.</li> 
</ol> 
<p><img alt="Add Athena as data source for Amazon Managed Grafana dashboard" class="alignnone wp-image-59058" height="807" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/11/Screenshot-2025-01-11-at-9.22.35 PM.png" width="799" /><br /> <em>Figure 5 – Add Athena as data source</em></p> 
<h2>Step 5: Create Amazon Managed Grafana dashboard using Athena as data source</h2> 
<p>Amazon Managed Grafana is a fully managed service that makes it easy to create, configure, and share interactive dashboards and charts for monitoring your data. You can also use Amazon Grafana to set up alerts and notifications based on specific conditions or thresholds, allowing you to quickly identify and respond to issues.</p> 
<p>In this step, we will use Amazon Managed Grafana to create a near real-time dashboard to visualize your AWS Transit Gateway Flow Logs.</p> 
<ol> 
 <li>You can either <a href="https://docs.aws.amazon.com/grafana/latest/userguide/v9-dash-creating.html">create a new Amazon Managed Grafana dashboard</a> or <a href="https://docs.aws.amazon.com/grafana/latest/userguide/dashboard-export-and-import.html">import one using JSON</a> to visualize your transit gateway flow logs.</li> 
 <li>Download the <a href="https://github.com/aws-samples/aws-CO-BlogPosts/blob/main/TGW-AMG-Dashboard/AWS_TGW_AMG_Dashboard.json">sample dashboard JSON file</a> that you can import to visualize various metrics and build upon this template.</li> 
</ol> 
<p><img alt="Demonstrates the how to import the sample Grafana dashboard template" class="alignnone wp-image-59060" height="709" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/11/Screenshot-2025-01-11-at-9.25.11 PM.png" width="800" /><br /> <em>Figure 6 – Import Amazon Managed Grafana dashboard template</em></p> 
<ol start="3"> 
 <li>Once the sample JSON is loaded successfully, your Amazon Managed Grafana dashboard for the Transit Gateway Flow Logs will provide:<br /> <strong><u>Comprehensive data insights: </u></strong>Track the total bytes exchanged between source and destination addresses for a clear overview of data transfers.<br /> <strong><u>Strategic prioritization</u></strong>: Identify the top source and destination addresses based on packet counts, enabling efficient prioritization of network analysis.<br /> <strong><u>Network optimization</u>:</strong> Gain valuable insights by visualizing the top three source and destination subnets or Amazon VPCs according to bytes transferred, aiding in optimizing network performance.<br /> <strong><u>Granular trend analysis</u>:</strong> Utilize Amazon Managed Grafana to analyze byte and packet flow trends, both inbound and outbound, within specific Regions and time ranges using selected transit gateways and their attachment IDs.<br /> <strong><u>Proactive issue detection</u></strong>: Stay ahead by detecting packet drops due to routing issues or black holes. Monitor and identify these incidents within chosen Regions and time frames for prompt action.</li> 
</ol> 
<p><img alt="Demonstrates the graphs and visuals that are created post importing the JSON Grafana dashboard template to analyze the flow and alert firewall logs" class="alignnone wp-image-59059" height="772" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/01/11/Screenshot-2025-01-11-at-9.23.33 PM.png" width="876" /><br /> <em>Figure 7 – Visualize AWS Transit Gateway flow logs on Amazon Managed Grafana Dashboard</em></p> 
<p>Now we have the AWS Transit Gateway Insights on Amazon Managed Grafana. This dashboard refreshes every five minutes and runs a query against the materialized views that we previously created in Amazon Athena. Finally, Amazon Managed Grafana alerting provides us with robust and actionable alerts that help us learn about problems in the system moments after they occur. To learn more about Amazon Managed Grafana alerting visit “<a href="https://docs.aws.amazon.com/grafana/latest/userguide/v9-alerts.html">Alerts in Grafana</a>”.</p> 
<h2><strong>Clean up</strong></h2> 
<p>To avoid ongoing charges in your AWS account, you should delete the AWS resources listed in the prerequisites section of this post. Furthermore, log in to the <a href="https://console.aws.amazon.com/console/home">AWS Management Console</a> and delete any manually created resources.</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html">Delete</a> AWS CloudFormation Stack.</li> 
 <li><a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-edit-delete-workspace.html">Delete</a> Amazon Managed Grafana workspace.</li> 
 <li><a href="https://docs.aws.amazon.com/athena/latest/APIReference/API_DeleteWorkGroup.html">Delete</a> Amazon Athena workgroup.</li> 
</ol> 
<p><u>Note</u>: You can delete only the empty S3 buckets using AWS CloudFormation. Delete CloudFormation stack fails in case there is content in S3 bucket. Empty the S3 bucket before initiating delete process for the CloudFormation template.</p> 
<h2><strong>Conclusion</strong></h2> 
<p>This blog post demonstrated how to create visualizations using Amazon Managed Grafana dashboards for your AWS Transit Gateway Flow Logs. The ability to visualize metrics data helps save time through proactive capacity planning and trend identification, which leads to infrastructure cost savings. Additionally, visualization on Amazon Managed Grafana dashboard helps identify anomalies in source-destination traffic and enables prompt troubleshooting steps to minimize resolution time.</p> 
<p>You can get hands-on experience with exploring <a href="https://catalog.workshops.aws/observability/en-US">&nbsp;One Observability Workshop</a>. Visit the <a href="https://aws-observability.github.io/observability-best-practices/guides/cost/cost-visualization/cost/">AWS Observability guide</a> to learn more about best practices. To get started and learn more, visit <a href="https://docs.aws.amazon.com/vpc/latest/tgw/tgw-flow-logs.html">Amazon VPC Transit Gateways Flow Logs</a> and <a href="https://docs.aws.amazon.com/grafana/latest/userguide/dashboard-overview.html">Amazon Managed Grafana Dashboards</a>.</p>
