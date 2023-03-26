---
title: "How to Manage AWS CDK Application Configuration"
date: 2023-03-26
draft: false
---
AWS CDK applications need to maintain certain configuration in the version control repository. Configuration can include application name, programming language version, version control branch, build configuration, account and Region per environment, etc. I often see the use of configuration files (e.g. `cdk.json`) for this purpose, in formats such as JSON and YAML.

In this post, I describe my recommendations for managing AWS CDK application configuration.

## Context

From my experience, builders use configuration files mainly to avoid: 1/ changing the application code because they don't own it 2/ triggering a build 3/ performing a deployment. With the AWS CDK, I think neither of these reasons makes configuration files more efficient than source code. Let's break them down below.

### Changing the application code
Builders who own the AWS CDK application configuration also own the application code. Hence, putting the configuration in, for example, `config.json` or `config.ts` is the same in terms of discoverability and ease of change.

### Triggering a build
The AWS CDK [application lifecycle](https://docs.aws.amazon.com/cdk/v2/guide/apps.html#lifecycle) always includes a build. Hence, changing `config.json` or `config.ts` would require a build.

### Performing a deployment
Applying the change to AWS CDK application configuration requires deployment by definition.

## Recommended approach

I recommend using application code to store configuration. For example:

```python
from collections import namedtuple

import aws_cdk.aws_codebuild as codebuild

BackendEnvironment = namedtuple("BackendEnvironment", ["name", "account", "region"])

GITHUB_CONNECTION_ARN = "arn:aws:codestar-connections:eu-west-1:111111111111:connection/1f244295-871f-411f-afb1-e6ca987858b6"
GITHUB_OWNER = "owner"
GITHUB_REPO = "repo"
GITHUB_TRUNK_BRANCH = "main"
CODEBUILD_BUILD_ENVIRONMENT = codebuild.BuildEnvironment(
    build_image=codebuild.LinuxBuildImage.STANDARD_5_0,
    privileged=True,
)
BACKEND_ENVIRONMENTS = [
    BackendEnvironment(name="Production", account="111111111111", region="eu-west-1"),
]
```
Using a programming language for configuration provides several benefits not found in textual configuration files. These include type-checking, static code analysis, auto-completion, and reuse. In Python, for example, you can model different environments using [data classes](https://docs.python.org/3.7/library/dataclasses.html) or flat constants.

## Conclusion

I hope this post can be useful to you for deciding on how to manage AWS CDK application configuration. If you think I’ve missed something, feel free to ping me [@alex_pulver](https://twitter.com/alex_pulver). Happy coding!