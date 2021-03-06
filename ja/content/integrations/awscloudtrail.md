---
description: Alert on suspicious AWS account activity.
integration_title: AWS CloudTrail
kind: integration
newhlevel: true
placeholder: true
title: Datadog-AWS CloudTrail Integration
---

<div class='alert alert-info'><strong>NOTICE:</strong>アクセスいただきありがとうございます。こちらのページは現在英語のみのご用意となっております。引き続き日本語化の範囲を広げてまいりますので、皆様のご理解のほどよろしくお願いいたします。</div>



{{< img src="integrations/awscloudtrail/cloudtrail_event.png" >}}

## Overview

AWS CloudTrail provides an audit trail for your AWS account. Datadog reads this audit trail and creates events you can search for in your stream and use for correlation on your dashboards. Here is an example of a CloudTrail event:

For information about the rest of the AWS services, see the [AWS tile][1]

## Setup
### Installation

Configure CloudWatch on AWS and ensure that the policy you created has the equivalent of the **AWSCloudTrailReadOnlyAccess** policy assigned. The actions in that policy are **s3:ListBucket**, **s3:GetBucketLocation**, and **s3:GetObject**. Also ensure that the policy gives access to the S3 bucket you selected for the CloudTrail Trail. Here is an example policy to give access to an S3 bucket.

{{< highlight json >}}
{
  "Version": "2012-10-17",
  "Statement": [
  {
    "Action": [
      "s3:ListBucket",
      "s3:GetBucketLocation",
      "s3:GetObject"
    ],
    "Effect": "Allow",
    "Resource": [
      "arn:aws:s3:::your-s3-bucket-name",
      "arn:aws:s3:::your-s3-bucket-name/*"
    ]
  } ]
}
{{< /highlight >}}


### Configuration

Open the AWS CloudTrail tile. The accounts you configured in the Amazon Web Services tile are shown here and you can choose what kinds of events will be collected by Datadog. If you would like to see other events that are not mentioned here, please reach out to [our support team][2].


## Troubleshooting

### I don't see a CloudTrail tile or there are no accounts listed

You need to first configure the [Amazon Web Services tile][1]. Once you complete this, the CloudTrail tile will be available and configurable.

[1]: /integrations/aws
[2]: /help
