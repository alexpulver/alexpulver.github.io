---
title: "Drilling Down to Amazon EC2 Instances Data Transfer Costs"
date: 2019-05-16
aliases:
  - /2019/05/16/drilling-down-to-amazon-ec2-instances-data-transfer-costs
---

The use case I would like to address in this post revolves around the need to understand which specific Amazon EC2 instances incur the highest data transfer charges, including high-level understanding of traffic flow (e.g. to/from Internet, inter-region, intra-region and etc.). For detailed analysis of traffic flow, I would recommend to use Amazon VPC Flow Logs – see [Querying Amazon VPC Flow Logs](https://docs.aws.amazon.com/athena/latest/ug/vpc-flow-logs.html) in Amazon Athena (serverless query service) documentation.

First, I’d like to mention that for most cost management needs, you should first take a look at AWS Cost Explorer. It has an easy-to-use interface that lets you visualize, understand, and manage your AWS costs and usage over time. Get started quickly by creating custom reports (including charts and tabular data) that analyze cost and usage data, both at a high level (e.g., total costs and usage across all accounts) and for highly-specific requests (e.g., `m2.2xlarge` costs within account Y that are tagged "project: secretProject"). Using AWS Cost Explorer, you can dive deeper into your cost and usage data to identify trends, pinpoint cost drivers, and detect anomalies.

Currently, [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) doesn’t provide details for a specific resource ID – EC2 instance in our case. On the other hand, [AWS Cost and Usage report](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-reports-costusage.html) tracks your AWS usage and provides estimated charges associated with your AWS account. The report contains line items for each unique combination of AWS product, usage type, and operation that your AWS account uses. You can customize the AWS Cost and Usage report to aggregate the information either by the hour or by the day.

Below I’ll show you how to use Amazon Athena to analyze the data from your AWS Cost and Usage report in Amazon S3 using standard SQL. This enables you to avoid creating your own data warehouse solutions to query AWS Cost and Usage report data.

First, follow the instructions in [Uploading an AWS Cost and Usage Report to Amazon Athena](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/athena.html) to create a report in Amazon S3 and configure Amazon Athena database on top of it.

The query below will produce a list of Amazon EC2 instance IDs sorted by total data transfer usage. `cost_and_usage_report` is the name I chose for Amazon Athena table.

```sql
SELECT
  line_item_resource_id,
  round(SUM(line_item_usage_amount), 2) AS sum_line_item_usage_amount
FROM cost_and_usage_report
WHERE
  line_item_product_code = 'AmazonEC2'
  AND product_product_family = 'Data Transfer'
  AND regexp_like(line_item_resource_id, 'i-')
GROUP BY line_item_resource_id
ORDER BY sum_line_item_usage_amount DESC
LIMIT 100
```

The result will look something like this:

{{< rawhtml >}}
<div style="overflow-x:auto;">
  <table>
      <tr>
          <th>line_item_resource_id</th>
          <th>sum_line_item_usage_amount</th>
      </tr>
      <tr>
          <td>i-0526081c…</td>
          <td>11.81</td>
      </tr>
      <tr>
          <td>i-04946b12…</td>
          <td>9.08</td>
      </tr>
      <tr>
          <td>i-0ae87b25…</td>
          <td>0.03</td>
      </tr>
  </table>
</div>
{{< /rawhtml >}}

Now we can dive deeper into the details of the specific Amazon EC2 instance using the next query:

```sql
SELECT
  line_item_resource_id,
  product_transfer_type,
  product_from_location,
  product_to_location,
  line_item_operation,
  line_item_line_item_description,
  round(SUM(line_item_usage_amount), 2) AS sum_line_item_usage_amount
FROM cost_and_usage_report
WHERE
  line_item_resource_id = 'i-0526081c...'
  AND product_product_family = 'Data Transfer'
GROUP BY
  line_item_resource_id,
  product_transfer_type,
  product_from_location,
  product_to_location,
  line_item_operation,
  line_item_line_item_description
ORDER BY sum_line_item_usage_amount DESC
LIMIT 10
```

The result below shows, for example, that most of the data transfer is attributed to intra-region traffic coming through the instance’s public IP address:

{{< rawhtml >}}
<div style="overflow-x:auto;">
  <table>
      <tr>
          <th>line_item_resource_id</th>
          <th>product_transfer_type</th>
          <th>product_from_location</th>
          <th>product_to_location</th>
          <th>line_item_operation</th>
          <th>line_item_line_item_description</th>
          <th>sum_line_item_usage_amount</th>
      </tr>
      <tr>
          <td>i-0526081c...</td>
          <td>IntraRegion</td>
          <td>EU (Ireland)</td>
          <td>EU (Ireland)</td>
          <td>PublicIP-In</td>
          <td>$0.010 per GB – regional data transfer – in/out/between EC2 AZs or using elastic IPs or ELB</td>
          <td>9.18</td>
      </tr>
      <tr>
          <td>i-0526081c...</td>
          <td>IntraRegion</td>
          <td>EU (Ireland)</td>
          <td>EU (Ireland)</td>
          <td>InterZone-In</td>
          <td>$0.010 per GB – regional data transfer – in/out/between EC2 AZs or using elastic IPs or ELB</td>
          <td>1.64</td>
      </tr>
      <tr>
          <td>i-0526081c...</td>
          <td>IntraRegion</td>
          <td>EU (Ireland)</td>
          <td>EU (Ireland)</td>
          <td>PublicIP-Out</td>
          <td>$0.010 per GB – regional data transfer – in/out/between EC2 AZs or using elastic IPs or ELB</td>
          <td>0.78</td>
      </tr>
      <tr>
          <td>i-0526081c...</td>
          <td>IntraRegion</td>
          <td>EU (Ireland)</td>
          <td>EU (Ireland)</td>
          <td>InterZone-Out</td>
          <td>$0.010 per GB – regional data transfer – in/out/between EC2 AZs or using elastic IPs or ELB</td>
          <td>0.2</td>
      </tr>
      <tr>
          <td>i-0526081c...</td>
          <td>AWS Outbound</td>
          <td>EU (Ireland)</td>
          <td>External</td>
          <td>RunInstances</td>
          <td>$0.000 per GB – first 1 GB of data transferred out per month</td>
          <td>0</td>
      </tr>
      <tr>
          <td>i-0526081c...</td>
          <td>AWS Inbound</td>
          <td>External</td>
          <td>EU (Ireland)</td>
          <td>RunInstances</td>
          <td>$0.000 per GB – data transfer in per month</td>
          <td>0</td>
      </tr>
  </table>
</div>
{{< /rawhtml >}}

If we would like to understand what is the exact source of the traffic at this point, we would need to get the ENI ID of the instance and query Amazon VPC Flow Logs as mentioned in the beginning of this post.