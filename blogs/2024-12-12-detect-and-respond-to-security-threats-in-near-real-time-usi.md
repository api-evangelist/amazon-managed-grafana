---
title: "Detect and respond to security threats in near real-time using Amazon Managed Grafana"
url: "https://aws.amazon.com/blogs/mt/detect-and-respond-to-security-threats-in-near-real-time-using-amazon-managed-grafana/"
date: "Thu, 12 Dec 2024 15:26:09 +0000"
author: "Sameeksha Garg"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-grafana/feed/"
---
<p>Security is “job zero” at AWS. It’s crucial to gain deeper insights into your AWS infrastructure’s security posture to respond quickly to threats. The ability to centrally monitor and visualize the security findings make it easier for you to identify any security threats or gaps and also keep the principle of least privilege in focus. You can visualize the insights into those security findings across multiple accounts using <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a> that retrieves and refreshes the data periodically.</p> 
<p><a href="https://aws.amazon.com/security-hub/">AWS Security Hub</a> offers a comprehensive overview of your AWS security status, allowing you to compare it with industry standards and best practices. Organizations may have additional requirements to centralize AWS Security Hub findings and integrate them with other operational data. Some of the benefits include consolidating AWS Security Hub findings across regions into a single dashboard view, and centralizing and correlating various security &amp; compliance data along with operational data into one dashboard.</p> 
<p>In this post we will show you how you can centralize your <a href="https://aws.amazon.com/security-hub/">AWS Security Hub</a> findings, that provide a single-pane-of-glass on workloads running on AWS cloud using <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a>.</p> 
<p>We will demonstrate similar integration with <a href="https://aws.amazon.com/guardduty/">Amazon GuardDuty</a> and <a href="https://aws.amazon.com/inspector/">Amazon Inspector</a> through another post.</p> 
<h2><strong>Architecture Overview:</strong><br /> <img alt="" class="aligncenter wp-image-57301 size-full" height="636" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/10/22/Figure1.png" width="956" /></h2> 
<p style="text-align: center;">Figure 1: Architecture Overview</p> 
<p>Please refer to the details on components of the solution depicted in the above architecture:</p> 
<ol> 
 <li>AWS Security Hub – AWS Security Hub is a cloud security posture management service that performs security best practices checks, aggregates alerts, and enables automated remediation.</li> 
 <li><a href="https://aws.amazon.com/eventbridge/">Amazon EventBridge Rule</a> – Amazon EventBridge rule will be triggered on a new finding or an update to existing finding in AWS Security Hub.</li> 
 <li>Custom <a href="https://aws.amazon.com/pm/lambda/">AWS Lambda</a> to process the Event – An AWS Lambda function to extract and transform the complex nested AWS Security Hub finding to a simpler JSON format that is more suitable to perform further analytics.</li> 
 <li><a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service (Amazon S3)</a>&nbsp;bucket to receive the processed event – The AWS Lambda function delivers each finding to this Amazon S3 bucket.</li> 
 <li><a href="https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html#crawler-running">AWS Glue crawler</a> and <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/serverless-etl-aws-glue/aws-glue-data-catalog.html">AWS Glue Data Catalog</a> – Configure an AWS Glue crawler to run every hour to parse the JSON data stored in Amazon S3 and update the AWS Glue Data Catalog containing the JSON table schema, metadata, and database partitions. This metadata is required to perform queries against the data in Amazon S3.</li> 
 <li><a href="https://docs.aws.amazon.com/athena/latest/ug/what-is.html">Amazon Athena</a> Workgroup – Amazon Athena is used to look at the schema from the AWS Glue Data Catalog and query the data in S3. The Amazon Managed Grafana Dashboard must have permissions to execute queries against the Amazon S3 data using Amazon Athena Workgroup and AWS Glue Data Catalog.</li> 
 <li>Amazon Managed Grafana – Deploy a dashboard using Amazon Managed Grafana to visualize data.</li> 
</ol> 
<p>Using the <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html">AWS CloudFormation</a> template provided in this blog, deploy the required resources in the same AWS Region and AWS account containing the Security Hub service where findings of all regions and accounts are aggregated into.</p> 
<h2><strong>Prerequisites:</strong></h2> 
<ol> 
 <li>Enable <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-accounts.html">AWS Security Hub</a> in member accounts. Enable required <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/standards-reference.html">security standards</a>, AWS native service integration, and partner integrations in all the member accounts across your AWS Regions.</li> 
 <li>Set up your AWS Security Hub administrator account. Designate one of the AWS accounts within your <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html">AWS Organizations</a> to be a delegated administrator for AWS Security Hub. This account can manage and receive findings across <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs-manage_accounts_members.html">member accounts</a>.</li> 
 <li>Enable <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation.html">Cross Region Aggregation</a>.</li> 
 <li>Set up <a href="https://docs.aws.amazon.com/athena/latest/ug/creating-workgroups.html">Amazon Athena workgroups</a>.</li> 
 <li>Set up Amazon Managed Grafana workspace. For information, and steps for creating the Amazon Managed Grafana workspace, see <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-create-workspace.html">Creating a Workspace</a>. 
  <ul> 
   <li>Amazon Managed Grafana allows you to configure user access through <a href="https://aws.amazon.com/iam/identity-center/">AWS IAM Identity Center</a> or other Identity Providers (IdP) based on SAML. Review <a href="https://docs.aws.amazon.com/grafana/latest/userguide/authentication-in-AMG-SAML.html">Amazon Managed Grafana supports direct SAML integration with identity providers</a>.</li> 
   <li>In this post, you will be using the AWS IAM Identity Center option. To set up authentication and authorization, follow the instructions in the <a href="https://docs.aws.amazon.com/grafana/latest/userguide/authentication-in-AMG-SSO.html">Amazon Managed Grafana User Guide</a> to enable AWS IAM Identity Center.</li> 
   <li>To use AWS data source configuration, first use the Amazon Managed Grafana console to enable service-managed <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-manage-permissions.html">AWS Identity and Access Management (IAM)</a> roles that grants the workspace with AWS IAM policies necessary to access resources in your AWS Account/Organization. Then, use the Amazon Managed Grafana workspace console to add <a href="https://docs.aws.amazon.com/grafana/latest/userguide/Athena-adding-AWS-config.html">Amazon Athena data source</a>.</li> 
  </ul> </li> 
</ol> 
<p>Once you have all the prerequisites in place, follow the instructions below for visualizing <a href="https://aws.amazon.com/security-hub/">AWS Security Hub</a> findings on <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a>.</p> 
<h2><strong>Deployment Steps:</strong></h2> 
<h3><strong>Step 1: Launch the AWS CloudFormation template</strong></h3> 
<p>Download and launch this AWS CloudFormation template to deploy AWS Lambda, Amazon S3 Bucket, AWS Glue Crawler, AWS Glue Database and its related components.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/artifacts/MTBlog/cloudops-1319/sechub-blog-templateV2.yml"><img alt="" class="aligncenter wp-image-57306" height="38" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/10/22/Picture-1.jpg" width="200" /></a></p> 
<p>Note: Some of the resources that this stack deploys incur costs when in use.</p> 
<p>Follow these steps to generate your resources utilizing an AWS CloudFormation template:</p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/">AWS Management Console</a>.</li> 
 <li>Navigate to the <a href="https://console.aws.amazon.com/cloudformation">AWS CloudFormation console</a> &gt; <strong>Create Stack &gt; “With new resources”</strong>.</li> 
 <li>Upload the yaml template file and choose Next.</li> 
 <li>Specify a <strong>“Stack name”</strong> and choose Next.</li> 
 <li>Leave the<strong> “Configure stack options”</strong> at default values and choose Next.</li> 
 <li>Review the details on the final screen and under <strong>“Capabilities”</strong> check the box for “I acknowledge that AWS CloudFormation might create IAM resources with custom names”.</li> 
 <li>Choose Submit.</li> 
</ol> 
<p style="text-align: center;"><img alt="" class="aligncenter wp-image-58449 size-full" height="390" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/12/11/Fig2-1.png" width="1678" /><br /> Figure 2: Acknowledgement</p> 
<p>Note: You can review the progress of your new stack under AWS CloudFormation &gt; Stacks &gt; [StackName] &gt; Events tab</p> 
<p>Once the Stack is created successfully, you will see the following resources deployed: <strong>Amazon EventBridge scheduler, AWS Lambda Function, Amazon S3 Bucket, AWS Glue Crawler, AWS Glue Database</strong> and the corresponding <strong>AWS IAM</strong> Roles and Policies are created successfully.</p> 
<h3><strong>Step 2: Create View in Amazon Athena using the below queries created as part of the AWS CloudFormation stack</strong></h3> 
<ol> 
 <li>Go to Amazon Athena &gt; Query editor &gt; Saved queries tab and choose the query named “AWS-securityhub”.<br /> Note: Workgroup created is named <strong>“Primary”</strong></li> 
</ol> 
<p><img alt="" class="aligncenter wp-image-58450 size-full" height="650" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/12/11/Fig3-1.png" width="2644" /></p> 
<p style="text-align: center;">Figure 3: Amazon Athena Saved Queries</p> 
<ol start="2"> 
 <li>On the Query editor, verify the Data source, Database and Table names while running the query. Upon successful execution, the query creates a View named “security_hub_findings”.</li> 
</ol> 
<p><img alt="" class="aligncenter wp-image-58451 size-full" height="1520" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/12/11/Fig4-1.png" width="3538" /></p> 
<p style="text-align: center;">Figure 4: Amazon Athena Query Editor</p> 
<h3><strong>Step 3: Configure Amazon Athena Data Source in Amazon Managed Grafana</strong></h3> 
<ol> 
 <li>Access the Amazon Managed Grafana console through the provided Amazon Managed Grafana workspace URL and log in using the user credentials you’ve set up.</li> 
 <li>Navigate to Administration &gt; Data sources and select Amazon Athena from the options.</li> 
 <li>Adjust the Amazon Athena settings by specifying the Default Region (us-east-1), Data source (AWSDataCatalog), Database (aws-security-hub-db), Workgroup (primary), and set the Output Location for your Amazon Athena query.</li> 
 <li>Choose “Save &amp; test” to confirm that the data source is functioning properly. You can now begin querying and visualizing metrics from the AWS environment.<br /> Note: If you encounter a permission denied error, ensure that the Amazon Managed Grafana service role permissions, as discussed in the previous step, are correctly configured.</li> 
</ol> 
<p><img alt="" class="aligncenter wp-image-58452 size-full" height="1736" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/12/11/Fig5-1.png" width="1524" /></p> 
<p style="text-align: center;">Figure 5: Amazon Athena as Data source</p> 
<h3><strong>Step 4: Create an Amazon Managed Grafana Dashboard</strong></h3> 
<p>Amazon Managed Grafana is a fully managed service designed to simplify the process of creating, configuring, and sharing interactive dashboards and charts for monitoring your data. It offers the ability to establish alerts and notifications based on specific conditions or thresholds, enabling swift identification and response to issues.</p> 
<p>In this next step, we will utilize Amazon Managed Grafana to generate a new AWS Security Hub findings detection dashboard.</p> 
<ol> 
 <li>Retrieve the AWS Security Hub findings dashboard JSON file from this <a href="https://d2908q01vomqb2.cloudfront.net/artifacts/MTBlog/cloudops-1319/SecurityHub-v3.json">link</a>.</li> 
 <li>Import the dashboard by navigating to Dashboards &gt; New and selecting Import in the Amazon Managed Grafana console. For additional information on exporting and importing dashboards, refer to the <a href="https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/import-dashboards/">documentation</a>.</li> 
</ol> 
<p><img alt="" class="aligncenter wp-image-58455 size-full" height="466" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/12/11/Fig8.png" width="904" /></p> 
<p style="text-align: center;">Figure 6: Amazon Managed Grafana Dashboard</p> 
<p>Finally, AWS Security Hub findings are integrated into Amazon Managed Grafana. This dashboard updates every 5 minutes, querying the materialized views established in Amazon Athena.</p> 
<p>Furthermore, Amazon Managed Grafana’s alerting system delivers strong and actionable alerts, enabling us to swiftly identify system issues as soon as they arise. For further insights into Amazon Managed Grafana alerting, please visit the “<a href="https://docs.aws.amazon.com/grafana/latest/userguide/alerts-overview.html">Alerts in Amazon Managed Grafana</a>” section.</p> 
<h2><strong>Clean up</strong></h2> 
<p>To avoid incurring future charges, delete all resources used in this post.</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html">Empty Amazon S3 bucket before deleting the AWS CloudFormation stack</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html">Delete AWS CloudFormation Stack</a></li> 
 <li><a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-edit-delete-workspace.html">Delete Amazon Managed Grafana Workspace</a></li> 
 <li><a href="https://docs.aws.amazon.com/athena/latest/ug/workgroups-create-update-delete.html">Delete Amazon Athena workgroup</a></li> 
</ul> 
<p>In this blog post, we showed how you can visualize and analyze your AWS Security Hub with Amazon Managed Grafana. By identifying potential threats quickly, sensitive data can be safeguarded more effectively. Near real-time dashboards allow for proactive measures, ensuring that critical workload remains secure.</p> 
<p>To learn more and get hands-on experience on AWS observability services, check the <a href="https://workshops.aws/categories/Observability">One Observability Workshop</a>.</p>
