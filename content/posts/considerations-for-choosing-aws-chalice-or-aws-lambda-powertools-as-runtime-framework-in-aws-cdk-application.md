---
title: "Considerations for choosing AWS Chalice or AWS Lambda Powertools as runtime framework in an AWS CDK application"
date: 2022-05-30T20:11:37+03:00
draft: true
---

Missing structure and outline:

* **Intro to the blog post** I’m about to read - what prompted you to write a trade-off (guide?)?
* **Intro to each tool** you are assessing - what’s their intended use? e.g., infra+runtime, runtime only, infra only, etc.
* **Intro to your criteria/requirements **- what is it that will make you more inclined to choose X vs Y?
* **Section for each tool** - what draws people to these tools? what are their rough edges and possible solutions? can they be used together (use their strengths)?
* **Your verdict** - what your current decision is based on your criteria/requirements and experience so far with all tools


Notes:

* Confused between the connections with Chalice, Construct Library, and Powertools - did you mean to talk about these 3 different options and their trade-offs? If so, make that more explicit (sub-sections can help)
* cdk-chalice appeared suddenly and made me reconsider whether I was understanding the post - is Alex covering cdk-chalice or AWS chalice or both?
* There’s quite a few of implementation details that might lose readers unfamiliar, like metric methods - examples could make these more explicit and educational
* Creating sections will help you strike a good balance between high level and details for each of them. It’ll also help the reader naturally switch context or jump ahead to the tool they’re mostly curious about


AWS Chalice generates Amazon API Gateway methods and resources for each path annotated in AWS Chalice code. This allows to configure granular usage plans, authentication methods, generate OpenAPI documentation, SDK, and more. The same configuration can be created using aws_apigatewayv2, but then would require manual synchronization between infrastructure and runtime code.

AWS Chalice validates that environment variables used in runtime code are configured in the AWS SAM template. I couldn’t think of an approach to do the same using AWS Construct Library and AWS Lambda Powertools - the infrastructure and runtime are fully decoupled here. I could project a file with environment variables from the infrastructure code to the runtime code. But the validation would still need to rely on tests (which can have human error/omission) as compared to synthesis time library validation.

AWS Chalice has a [test client](https://aws.github.io/chalice/topics/testing.html) that supports generating various service events and an HTTP client interface. AWS Lambda Powertools requires specifying the event and context objects without additional typed abstraction that is a part of the library.

AWS Chalice produces AWS SAM template, hence features available are those supported by AWS SAM. AWS Construct Library provides the flexibility to use all AWS Lambda configuration options.

AWS Chalice produces certain CloudFormation outputs, that can’t be controlled through cdk-chalice. Hence users get surprising CloudFormation outputs in their stacks. Using aws_apigatewayv2 + aws_lambda_python provides the ability to fully control what CloudFormation outputs are created, and when.

cdk-chalice user had to understand AWS Chalice configuration format, in addition to AWS Construct Library API. Using aws_apigatewayv2 + aws_lambda_python doesn’t require this additional effort.

cdk-chalice exposes aws_cdk.aws_sam L1 constructs due to AWS Chalice producing AWS SAM template. These don’t have convenience methods such as the various aws_cdk.aws_lambda.Function [metric](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Function.html#metricmetricname-props) methods. AWS Construct Library exposes L2 constructs for Amazon API Gateway and AWS Lambda, so allows to benefit from L2 abstraction capabilities.

cdk-chalice requires defining and versioning AWS Chalice configuration file. The configuration file serves as a passthrough to the final CloudFormation template generated by AWS CDK ([@aws-cdk/cloudformation-include](https://docs.aws.amazon.com/cdk/api/latest/docs/cloudformation-include-readme.html) module is used to merge the AWS SAM template with the broader AWS CDK application). The presence of this file may be confusing to library users, since it is internal implementation details that leaks. Additionally, AWS CDK [Token](https://docs.aws.amazon.com/cdk/latest/guide/tokens.html) values are randomized on each synthesis and marks the configuration file as modified in Git.

If there is a configuration not supported through AWS Chalice, the developer needs to look up the resource name in the synthesized AWS CloudFormation template, then customize the resource in the AWS CDK code. The resource is accessible through cdk-chalice API and is based on [@aws-cdk/cloudformation-include](https://docs.aws.amazon.com/cdk/api/latest/docs/cloudformation-include-readme.html) module.
