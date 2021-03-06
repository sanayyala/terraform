---
layout: "aws"
page_title: "AWS: aws_lambda_function"
sidebar_current: "docs-aws-resource-lambda-function"
description: |-
  Provides a Lambda Function resource. Lambda allows you to trigger execution of code in response to events in AWS. The Lambda Function itself includes source code and runtime configuration.
---

# aws\_lambda\_function

Provides a Lambda Function resource. Lambda allows you to trigger execution of code in response to events in AWS. The Lambda Function itself includes source code and runtime configuration.

For information about Lambda and how to use it, see [What is AWS Lambda?][1]

## Example Usage

```
resource "aws_iam_role" "iam_for_lambda" {
  name = "iam_for_lambda"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_lambda_function" "test_lambda" {
  filename         = "lambda_function_payload.zip"
  function_name    = "lambda_function_name"
  role             = "${aws_iam_role.iam_for_lambda.arn}"
  handler          = "exports.test"
  source_code_hash = "${base64sha256(file("lambda_function_payload.zip"))}"

  environment {
    variables = {
      foo = "bar"
    }
  }
}
```

## Argument Reference

* `filename` - (Optional) A [zip file][2] containing your lambda function source code. If defined, The `s3_*` options cannot be used.
* `s3_bucket` - (Optional) The S3 bucket location containing your lambda function source code. Conflicts with `filename`.
* `s3_key` - (Optional) The S3 key containing your lambda function source code. Conflicts with `filename`.
* `s3_object_version` - (Optional) The object version of your lambda function source code. Conflicts with `filename`.
* `function_name` - (Required) A unique name for your Lambda Function.
* `handler` - (Required) The function [entrypoint][3] in your code.
* `role` - (Required) IAM role attached to the Lambda Function. This governs both who / what can invoke your Lambda Function, as well as what resources our Lambda Function has access to. See [Lambda Permission Model][4] for more details.
* `description` - (Optional) Description of what your Lambda Function does.
* `memory_size` - (Optional) Amount of memory in MB your Lambda Function can use at runtime. Defaults to `128`. See [Limits][5]
* `runtime` - (Required) See [Runtimes][6] for valid values.
* `timeout` - (Optional) The amount of time your Lambda Function has to run in seconds. Defaults to `3`. See [Limits][5]
* `publish` - (Optional) Whether to publish creation/change as new Lambda Function Version. Defaults to `false`.
* `vpc_config` - (Optional) Provide this to allow your function to access your VPC. Fields documented below. See [Lambda in VPC][7]
* `environment` - (Optional) The Lambda environment's configuration settings. Fields documented below.
* `kms_key_arn` - (Optional) The ARN for the KMS encryption key.
* `source_code_hash` - (Optional) Used to trigger updates. This is only useful in conjunction with `filename`.
  The only useful value is `${base64sha256(file("file.zip"))}`.

**vpc\_config** requires the following:

* `subnet_ids` - (Required) A list of subnet IDs associated with the Lambda function.
* `security_group_ids` - (Required) A list of security group IDs associated with the Lambda function.

~> **NOTE:** if both `subnet_ids` and `security_group_ids` are empty then vpc_config is considered to be empty or unset.

For **environment** the following attributes are supported:

* `variables` - (Optional) A map that defines environment variables for the Lambda function.

## Attributes Reference

* `arn` - The Amazon Resource Name (ARN) identifying your Lambda Function.
* `qualified_arn` - The Amazon Resource Name (ARN) identifying your Lambda Function Version
  (if versioning is enabled via `publish = true`).
* `version` - Latest published version of your Lambda Function.
* `last_modified` - The date this resource was last modified.
* `kms_key_arn` - (Optional) The ARN for the KMS encryption key.
* `source_code_hash` - Base64-encoded representation of raw SHA-256 sum of the zip file
  provided either via `filename` or `s3_*` parameters.

[1]: https://docs.aws.amazon.com/lambda/latest/dg/welcome.html
[2]: https://docs.aws.amazon.com/lambda/latest/dg/walkthrough-s3-events-adminuser-create-test-function-create-function.html
[3]: https://docs.aws.amazon.com/lambda/latest/dg/walkthrough-custom-events-create-test-function.html
[4]: https://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html
[5]: https://docs.aws.amazon.com/lambda/latest/dg/limits.html
[6]: https://docs.aws.amazon.com/lambda/latest/dg/API_CreateFunction.html#SSS-CreateFunction-request-Runtime
[7]: http://docs.aws.amazon.com/lambda/latest/dg/vpc.html

## Import

Lambda Functions can be imported using the `function_name`, e.g.

```
$ terraform import aws_lambda_function.tesr_lambda my_test_lambda_function
```
