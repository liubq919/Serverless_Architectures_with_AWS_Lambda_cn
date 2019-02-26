### 无服务器架构的最佳实践

AWS Well-Architected Framework包含的策略可帮助你将工作量与我们的最佳实践进行比较，并获得生成稳定高效系统的指导，以便你可以专注于功能需求。它基于五大支柱:安全性、可靠性、性能效率、成本优化和卓越运营。框架中的许多准则适用于无服务器应用。但是，有一些特定的实现步骤或模式对于无服务器架构是唯一的。在以下部分中，我们将针对每个Well-Architected支柱介绍一组无服务器的专用推荐。

#### 安全最佳实践

在应用中设计和实现安全性应始终是第一优先级 - 这不会因无服务器架构而改变。与服务器托管的应用相比，保护无服务器应用的主要区别显而易见 - 没有服务器可供保护。但是，你仍需要考虑应用程序的安全性。无服务器安全性仍然存在责任共担模型。

使用Lambda和无服务器架构，而不是通过防病毒/恶意软件，文件完整性监控，入侵检测/防御系统，防火墙等实现应用安全性，你可以通过编写安全的应用代码，对源代码更改进行严格的访问控制来确保实现安全性最佳实践，并对与Lambda函数集成的每个服务遵循AWS安全最佳实践。

下面是应该应用于许多无服务器用例的无服务器安全性最佳实践的简要列表，尽管你自己的特定安全性和合规要求应该得到很好的理解，并且可能包括比我们在这里描述的更多的内容。
- **每个函数一个IAM角色**
   您的AWS账户中的每个Lambda函数都应与IAM角色具有1：1的关系。即使多个函数以完全相同的策略开始，也要始终解耦IAM角色，以便可以确保在将来为函数使用最少的权限策略。
   例如，如果你共享Lambda函数的IAM角色，该角色需要跨多个Lambda函数访问AWS KMS密钥，那么所有这些函数现在都可以访问相同的加密密钥。
- **临时AWS凭证**
  Lambda函数代码或配置中不应该包含任何长期使用的AWS凭据。（这对于静态代码分析工具来说非常有用，可以确保它永远不会出现在您的代码库中！）对于大多数情况，IAM执行角色是与其他AWS服务集成所需的全部内容。对于大多数情况，IAM执行角色是与其他AWS服务集成所需的全部内容。只需通过AWS SDK在代码中创建AWS服务客户端，而无需提供任何凭证。为角色生成的临时凭证，SDK会自动管理其检索和轮换。以下是使用Java的示例。

    *AmazonDynamoDB client = AmazonDynamoDBClientBuilder.defaultClient();*
    *Table myTable = new Table(client, "MyTable");*

    此代码片段是AWS SDK for Java创建对象以与DynamoDB表进行交互所需的全部内容，该表使用分配给函数的临时IAM凭据自动将其请求签名到DynamoDB API。

    但是，在某些情况下，执行角色没有足够多的函数所需的访问类型。此情况可能是由于Lambda函数执行了一些跨账户集成，或者你通过联合[Amazon Cognito](https://aws.amazon.com/cn/cognito/)认证角色与[DynamoDB细粒度访问控制](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/specifying-conditions.html)以实现特定用户的访问控制策略。对于跨账户用例，你应该授权执行角色内授予访问[AWS Security Token Service](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html)中的AssumeRole API，并将其集成以检索临时访问凭证。

    对于特定用户的访问控制策略，应该向函数提供用户身份识别，然后将其与Amazon Cognito API [GetCredentialsForIdentity](https://docs.aws.amazon.com/cognitoidentity/latest/APIReference/API_GetCredentialsForIdentity.html)集成。在这种情况下，你必须确保代码正确地管理这些凭证，以便为与Lambda函数调用相关的每个用户利用正确的凭证。对于应用程序来说，将这些每个用户的凭据加密并存储在DynamoDB或Amazon ElastiCache这样的地方作为用户会话数据的一部分是很常见的，这样，与返回用户的后续请求重新生成凭据相比，可以以更低的延迟和更灵活的伸缩性检索凭据。
- **持久化密码**
  在某些情况下，你可能拥有Lambda函数需要使用的长期密码（例如，数据库凭据，依赖服务访问密钥，加密密钥等）。我们为应用中的密码生命周管理期推荐了一些选项：
  - [使用Encryption Helpers（加密助手）的Lambda环境变量](https://docs.aws.amazon.com/lambda/latest/dg/env_variables.html#env_encrypt)
      **优势** - 提供直接访问函运行时环境，最大限度地减少了检索密码所需的延迟和代码。
      **劣势** - 环境变量与函数版本耦合。更新环境变量需要新的函数版本（更严格的是，但也提供稳定版本历史记录）。
  -  [Amazon EC2 Systems Manager Parameter Store46](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html)
      **优势** - 与Lambda函数完全解耦，在密码和函数之间提供了最大的灵活性
      **劣势** - 检索参数/密码需要向Parameter Store发送请求。虽然不会实质影响，但这确实会增加环境变量的延迟以及额外的服务依赖性，并且需要编写稍微多一点的代码。
- **使用密码**
  密码应用始终只能存在内存中，永远不能被记录或者写到磁盘上。在应用保持运行的同时，在需要撤消密码的情况下，编写管理密码轮换的代码。
- **API授权**
  使用API网关作为Lambda函数的事件源相对于其他AWS服务事件源选项是独特的，因为你拥有API客户端认证与授权的所有权。API Gateway可以通过提供原生[AWS SigV4身份验证](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)，[生成的客户端SDK](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-generate-sdk.html)和[自定义授权程序](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)等功能来执行大量繁重工作。但是，你仍然有责任确保API的安全状态符合设置的标准。有关API安全性最佳实践的详细信息，请参阅[此文档](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html)。
- **VPC Security**
  如果Lambda函数需要访问部署在VPC中的资源，那么应该通过使用最少权限安全组、特定于Lambda函数的子网、网络ACL和路由表来实现网络安全最佳实践，这些表仅允许来自Lambda函数的流量到达预期目的地。
  请记住，这些实践和策略会影响Lambda函数连接到所需依赖的方式。调用Lambda函数仍然通过事件源和Invoke API进行(两者都不受VPC配置的影响)。
- **部署访问控制**
  对UpdateFunctionCode API的调用类似于代码部署。通过UpdateAlias API将别名移动到新发布的版本类似于代码发布。以极高的敏感性对待能开启函数代码/别名的Lambda api的访问。因此，你应该为任何函数(至少是生产函数)消除用户对这些api的直接访问，以消除人为错误的可能性。应通过自动化实现对Lambda函数的代码更改。记住这一点，部署到Lambda的入口点将成为启动持续集成/持续推送管道的地方。这可能是仓库中的发布分支，S3 bucket，其中上传了一个新的代码包，触发[AWS CodePipeline](https://aws.amazon.com/cn/codepipeline/)管道，或者是某个特定于组织和流程的仓库。无论它在哪里，它都成为一个新的地方，在这里你应该执行严格的访问控制机制，以适合团队结构和角色。
