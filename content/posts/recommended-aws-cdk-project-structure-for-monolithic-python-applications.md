---
title: "Recommended AWS CDK Project Structure for Monolithic Python Applications"
date: 2023-03-25
draft: false
---

In my [Recommended AWS CDK project structure for Python applications](https://aws.amazon.com/blogs/developer/recommended-aws-cdk-project-structure-for-python-applications/) blog post, the application is a component (User Management Backend) with a dedicated Git repository and a single pipeline. This is the option #4 in the figure below.

{{< figure src="/images/recommended-aws-cdk-project-structure-for-monolithic-python-applications-1.png" title="Source: https://d1.awsstatic.com/events/Summits/reinvent2022/DOP321_Organize-application-components-into-repositories-and-pipelines-with-AWS-CDK.pdf" >}}

Some builders initially prefer to manage the workload as a monolith in a single repository with one or more pipelines. In this post, I describe an application for options #1 and #2 in the above figure.

## Concepts

I use the following [terminology](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html) from the AWS Well-Architected Framework:
* A **component** is the code, configuration, and AWS Resources that together deliver against a requirement. A component is often the unit of technical ownership, and is decoupled from other components.
* The term **workload** is used to identify a set of components that together deliver business value. A workload is usually the level of detail that business and technology leaders communicate about.
* We think about **architecture** as being how components work together in a workload. How components communicate and interact is often the focus of architecture diagrams.

## Architecture

I am extending the User Management workload to 3 components: `Users UI`, `Users API` (`Backend` in the original post), and `Chatbot API`.

{{< figure src="/images/recommended-aws-cdk-project-structure-for-monolithic-python-applications-2.png" title="User Management workload" >}}

## Project structure

The project now has a top-level package for the workload. The workload package contains a sub-package for each component.

```shell
usermanagement/
    chatbot_api/
        nlp/
            runtime/
                lambda_function.py
        component.py  # defines NLP infrastructure
    users_api/
        api/
            runtime/
                lambda_function.py
            infrastructure.py
        database/
            infrastructure.py
        operations/
            infrastructure.py
        component.py
    users_ui/
        spa/
            runtime/
                index.html
        component.py  # defines CDN and SPA infrastructure
    workload.py
app.py
toolchain.py
```

_Note:_ The infrastructure for Chatbot API and Users UI logical units (NLP and CDN/SPA respectively) is quite simple. Hence, I decided to define the infrastructure directly in the `component.py` file. I did create dedicated directories for the logical units' runtime code as it is more involved.

The new `usermanagement/workload.py` module composes the components into the deployment layout. But first, I need to decide on the deployment layout itself :smiley:. The options range from workload as a single stack to each component as one or more stacks. I will describe both below.

Let's go through the related changes in `app.py`, `usermanagement/users_api/component.py` (`backend/component.py` in the original blog post), `toolchain.py`, and introduce the new `usermanagement/workload.py`.

## Workload as a single stack

**app.py**

This time, I import the workload class instead of the component class. I also change `constants.APP_NAME` from `UserManagementBackend` (component) to `UserManagement` (workload).

```python
# The AWS CDK application entry point
...
import constants
from usermanagement.workload import UserManagement
from toolchain import Toolchain

app = cdk.App()

# Workload sandbox stack
UserManagement(
    app,
    constants.APP_NAME + "Sandbox",
    api_lambda_reserved_concurrency=1,
    database_dynamodb_billing_mode=dynamodb.BillingMode.PAY_PER_REQUEST,
    env=cdk.Environment(
        account=os.environ["CDK_DEFAULT_ACCOUNT"],
        region=os.environ["CDK_DEFAULT_REGION"],
    ),
)

# Toolchain stack (defines the continuous deployment pipeline)
Toolchain(
    app,
    constants.APP_NAME + "Toolchain",
    env=cdk.Environment(account="111111111111", region="eu-west-1"),
)

app.synth()
```

**usermanagement/workload.py**

The `UserManagement` class implements a workload stack that composes the components together.

```python
# User Management workload deployment layout
...
from usermanagement.chatbot_api.component import ChatbotAPI
from usermanagement.users_api.component import UsersAPI
from usermanagement.users_ui.component import UsersUI

class UserManagement(cdk.Stack):
    def __init__(
        self,
        scope: cdk.Construct,
        id_: str,
        *,
        api_lambda_reserved_concurrency: int,
        database_dynamodb_billing_mode: dynamodb.BillingMode,
        **kwargs: Any,
    ):
        super().__init__(scope, id_, **kwargs)

        chatbot_api = ChatbotAPI(self, "ChatbotAPI")
        users_api = UsersAPI(
            self,
            "UsersAPI",
            api_lambda_reserved_concurrency=1,
            database_dynamodb_billing_mode=dynamodb.BillingMode.PAY_PER_REQUEST,
        )
        users_ui = UsersUI(
            self, 
            "UsersUI",
            chatbot_api_endpoint=chatbot_api.endpoint,
            users_api_endpoint=users_api.endpoint,
        )
	
        self.users_ui_endpoint = cdk.CfnOutput(
            self, 
            "UsersUIEndpoint", 
            value=users_ui.endpoint,
        )
```

**usermanagement/users_api/component.py**

The component becomes a construct. That also means removing `**kwargs` because the [`cdk.Construct`](https://docs.aws.amazon.com/cdk/api/v2/python/constructs/Construct.html) base class accepts only `scope` and `id_` parameters. I change the definition of the `self.endpoint` (`self.api_endpoint` in the original post) attribute from `cdk.CfnOutput` to pure value, because I want to centralize all stack outputs in `usermanagement/workload.py`.

```python
# Users API component
...
from usermanagement.users_api.api.infrastructure import API
from usermanagement.users_api.database.infrastructure import Database
from usermanagement.users_api.monitoring.infrastructure import Monitoring

class UsersAPI(cdk.Construct):
    def __init__(
        self,
        scope: cdk.Construct,
        id_: str,
        *,
        api_lambda_reserved_concurrency: int,
        database_dynamodb_billing_mode: dynamodb.BillingMode,
    ):
        super().__init__(scope, id_)
		
        database = Database(
            self, 
            "Database", 
            dynamodb_billing_mode=database_dynamodb_billing_mode
        )
        api = API(
            self,
            "API",
            dynamodb_table_name=database.dynamodb_table.table_name,
            lambda_reserved_concurrency=api_lambda_reserved_concurrency,
        )
        Monitoring(self, "Monitoring", database=database, api=api)
        
        database.dynamodb_table.grant_read_write_data(api.lambda_function)
	
        self.endpoint = api.api_gateway_http_api.url
```

**toolchain.py**

The only change here is to use `UserManagement` class representing the workload.

```python
# Continuous deployment pipeline
...
import aws_cdk as cdk
from aws_cdk import pipelines

import constants
from usermanagement.workload import UserManagement

class Toolchain(cdk.Stack):
    def __init__(self, scope: cdk.Construct, id_: str, **kwargs: Any):
        super().__init__(scope, id_, **kwargs)
        ...
        pipeline = pipelines.CodePipeline(...)
        Toolchain._add_production_stage(codepipeline)
    ...
    @staticmethod
    def _add_production_stage(self, pipeline: pipelines.CodePipeline) -> None:
        production = cdk.Stage(
            pipeline,
            PRODUCTION_ENV_NAME,
            env=cdk.Environment(
                account=PRODUCTION_ENV_ACCOUNT, region=PRODUCTION_ENV_REGION
            ),
        )
        usermanagement = UserManagement(
            production,
            constants.APP_NAME + PRODUCTION_ENV_NAME,
            api_lambda_reserved_concurrency=10,
            database_dynamodb_billing_mode=dynamodb.BillingMode.PROVISIONED,
            stack_name=constants.APP_NAME + PRODUCTION_ENV_NAME,
        )
        ...
        pipeline.add_stage(production, post=[smoke_test])
```

## Each component as one or more stacks

**app.py**

I still import and use the workload class here. You can notice that I specify a new `stack_namespace` parameter instead of ID parameter. The next `usermanagement/workload.py` section dives into that.

```python
# The AWS CDK application entry point
...
import constants
from usermanagement.workload import UserManagement
from toolchain import Toolchain

app = cdk.App()

# Workload sandbox stack
UserManagement(
    app,
    stack_namespace=constants.APP_NAME + "Sandbox",
    api_lambda_reserved_concurrency=1,
    database_dynamodb_billing_mode=dynamodb.BillingMode.PAY_PER_REQUEST,
    env=cdk.Environment(
        account=os.environ["CDK_DEFAULT_ACCOUNT"],
        region=os.environ["CDK_DEFAULT_REGION"],
    ),
)

# Toolchain stack (defines the continuous deployment pipeline)
Toolchain(
    app,
    constants.APP_NAME + "Toolchain",
    env=cdk.Environment(account="111111111111", region="eu-west-1"),
)

app.synth()
```

**usermanagement/workload.py**

The `UserManagement` class implements a workload container object (**not a stack**) that composes the components together. It passes the `scope` to each component stack. The new `stack_namespace` parameter sets prefix for component stacks ID and name. Now, I set the stack name in `UserManagement` class and not in the toolchain. This is because the toolchain can't pass a single stack name as it did in the single stack option. Hence, the `UserManagement` container object should assume that responsiblity based on the `stack_namespace` parameter.

Since the `UserManagement` class is not a stack, it doesn't (and can't) define stack outputs. The component stacks define their own outputs. I can still define dependencies between the stacks here if needed (explicitly or implicitly).

```python
# User Management workload
...
from usermanagement.chatbot_api.component import ChatbotAPI
from usermanagement.users_api.component import UsersAPI
from usermanagement.users_ui.component import UsersUI

class UserManagement:
    def __init__(
        self,
        scope: cdk.Construct,
        *,
        stack_namespace: str,
        api_lambda_reserved_concurrency: int,
        database_dynamodb_billing_mode: dynamodb.BillingMode,
        **kwargs: Any,
    ):
        chatbot_api = ChatbotAPI(
            scope,
            f"{stack_namespace}-ChatbotAPI",
            stack_name=f"{stack_namespace}-ChatbotAPI",
            **kwargs,
        )
        users_api = UsersAPI(
            scope,
            f"{stack_namespace}-UsersAPI",
            api_lambda_reserved_concurrency=1,
            database_dynamodb_billing_mode=dynamodb.BillingMode.PAY_PER_REQUEST,
            stack_name=f"{stack_namespace}-UsersAPI",
            **kwargs,
        )
        users_ui = UsersUI(
            scope,
            f"{stack_namespace}-UsersUI",
            chatbot_endpoint=chatbot_api.endpoint,
            users_api_endpoint=users_api.endpoint,
            stack_name=f"{stack_namespace}-UsersUI",
            **kwargs,
        )
```

**usermanagement/users_api/component.py**

The implementation here is similar to the original post. The only differences are the imports namespace that starts on the workload level and the stack output attribute name.

```python
# Users API component deployment layout
...
from usermanagement.users_api.api.infrastructure import API
from usermanagement.users_api.database.infrastructure import Database
from usermanagement.users_api.monitoring.infrastructure import Monitoring

class UsersAPI(cdk.Stack):
    def __init__(
        self,
        scope: cdk.Construct,
        id_: str,
        *,
        api_lambda_reserved_concurrency: int,
        database_dynamodb_billing_mode: dynamodb.BillingMode,
        **kwargs: Any,
    ):
        super().__init__(scope, id_, **kwargs: Any,)
		
        database = Database(
            self, 
            "Database", 
            dynamodb_billing_mode=database_dynamodb_billing_mode
        )
        api = API(
            self,
            "API",
            dynamodb_table_name=database.dynamodb_table.table_name,
            lambda_reserved_concurrency=api_lambda_reserved_concurrency,
        )
        Monitoring(self, "Monitoring", database=database, api=api)
        
        database.dynamodb_table.grant_read_write_data(api.lambda_function)
	
        self.endpoint = cdk.CfnOutput(
            self, 
            "Endpoint", 
            value=api.api_gateway_http_api.url,
        )
```

**toolchain.py**

Since `UserManagement` is a container object, this time the toolchain passes only the `stack_namespace` parameter. The toolchain doesn't pass stack ID and name parameters, because `UserManagement` defines them.

```python
# Continuous deployment pipeline
...
import aws_cdk as cdk
from aws_cdk import pipelines

import constants
from usermanagement.workload import UserManagement

class Toolchain(cdk.Stack):
    def __init__(self, scope: cdk.Construct, id_: str, **kwargs: Any):
        super().__init__(scope, id_, **kwargs)
        ...
        pipeline = pipelines.CodePipeline(...)
        Toolchain._add_production_stage(codepipeline)
    ...
    @staticmethod
    def _add_production_stage(self, pipeline: pipelines.CodePipeline) -> None:
        production = cdk.Stage(
            pipeline,
            PRODUCTION_ENV_NAME,
            env=cdk.Environment(
                account=PRODUCTION_ENV_ACCOUNT, region=PRODUCTION_ENV_REGION
            ),
        )
        usermanagement = UserManagement(
            production,
            stack_namespace=constants.APP_NAME + PRODUCTION_ENV_NAME,
            api_lambda_reserved_concurrency=10,
            database_dynamodb_billing_mode=dynamodb.BillingMode.PROVISIONED,
        )
        ...
        pipeline.add_stage(production, post=[smoke_test])
```

## Conclusion
In this post, I described the recommended project structure for monolithic applications. The options covered range from deploying the application as a single stack to using a stack per component. You can also create more than one stack in a component if needed. I used a single pipeline for the application. If you would like to create more than one pipeline (e.g. pipeline per component), look at [Integrate GitHub monorepo with AWS CodePipeline to run project-specific CI/CD pipelines](https://aws.amazon.com/blogs/devops/integrate-github-monorepo-with-aws-codepipeline-to-run-project-specific-ci-cd-pipelines/) blog post.

If you think I’ve missed something, feel free to ping me [@alex_pulver](https://twitter.com/alex_pulver). Happy coding!