#### 持续推送

我们建议您通过CI/CD管道以编程方式管理所有的无服务器部署。这是因为使用Lambda开发新功能和推送代码更改的速度将允许更频繁地部署。手动部署与更频繁部署的需求相结合，常常会导致手动流程成为瓶颈并容易出错。

AWS CodeCommit，AWS CodePipeline，AWS CodeBuild，AWS SAM和AWS CodeStar提供的功能提供了一组特性，你可以将这些特性原生地组合到一个整体和自动化的无服务器CI/CD管道中（管道本身也没有你需要管理的基础设施）。

下面是这些服务在定义良好的持续交付策略中如何发挥作用。

**AWS CodeCommit** - 提供托管的私有Git存储库，使你能够托管无服务器的源代码，创建符合我们建议的分支策略(包括细粒度访问控制)，并与AWS CodePipeline集成，以便在发布分支中发生新的提交时触发新的管道执行。

**AWS CodePipeline** - 定义管道中的步骤。通常，AWS代码管道从源代码更改到达的地方开始。然后执行构建阶段，对新构建执行测试，并将构建部署和发布到现实环境中。AWS CodePipeline为每个阶段提供了与其他AWS服务的原生集成选项。

**AWS CodeBuild** - 可以用于管道的构建状态。使用它来构建代码、执行单元测试和创建新的Lambda代码包。然后，与AWS SAM集成，将代码包推送到Amazon S3，并通过AWS CloudFormation将新包推送到Lambda。
在新版本通过AWS CodeBuild发布到Lambda函数之后，你可以通过创建以部署为中心的Lambda函数来自动化AWS CodePipeline管道中的后续步骤。它们将拥有执行集成测试、更新函数别名、确定是否需要立即回滚以及在应用部署期间需要发生的任何其他以应用为中心的步骤(如缓存刷新、通知消息等)的逻辑。这些以部署为中心的Lambda函数中的每一个都可以被逐次调用，作为使用Invoke动作的AWS CodePipeline管道中的一个步骤。有关在AWS CodePipeline中使用Lambda的详细信息，请参阅[此文档](https://docs.aws.amazon.com/zh_cn/codepipeline/latest/userguide/actions-invoke-lambda-function.html)。

最后，每个应用和组织对于将源代码从仓库迁移到生产环境都有自己的需求。在此过程中引入的自动化程度越高，使用Lambda实现的灵活性就越高

**AWS CodeStar** - 用于创建无服务器应用(和其他类型的应用)的统一用户界面，帮助你从头开始遵循这些最佳实践。在AWS CodeStar中创建新项目时，你将自动开始使用完全实现且集成的持续推送工具链（使用前面提到的AWS CodeCommit，AWS CodePipeline和AWS CodeBuild服务）。你还可以在其中管理项目SDLC的所有方面，包括团队成员管理、问题跟踪、开发、部署和操作。有关AWS CodeStar的更多信息，请访问[这里](https://aws.amazon.com/codestar/)。

