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