---
title: "Visualizing Amazon DynamoDB data with Amazon OpenSearch Service and Amazon Managed Grafana"
url: "https://aws.amazon.com/blogs/mt/visualizing-amazon-dynamodb-data-with-amazon-opensearch-service-and-amazon-managed-grafana/"
date: "Fri, 30 May 2025 19:12:01 +0000"
author: "Arun Chandapillai"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-grafana/feed/"
---
<p>High-performance applications with unlimited throughput capabilities pose significant monitoring challenges, especially when tracking real-time metrics, utilization, and throttling events across distributed database workloads. Near real-time visibility into metrics is crucial for application performance and cost optimization.</p> 
<p>AWS allows you to seamlessly integrate multiple services to tackle these operational complexities. With <a href="https://aws.amazon.com/dynamodb/">Amazon DynamoDB</a>, you can build applications that require fast local reads and write performances. With <a href="https://aws.amazon.com/opensearch-service/">Amazon OpenSearch Service</a>, you can have real-time search, monitoring, and analysis of business and operational data. With <a href="https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/OpenSearchIngestionForDynamoDB.html">Amazon DynamoDB zero-ETL integration with Amazon OpenSearch Service</a>, you can synchronize the data between Amazon DynamoDB and Amazon OpenSearch Service. The integration is implemented through the DynamoDB plugin for OpenSearch Ingestion, providing a fully managed, no-code experience for data ingestion. This enables you to use the search features of Amazon OpenSearch Service against your data in Amazon DynamoDB. With <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a>, you can add Amazon OpenSearch Service as a data source to query and correlate metrics and logs, then view and analyze all of that data in a single visualization or dashboard.</p> 
<p>Building on the above capabilities, this blog post demonstrates how to create a pipeline connecting DynamoDB storage, OpenSearch analytics, and Grafana visualization. You will learn how to gain insights from your data more quickly and efficiently without complex ETL processes. The solution enables near real-time visualization of DynamoDB application metrics in Grafana dashboards, allowing quick identification of performance issues and easier troubleshooting.</p> 
<h3>Solution overview</h3> 
<p>The following diagram illustrates the different components of this architecture.</p> 
<div class="wp-caption alignnone" id="attachment_60820" style="width: 905px;">
 <img alt="The diagram depicts the architecture diagram" class="wp-image-60820 size-full" height="447" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/05/14/Architecture_Diagram_1.jpg" title="Figure 1: The diagram depicts the architecture diagram" width="895" />
 <p class="wp-caption-text" id="caption-attachment-60820">Figure 1:Architecture diagram</p>
</div> 
<p>At a high level, the steps can be summarized as follows:</p> 
<ul> 
 <li>Enable zero-ETL integration to synchronize the data between Amazon DynamoDB and Amazon OpenSearch Service.</li> 
 <li>Use Amazon Managed Grafana built-in data source for Amazon OpenSearch Service to discover the OpenSearch Service accounts, and manage the configuration of the authentication credentials that are required to access OpenSearch.</li> 
 <li>Build dashboards in Amazon Managed Grafana to view near real-time health of your Amazon DynamoDB.</li> 
</ul> 
<h3>Prerequisites</h3> 
<p>For this walk through, you will need the following:</p> 
<ul> 
 <li>AWS account</li> 
 <li><a href="https://docs.aws.amazon.com/iam/">IAM user/role</a> with required permissions to perform tasks in walk through section</li> 
 <li><a href="https://aws.amazon.com/cli/">AWS CLI</a></li> 
 <li><a href="https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html">AWS CDK</a></li> 
 <li><a href="https://git-scm.com/downloads">Git</a>, <a href="https://docs.npmjs.com/downloading-and-installing-node-js-and-npm">NPM</a>, <a href="https://docs.npmjs.com/downloading-and-installing-node-js-and-npm">Node.js</a>, <a href="https://www.python.org/downloads/">Python 3.x</a>, <a href="https://pypi.org/project/boto3/">boto 3 Python library</a> installed locally</li> 
 <li><a href="https://console.aws.amazon.com/grafana/">Amazon Managed Grafana workspace</a></li> 
</ul> 
<h3>Walk through</h3> 
<p>The following sections walk you through the solution. For this demonstration we will use movies data as the sample dataset.</p> 
<h3>Enabling zero-ETL integration between Amazon DynamoDB and Amazon OpenSearch service</h3> 
<p>To begin, you will deploy the necessary AWS services using CDK-based Infrastructure as Code (IaC), enabling zero-ETL integration between DynamoDB and Amazon OpenSearch. The initial step is to configure your local environment for CDK deployment. For that you need to establish credentials to authenticate with your AWS account from your local environment. The preferred and secured approach is to use temporary credentials as per the instructions in <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-short-term.html">Authenticating with short-term credentials for the AWS CLI</a>.</p> 
<ol> 
 <li>Setup and CDK bootstrapping: <pre><code class="lang-bash"># Clone the solution repository
git clone https://github.com/aws-samples/sample-grafana-opensearch-dynamodb.git
cd &lt;directory&gt;

# Install dependencies
npm install

# Bootstrap the CDK 
cdk boostrap</code></pre> </li> 
 <li>Creating a DynamoDB table:<br /> To create the DynamoDB table, run the CDK stack named ‘DynamoDBStack’ using the following command. This will create a DynamoDB table named ‘movies-datastore’ with a primary key named ‘title’. As part of the table creation, the stack also enables “Streams” on your DynamoDB table with the stream view type as “New and old images”.<p></p> <pre><code class="lang-bash">cdk deploy DynamoDBStack</code></pre> </li> 
 <li>Creating an OpenSearch domain:<br /> Next, create the Amazon OpenSearch domain for storing and searching the “movies” data. To do this, run the CDK stack named ‘OpenSearchStack’ using the following command. This stack creates an Amazon OpenSearch domain with required configurations along with the following resources. a/ A KMS key with the policies for encrypting the data in the OpenSearch domain and b/ An IAM Role with the permissions to access the DynamoDB table, its associated streams, and the OpenSearch domain.<p></p> <pre><code class="lang-bash">cdk deploy OpenSearchStack</code></pre> </li> 
 <li>Creating the OpenSearch Ingestion Pipeline for the zero-ETL integration:<br /> Now that you have the DynamoDB table, which is the source, and the OpenSearch domain, which is the destination, create the OpenSearch ingestion pipeline. There are two ways to create the ingestion pipeline:<br /> (a) Using AWS Management Console to configure the “Zero-ETL with DynamoDB” blueprint from the available templates.<br /> (b) Programmatically running the CKD stack ‘PipelineStack’ which creates the following: (1) OpenSearch ingestion pipeline with the following configurations: DynamoDB table as the source, OpenSearch domain as the sink, Associating the IAM role created in step 3, and Streaming ingestion for data updates and (2) CloudWatch log group to write the ingestion log.<p></p> <pre><code class="lang-bash">cdk deploy PipelineStack</code></pre> </li> 
 <li>Configuring authorization within the OpenSearch domain:<br /> OpenSearch authentication is configured at the AWS IAM level and the authorization is configured within the OpenSearch security settings accessed through the OpenSearch domain url. You should always follow the principle of <a href="https://docs.aws.amazon.com/wellarchitected/latest/framework/sec_permissions_least_privileges.html">least privilege</a> while granting access.Using the <a href="https://signin.aws.amazon.com/signup">AWS Management Console</a>, navigate to <a href="https://us-east-2.signin.aws.amazon.com/oauth">AWS Secrets Manager</a> -&gt; Secrets -&gt; OpenSearchMasterUser to obtain the master user credentials to log in to the OpenSearch domain. Once you have authenticated to the OpenSearch domain, using the hamburger menu on the left side, navigate to Security -&gt; Roles to authorize the IAM role that was created in step 3.<p></p> 
  <div class="wp-caption alignnone" id="attachment_59665" style="width: 946px;">
   <img alt="Use the hamburger menu on the left side and navigate to &quot;Security” -&gt; “Roles”" class="wp-image-59665 size-full" height="340" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/03/13/OpenSearch-1.png" width="936" />
   <p class="wp-caption-text" id="caption-attachment-59665">Figure 2: Use the hamburger menu on the left side and navigate to “Security” -&gt; “Roles”</p>
  </div> <p>Next, choose the “all_access” role and select “Manage mapping” within the “Mapped users” tab.</p> 
  <div class="wp-caption alignnone" id="attachment_59666" style="width: 946px;">
   <img alt="Choosing the “all_access” role and selecting “Manage mapping” within the “Mapped users” tab" class="wp-image-59666 size-full" height="334" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/03/13/OpenSearch-2.png" width="936" />
   <p class="wp-caption-text" id="caption-attachment-59666">Figure 3: Choose the “all_access” role and select “Manage mapping” within the “Mapped users” tab</p>
  </div> <p>Next, enter the ARN of the IAM role created in step 3 in “Backend roles” and select “Map”.</p> 
  <div class="wp-caption alignnone" id="attachment_59667" style="width: 946px;">
   <img alt="Entering the ARN of the IAM role created in step 3 in “Backend roles” and selecting “Map”" class="wp-image-59667 size-full" height="466" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/03/13/OpenSearch-3.png" width="936" />
   <p class="wp-caption-text" id="caption-attachment-59667">Figure 4: Enter the ARN of the IAM role created in step 3 in “Backend roles” and select “Map”</p>
  </div> <p>You can validate the above implementation by running the python script using the following command that loads the DynamoDB table with a sample movie data. If everything is configured correctly, the zero-ETL integration will automatically capture the DynamoDB table updates and stream them directly into the OpenSearch domain through the ingestion pipeline. For monitoring, use the CloudWatch logs created in step 4.</p> <pre><code class="lang-bash">python UploadData.py</code></pre> </li> 
</ol> 
<h3>Configuring Amazon Managed Grafana</h3> 
<ol> 
 <li>Creating a Grafana workspace:<br /> Grafana is a visualization tool that is popular for its extensible data support. Amazon Managed Grafana is a fully managed and secure data visualization service that is easy to deploy, operate, and scale. To create a Grafana workspace, follow the steps mentioned in <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-create-workspace.html">Creating a workspace</a>.</li> 
 <li>Configuring OpenSearch authorization for <a href="https://docs.aws.amazon.com/grafana/latest/userguide/using-service-linked-roles.html#slr-permissions">Amazon Managed Grafana service-linked</a> IAM role:<br /> A <a href="https://docs.aws.amazon.com/grafana/latest/userguide/using-service-linked-roles.html#create-slr">service-linked role</a> makes setting up Amazon Managed Grafana easier because you don’t have to manually add the necessary permissions, as these roles are predefined by Amazon Managed Grafana and include all the permissions that the service requires to call other AWS services on your behalf. In our use-case, the defined permissions include the trust policy and the permissions policy to access OpenSearch. You can create a service-linked role via AWS Management Console, the AWS CLI, or the AWS API. Grafana needs additional permissions to access OpenSearch because authorization is configured within the OpenSearch security settings. To configure the necessary permissions, refer to section 1, step 5 where we covered the setup process.</li> 
 <li>Configuring Amazon OpenSearch Service data source:<br /> Using OpenSearch Service data source, you can perform OpenSearch queries in Grafana in order to visualize data that stored in OpenSearch and build dashboards. To configure Amazon OpenSearch Service as a data source in Grafana workspace, follow the steps mentioned in <a href="https://docs.aws.amazon.com/grafana/latest/userguide/ES-adding-AWS-config.html">Use AWS data source configuration to add OpenSearch Service as a data source</a>. The following image shows a successfully configured OpenSearch data source.<p></p> <p></p>
  <div class="wp-caption alignnone" id="attachment_59673" style="width: 686px;">
   <img alt="Grafana data source configuration to add OpenSearch Service as a data source" class="wp-image-59673 size-large" height="1024" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/03/13/Grafana_1-676x1024.jpg" width="676" />
   <p class="wp-caption-text" id="caption-attachment-59673">Figure 5: Grafana data source configuration to add OpenSearch Service as a data source</p>
  </div></li> 
 <li>Creating a dashboard:<br /> A dashboard is a set of one or more panels that allows you to show your data in a visual form. To create a dashboard, follow the steps mentioned in <a href="https://docs.aws.amazon.com/grafana/latest/userguide/v10-dash-creating.html">Creating dashboards</a>. Make sure to select Amazon OpenSearch Service as your data source in the panel. The following image shows a sample dashboard.<p></p> 
  <div class="wp-caption alignnone" id="attachment_59674" style="width: 1034px;">
   <img alt="Grafana dashboard querying Amazon OpenSearch to visualize data from DynamoDB" class="wp-image-59674 size-large" height="536" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/03/13/Grafana_2-1024x536.jpg" width="1024" />
   <p class="wp-caption-text" id="caption-attachment-59674">Figure 6: Grafana dashboard querying Amazon OpenSearch to visualize data from DynamoDB</p>
  </div> <p>A dashboard in Grafana is represented by a JSON object, which stores metadata of its dashboard. You can create a similar dashboard by uploading the provided <a href="https://github.com/aws-samples/sample-grafana-opensearch-dynamodb/tree/main/dashboard">dashboard.json file</a> and following the steps mentioned in <a href="https://docs.aws.amazon.com/grafana/latest/userguide/dashboard-export-and-import.html#importing-a-dashboard">Importing a dashboard</a>.</p></li> 
</ol> 
<h3>Cleanup</h3> 
<p>To avoid ongoing charges in your AWS account, you should delete the AWS resources by running the CDK destroy command from the project directory. Furthermore, log in to the <a href="https://console.aws.amazon.com/">AWS Management Console</a> and delete any manually created resources.</p> 
<pre><code class="lang-bash">cdk destroy --all</code></pre> 
<h3>Conclusion</h3> 
<p>In this post, you learned how to visualize metrics stored in DynamoDB in near real-time Grafana dashboards. We provided you with an infrastructure as code (IaC) template and a sample dashboard JSON file to test the solution. We are here to help and if you need further assistance, reach out to AWS Support and your AWS account team.</p>
