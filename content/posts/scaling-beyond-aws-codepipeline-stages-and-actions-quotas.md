---
title: "Scaling beyond AWS CodePipeline stages and actions quotas with CDK Pipelines"
date: 2022-05-29T14:40:18+03:00
draft: true
---

AWS CodePipeline has (at the time of this writing) a [non-adjustable quota](https://docs.aws.amazon.com/codepipeline/latest/userguide/limits.html) of 50 stages in a pipeline, 50 actions in a stage, and total of 500 actions per pipeline. High number of environments or artifacts can drive a pipeline to exceed 50 stages, hitting the non-adjustable quota.

To scale beyond these quotas, consider AWS Step Functions. You can use the [Map](https://docs.aws.amazon.com/step-functions/latest/dg/amazon-states-language-map-state.html) state to fan out execution of AWS CodeBuild project(s). CodePipeline has a [native integration with Step Functions](https://docs.aws.amazon.com/codepipeline/latest/userguide/integrations-action-type.html#w2aac11b7c17c11), so no custom code would be required for such invocation. Step Functions, in turn, has a [native integration with CodeBuild](https://docs.aws.amazon.com/step-functions/latest/dg/connect-codebuild.html), so no custom code would be required here as well.

The flow would be:
CDK Pipelines `Assets` stage => `Step Functions` action => `Map` state with artifact IDs (potentially with concurrency) => `CodeBuild` project => [`cdk-assets`](https://www.npmjs.com/package/cdk-assets) command.

The `Map` state can accept artifact IDs managed by the [CDK Pipelines](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.pipelines-readme.html) module, and set them as environment variables for the CodeBuild project (using [`environmentVariablesOverride`](https://docs.aws.amazon.com/codebuild/latest/APIReference/API_StartBuild.html#CodeBuild-StartBuild-request-environmentVariablesOverride) parameter). The build would use `cdk-assets` to uploads these artifacts. Map state supports concurrency, which can further speed up processing. Step Functions now supports payload sizes up to 256KB, which should support processing large number of artifacts in a single CodePipeline stage and action. The number of artifacts processed depends on what will be the actual parameters passed to CodeBuild from Map state array.