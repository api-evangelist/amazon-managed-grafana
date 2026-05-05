---
title: "Gain visibility of AWS backup activities using Amazon Managed Grafana"
url: "https://aws.amazon.com/blogs/mt/gain-visibility-of-aws-backup-activities-using-amazon-managed-grafana/"
date: "Mon, 11 Aug 2025 18:00:01 +0000"
author: "Anjali Sharma"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-grafana/feed/"
---
<p><a href="https://aws.amazon.com/backup/">AWS Backup</a> is a comprehensive service that simplifies the process of centralizing and automating data protection across various AWS services, both in the cloud and on-premises, all managed seamlessly. Organizations have different requirements and want to track their backup, copy and restore activities across AWS cloud resources. Currently, in order to view status of resource backup operations, it requires you to have AWS Identity and Access Management (IAM) user access privileges on the AWS Management Console. The ability to centrally monitor backup activities and generate insights makes it easier for you to identify any failures while still maintaining the principle of least privilege You can visualize backup activities across multiple accounts using&nbsp;<a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a>&nbsp;that retrieves and refreshes the data periodically.</p> 
<p>This post explains how <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a> dashboards help you to visualize and track <a href="https://aws.amazon.com/backup/">AWS Backup Metrics</a>.</p> 
<h2><strong>Architecture Overview</strong></h2> 
<p>The following architecture diagram showcases <a href="https://aws.amazon.com/backup/">AWS Backup</a> Job events that are triggered to <a href="https://aws.amazon.com/eventbridge/">Amazon Event Bridge</a> in real-time. This functionality enables the collection of AWS Backup Job statuses, which are then saved in Amazon Simple Storage Service (<a href="https://aws.amazon.com/s3/">Amazon S3</a>) buckets using <a href="https://aws.amazon.com/kinesis/">Amazon Kinesis</a>. Subsequently, <a href="https://docs.aws.amazon.com/athena/latest/ug/what-is.html">AWS Glue</a> retrieves this data from an Amazon S3 bucket, extracts the metadata, and establishes table definitions within the AWS Glue Data Catalog. After the data is stored in the database. <a href="https://docs.aws.amazon.com/athena/latest/ug/what-is.html">Amazon Athena</a> generates a structured view of the information. Finally, the <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a> team utilizes the Athena data source to create a comprehensive <a href="https://aws.amazon.com/backup/">AWS Backup</a> report dashboard on Amazon Managed Grafana.</p> 
<p><img class="aligncenter" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-1-2.png" /></p> 
<p style="text-align: center;">Figure 1. Architecture Diagram</p> 
<h2><strong>Prerequisites:</strong></h2> 
<ol> 
 <li>Enable and configure AWS Backup Service.</li> 
 <li>Configure <a href="https://docs.aws.amazon.com/aws-backup/latest/devguide/manage-cross-account.html">cross-account managemen</a>t feature in AWS Backup to manage and monitor your backup, restore, and copy jobs across AWS Regions and AWS accounts that you configure with AWS <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html">Organizations</a>.</li> 
 <li>Set up <a href="https://docs.aws.amazon.com/athena/latest/ug/workgroups-procedure.html">Amazon Athena workgroups</a>.</li> 
 <li>Set up Amazon Managed Grafana workspace. For information, and steps for creating the Amazon Managed Grafana workspace, see <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-create-workspace.html">Creating a Workspace</a>.</li> 
</ol> 
<ul> 
 <li>For user authentication and authorization, Amazon Managed Grafana can integrate with identity providers (IdPs) that support SAML 2.0 and also can integrate with <a href="https://aws.amazon.com/iam/identity-center/">AWS IAM Identity Center</a>. Review the <a href="https://aws.amazon.com/blogs/mt/amazon-managed-grafana-supports-direct-saml-integration-with-identity-providers/">Amazon Managed Grafana supports direct SAML integration with identity providers</a>.</li> 
 <li>To use AWS data source configuration, first use the Amazon Managed Grafana console to enable service-managed Identity and Access Management (IAM) roles that grant the workspace with IAM policies necessary to read resources in your account/Organization. Then, use the Amazon Managed Grafana workspace console to add <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AWS-Athena.html">Athena Data Source</a>.</li> 
</ul> 
<h3><strong>Step 1: Launch the AWS CloudFormation template</strong></h3> 
<p>Download and launch the following AWS CloudFormation template to deploy Kinesis Firehose, Glue Crawler, Glue Database and its related components.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/artifacts/MTBlog/cloudops-1494/cfn_backupblog_v2.1.yaml"><strong>Solution file</strong></a></p> 
<p><strong>Note:</strong> Some of the resources that this stack deploys incur costs when in use.</p> 
<p>To create your resources using AWS CloudFormation template, complete the following steps:</p> 
<ul> 
 <li>Sign in to the <a href="https://aws.amazon.com/console/">AWS Management Console</a></li> 
 <li>Navigate to the <a href="https://ap-southeast-2.signin.aws.amazon.com/oauth?client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcloudformation&amp;code_challenge=ihKN-x6Y9QR1w8GYlE0LmMenvM9PW_VUV0FXdEJOlMA&amp;code_challenge_method=SHA-256&amp;response_type=code&amp;redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fcloudformation%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26oauthStart%3D1722231736949%26state%3DhashArgsFromTB_ap-southeast-2_cb8c2de1b76d9e68">AWS CloudFormation console</a> &gt; <strong>Create Stack</strong> &gt; <strong>“With new resources”</strong></li> 
 <li>Specify a <strong>“Stack name”</strong> and choose Next</li> 
 <li>Leave the<strong> “Configure stack options”</strong> at default values and choose Next</li> 
 <li>Review the details on the final screen and under <strong>“Capabilities”</strong> check the box for “I acknowledge that AWS CloudFormation might create IAM resources with custom names”</li> 
 <li>Choose Submit</li> 
</ul> 
<p><img class="aligncenter" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-1-2.jpg" /></p> 
<p style="text-align: center;">Figure 2. Acknowledgement</p> 
<p><strong>Note:</strong> You can review the progress of your new stack under AWS CloudFormation &gt; Stacks &gt; [StackName] &gt; Events tab</p> 
<p>Once the Stack is created successfully, you will see the following resources deployed:<br /> <strong>Amazon EventBridge Scheduler, AWS Kinesis Firehose Delivery Stream, Amazon S3 Bucket, AWS Glue Crawler, AWS Glue Database</strong> and the corresponding <strong>AWS IAM Roles</strong> and Policies are created successfully.</p> 
<h3><strong>Step 2: Create View in Amazon Athena using the below queries:</strong></h3> 
<ul> 
 <li>Go to Amazon Athena &gt; Query editor &gt; Saved queries tab and choose the query name <strong>“AWS-Backup-Events”</strong></li> 
</ul> 
<p><img class="aligncenter" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-2-3.png" /></p> 
<p style="text-align: center;">Figure 3. Saved Athena Query</p> 
<ul> 
 <li>On the Query editor, verify the Data source, Database and Table names and replace &lt;account-id&gt; with your account id while running the query. Upon successful execution, the query creates a View named “grafana_view”.<br /> <strong>Note: backupsizeinbytes</strong> attribute would only be available for tasks with the status COMPLETED or RUNNING.</li> 
</ul> 
<p><img class="aligncenter" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-1-3.png" /></p> 
<p style="text-align: center;">Figure 4. Run Athena Query in Query Editor</p> 
<h3 style="text-align: left;"><strong>Step 3: Configure Amazon Athena Data Source in Amazon Managed Grafana</strong></h3> 
<ul> 
 <li>Launch the Amazon Managed Grafana console using the Grafana workspace URL and login using the user credentials you configured</li> 
 <li>Under Administration &gt; Data sources &gt; choose Amazon Athena</li> 
 <li>Configure the Amazon Athena settings by choosing Default Region (us-east-1), Data source (AWSDataCatalog), Database (aws-backup-event-records), Workgroup (primary) and the Output Location of your Athena query</li> 
 <li>Choose Save &amp; test to verify that the data source is working. Start querying and visualizing the metrics from the AWS environment</li> 
</ul> 
<p><strong>Note:</strong> In case you receive a permission denied error, verify the Grafana service role permissions.</p> 
<p><img class="aligncenter" height="503" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-1-4.png" width="874" /></p> 
<p style="text-align: center;">Figure 5. Amazon Athena Datasource Configuration</p> 
<p><img alt="" class=" wp-image-55506 aligncenter" height="657" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/08/26/Screenshot-2024-08-26-at-7.09.53 PM-1024x769.png" width="875" /></p> 
<p style="text-align: center;">Figure 6. Load the JSON Code</p> 
<p style="text-align: center;"><img class="aligncenter" height="613" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-6-1.png" width="875" /></p> 
<p style="text-align: center;">Figure 7. Import the dashboard using JSON Code</p> 
<h3><strong>Step 4: Create an Amazon Managed Grafana Dashboard</strong></h3> 
<p>Amazon Managed Grafana is a fully managed service designed to simplify the process of creating, configuring, and sharing interactive dashboards and charts for monitoring your data. It offers the ability to establish alerts and notifications based on specific conditions or thresholds, enabling swift identification and response to issues.<br /> In this next step, we will utilize Amazon Managed Grafana to generate a new AWS Backup dashboard.</p> 
<ul> 
 <li>Retrieve the AWS Backup dashboard JSON file from this <a href="https://d2908q01vomqb2.cloudfront.net/artifacts/MTBlog/cloudops-1494/Grafana.json" rel="noopener" target="_blank">link</a>.</li> 
 <li>Import the dashboard by navigating to Dashboards &gt; New and selecting Import in the Amazon Managed Grafana console. For additional information on exporting and importing dashboards, refer to the documentation.<br /> Note: You have the option to upload a dashboard JSON file, paste a dashboard URL, or directly input dashboard JSON text into the designated text area.</li> 
</ul> 
<p><img class="aligncenter" height="333" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-7-1.png" width="875" /></p> 
<p style="text-align: center;">Figure 8. Amazon Managed Grafana Dashboard</p> 
<p><img class="aligncenter" height="369" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/07/29/Picture-8-1.png" width="874" /></p> 
<p style="text-align: center;">Figure 9. Amazon Managed Grafana Dashboard</p> 
<p>Finally, AWS Backup Report is integrated into Amazon Managed Grafana. This centralized backup console offers a consolidated view of your backups and backup activity logs, making it easier to audit your backups and ensure compliance. Furthermore, Amazon Managed Grafana’s alerting system delivers actionable alerts, enabling us to swiftly identify system issues near real time For further insights into Amazon Managed Grafana alerting, please visit the “<a href="https://docs.aws.amazon.com/grafana/latest/userguide/alerts-overview.html">Alerts in Grafana</a>” section.</p> 
<p><strong>Note:</strong> For Cross Account or Cross Region Monitoring, by default you will not get the event in the Management account, you can only see the backup jobs that you create. In order for cross account/region you need to push the event to the target bus (Management Account). Refer <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-cross-account.html">Sending and receiving Amazon EventBridge events between AWS accounts</a>.</p> 
<h2><strong>Clean up</strong></h2> 
<p>You will continue to incur cost until you clean up the infrastructure that you created for this post:</p> 
<ul> 
 <li>Delete <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html">AWS CloudFormation Stack</a><br /> <strong>Note:</strong> You can delete only the empty S3 buckets using AWS CloudFormation. Delete CloudFormation stack fails in case there is content in S3 bucket. Empty the S3 bucket before initiating delete process for the CloudFormation template.</li> 
 <li>Delete <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-edit-delete-workspace.html">Amazon Managed Grafana Workspace</a></li> 
 <li>Delete <a href="https://docs.aws.amazon.com/athena/latest/APIReference/API_DeleteWorkGroup.html">Amazon Athena workgroup</a></li> 
</ul> 
<h2><strong>Conclusion</strong></h2> 
<p>In this blog post, we introduced the AWS Backup monitoring solution using Amazon Managed Grafana. You can obtain enriched cross-account, multi-Region daily reports on your AWS Backup activities, and visualize the data using Amazon Managed Grafana dashboards. The aggregated reports and visualization dashboards can help you quickly identify and report on items and trends related to your data protection activities across your AWS accounts. You can customize the sample CloudFormation templates provided in this blog to meet your organization’s monitoring requirements, and gain the insights and visibility into your AWS Backup operations as needed.</p> 
<p>To get started and learn more, visit <a href="https://docs.aws.amazon.com/aws-backup/latest/devguide/getting-started.html">Getting started with AWS Backup</a> and <a href="https://docs.aws.amazon.com/grafana/latest/userguide/dashboard-overview.html">Amazon Managed Grafana Dashboards</a>. You can get hands-on experience with the <a href="https://catalog.workshops.aws/observability/en-US">AWS Observability services at One Observability Workshop</a>. Visit the <a href="https://aws-observability.github.io/observability-best-practices/guides/cost/cost-visualization/cost/">AWS Observability guide</a> to learn more about best practices.</p>
