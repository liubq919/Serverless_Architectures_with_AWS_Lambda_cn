## AWS Lambda — 深入研究

本白皮书的其余部分将帮助你理解Lambda的组件和特性，以及在使用Lambda构建和拥有无服务器应用时的各方面的最佳实践。

让我们从进一步扩展和解释在介绍中描述的Lambda的每个主要组件开始深入研究:函数代码、事件源和函数配置。

### Lambda函数代码

其核心是使用Lambda来执行代码。这可以用Lambda (Java, Node)支持的任何语言编写的代码。以及在编写代码时上传的任何代码或包。你可以自由地将任何库、构件（artifacts）或已编译的本地二进制文件作为函数代码包的一部分在运行时环境上执行。如果愿意，你甚至可以执行用另一种编程语言(PHP、Go、SmallTalk、Ruby等)编写的代码，只要在AWS Lambda运行时环境所支持的一种语言中上传然后调用该代码（[请查看教程](https://aws.amazon.com/cn/blogs/compute/scripting-languages-for-aws-lambda-running-php-ruby-and-go/)）。

Lambda运行时环境基于Amazon Linux AMI(请参阅[此处](https://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html)的当前环境细节)，因此你应该在匹配的环境中编译和测试计划在Lambda内部运行的组件。为了帮助你在Lambda内部运行之前执行这种类型的测试，AWS提供了一组名为[AWS SAM Local](https://github.com/awslabs/aws-sam-cli)的工具来支持Lambda函数的本地测试。我们将在本文的无服务器开发最佳实践部分讨论这些工具。

#### 函数代码包
功能代码**包**包含你希望在执行代码时在本地可用的所有组件。一个包将至少包括函数在被调用时你希望Lambda服务执行的代码函数。但是，它也可能包含代码在执行时引用的其它组件，例如，代码将导入的其他文件、类和库、希望执行的二进制文件或代码在被调用时引用的配置文件。在发布时，函数代码包的最大压缩大小为50M，最大解压大小为250M。（有关AWS Lambda限制的完整列表，请参阅[此文档](https://docs.aws.amazon.com/lambda/latest/dg/limits.html)）

当创建Lambda函数时(通过AWS管理控制台，或者使用CreateFunction API)，可以引用上传包的S3 bucket和对象key。或者，在创建函数时直接上传代码包。Lambda将代码包存储在由该服务管理的S3 bucket中。当你将更新的代码发布到现有的Lambda函数时，也可以使用相同的选项（通过[UpdateFunctionCode](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/API_UpdateFunctionCode.html) API）。

当事件发生时，你的代码包将从S3 bucket下载，安装在Lambda运行时环境中，并根据需要进行调用。在Lambda管理的环境中，按照触发函数事件数量所需的规模，按需进行伸缩。

#### 处理程序

当Lambda函数被调用时，代码执行从所谓的**处理程序**开始。处理程序是你创建并包含在程序包中的特定代码方法（Java，C＃）或函数（Node.js，Python）。在创建Lambda函数时指定处理程序。Lambda支持的每种语言对于如何在包中定义和引用函数处理程序都有自己的要求。

下面的链接将帮助你开始使用每种被支持的语言。

| 语言 | 定义处理程序示例  |
| ------ | --------- |
|  [Java](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/java-programming-model.html)  | `MyOutput output handlerName(MyEvent event, Context context) {` |
|        | ` ...` |
|        | ` }` |
|  [Node.js](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/programming-model.html)  | `exports.handlerName = function(event, context, callback) {` |
|        | ` ...` |
|        | ` // callback parameter is optional` |
|        | ` }` |
|  [Python](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/python-programming-model.html)  | `def handler_name(event, context):` |
|        | ` ...` |
|        | `return some_value` |
|  [C#](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/dotnet-programming-model.html)  | `myOutput HandlerName(MyEvent event, ILambdaContext context) {` |
|        | ` ...` |
|        | ` }` |

一旦在Lambda函数中成功调用处理程序，运行时环境就属于你编写的代码。Lambda函数可以自由执行你认为合适的任何逻辑，由你在处理程序中开始编写的代码驱动。这意味着处理程序可以调用你所上传的文件和类中的其他方法和函数。代码可以导入上传的第三方库，并安装和执行上传的本地二进制文件(只要它们能够在Amazon Linux上运行)。它还可以与其他AWS服务交互，或者向它所依赖的web服务发出API请求，等等。

#### 事件对象

以一种被支持的语言编写Lambda函数且当其被调用时，提供给处理程序函数的参数之一是**事件对象**。事件的结构和内容不同，具体取决于创建它的事件源。事件参数的内容包括Lambda函数驱动其逻辑所需的所有数据和元数据。例如，由API网关创建的事件将包含与API客户端发出的HTTPS请求相关的详细信息（例如，路径，查询字符串，请求体），而一个由Amazon S3创建新对象时所引起的事件将包括bucket和新对象的详细信息。

#### 上下文对象

Lambda函数还被提供了一个**上下文对象**。下文对象允许函数代码与Lambda执行环境进行交互。根据Lambda函数使用的语言运行时，上下文对象的内容和结构会有所不同，但至少它将包含：
- AWS RequestId - 用于跟踪Lambda函数的特定调用（对于错误报告或与联系AWS Support时非常重要）
- 剩余时间 - 函数超时发生前剩余的毫秒数（Lambda函数在发布时最多可以运行300秒，但是可以配置更短的超时）
- 日志 - 每种语言运行时都提供了将日志语句输出到Amazon CloudWatch日志的功能。上下文对象包含有关将日志语句发送到哪个CloudWatch Logs流的信息。在每一种语言运行时日志是如何被处理的更多信息，请参见以下内容：
  1. [Java](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/java-logging.html)
  2. [Node.js](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/nodejs-prog-model-logging.html)
  3. [Python](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/python-logging.html)
  4. [C#](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/dotnet-logging.html)

#### 编写AWS Lambda代码 - 无状态与复用

在为Lambda编写代码时，理解中央租户非常重要：**你的代码不能对状态进行假设**。这是因为当函数容器第一次被创建并被调用时，Lambda全托管服务。出于多种原因，容器可以第一次被调用。比如，触发Lambda函数的事件的并发量正在增加，超出了之前为函数创建的容器数量，事件在几分钟内第一次触发Lambda函数，等等。Lambda负责弹性伸缩函数容器以满足实际需求，而代码需要能够相应地进行操作。尽管Lambda不会中断正在运行的特定调用的处理，但是你的代码不需要考虑这种级别的波动性。

这意味着你的代码无法做出任何假设，即状态将从一次调用保留到下一次。但是，函数容器每次被创建和调用时，它都会保持活动状态，并且在终止之前至少有几分钟时间可用于后续调用。当之前已经被激活并至少被调用过一次的容器上发生后续调用时，我们说调用是在一个**已预热容器**上运行的。当Lambda函数发生调用时，需要首次创建和调用函数代码包，我们说调用正在经历一个**冷启动**。

![3](images/Figure3.jpg)

根据代码执行的逻辑，了解代码如何利用已预热容器的优势可以在Lambda中实现更快的代码执行。反过来，这会导致更快的响应和更低的成本。有关如何利用已预热容器提高Lambda函数性能的更多详细信息和示例，请参阅本白皮书后续的“最佳实践”部分。

总的来说，Lambda支持的每种语言都有自己的打包源代码模型和优化可能性。访问[此页面](https://docs.aws.amazon.com/lambda/latest/dg/programming-model-v2.html)以开始使用每种支持的语言。

### Lambda函数事件源

现在你已经知道Lambda函数代码的组成部分，接下来让我们看看调用代码的**事件源**或触发器。虽然Lambda提供了调用API以能够直接调用函数，但你可能只会将其用于测试和运维。相反，你可以将Lambda函数与AWS服务中发生的事件源关联起来，这些事件源将根据需要调用函数。你不必编写、伸缩或维护将事件源与Lambda函数集成在一起的任何软件。

#### 调用模式
调用Lambda函数有两种模型：
- 推送模式 - 在另一个AWS服务中每次发生特定事件时，都会调用Lambda函数（比如，新对象被添加到S3 bucket）
- 拉取模式 - Lambda轮询数据源并使用到达数据源的任何新记录调用你的函数，在单个函数调用中将新记录批处理（比如，Amazon Kinesis或者Amazon DynamoDB流中的新纪录）

此外，Lambda函数可以同步或异步执行。触发Lambda函数时，你可以使用提供的参数**调用类型（InvocationType）**以选择执行类型，这个参数有三个可能的值：
- 请求响应 - 同步执行。
- 事件 - 同步执行。
- 排练（DryRun）- 测试调用方是否允许调用，但不要执行该函数。

每个事件源指定如何调用函数。事件源还负责生成自己的事件参数，如前所述。

下面的表提供了一些更流行的事件源如何与Lambda函数集成的详细信息。你可以在[这里](http://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html)找到支持的事件源的完整列表。

#### 推送模式事件源

##### Amazon S3

|  |  |
| ------ | --------- |
|  调用模式  | 推送 |
|  调用类型  | 事件 |
|  说明  | S3事件通知(如ObjectCreated和ObjectRemoved)在发布时可以将其配置成调用Lambda函数|
|  用例案例  | 为用户通过应用程序上传到S3 bucket的图像创建图像修改（缩略图，不同分辨率，水印等）|
| | 处理上传到S3 bucket的原始数据，并将转换后的数据作为大数据管道的一部分移动到另一个S3 bucket|

##### Amazon API Gateway

|  |  |
| ------ | --------- |
|  调用模式  | 推送 |
|  调用类型  | 事件或请求响应 |
|  说明  | 使用API Gateway创建的API方法可以使用Lambda函数作为其服务后端。如果选择Lambda作为API方法的集成类型，则会同步调用Lambda函数（Lambda函数的响应用作API响应）。使用此集成类型，API Gateway还可以充当Lambda函数的简单代理。 API网关不会自行执行任何处理或转换，并将请求的所有内容传递给Lambda。|
| | 如果你希望API作为事件异步调用函数并立即返回空响应，则可以将API Gateway用作AWS服务代理(AWS Service Proxy)，并与Lambda调用API(Lambda Invoke API）集成，在请求标头中提供事件调用类型（Event InvocationType)。如果API客户端不需要基于请求返回任何信息并且希望获得尽可能快地响应时间，那么这是一个很好的选择。（此选项非常适合将用户在网站或应用上的交互推送到服务后端进行分析。）|
|  用例案例  | Web服务后端（Web应用，移动应用，微服务架构等）|
| | 遗留服务集成(Lambda函数，用于将遗留SOAP后端转换为新的现代REST API)。|
| | 任何其他用例，将HTTPS作为应用程序组件之间的恰当集成机制|

##### Amazon SNS

|  |  |
| ------ | --------- |
|  调用模式  | 推送 |
|  调用类型  | 事件 |
|  说明  | 发布到SNS主题的消息可以作为事件传递到Lambda函数。|
|  用例案例  | 自动响应CloudWatch告警。|
| | 处理来至于可以原生推送消息到SNS主题的其他服务（AWS或其他）的事件。|

##### AWS CloudFormation

|  |  |
| ------ | --------- |
|  调用模式  | 推送 |
|  调用类型  | 请求响应 |
|  说明  | 作为部署AWS CloudFormation堆栈的一部分，可以将Lambda函数指定为自定义资源，以执行任何自定义命令并将数据提供回正在进行的堆栈创建。|
|  用例案例  | 扩展AWS CloudFormation功能以囊括AWS CloudFormation本身尚未支持的AWS服务特性。|
| | 在堆栈创建/更新/删除过程的关键阶段执行自定义验证或报告。|

##### Amazon CloudWatch Events

|  |  |
| ------ | --------- |
|  调用模式  | 推送 |
|  调用类型  | 事件 |
|  说明  | 许多AWS服务将资源状态更改发布到CloudWatch Events。然后可以过滤这些事件并将其路由到Lambda函数以进行自动响应。|
|  用例案例  | 事件驱动的操作自动化（例如，每次启动新的EC2实例时执行操作，当AWS Trusted Advisor报告新的状态更改时通知适当的邮件列表）。|
| | 替换先前使用cron完成的任务（CloudWatch Events支持已预定事件）。|

##### Amazon Alexa

|  |  |
| ------ | --------- |
|  调用模式  | 推送 |
|  调用类型  | 请求响应 |
|  说明  | 你可以编写Lambda函数作为Amazon Alexa Skills的服务后端。当一个Alexa用户与你的技能互动，Alexa的自然语言理解和处理能力将传递他们的互动到你的Lambda函数。|
|  用例案例  | 你自己的Alexa技能。|

#### 拉取模式事件源

##### Amazon DynamoDB

|  |  |
| ------ | --------- |
|  调用模式  | 拉取 |
|  调用类型  | 请求/响应 |
|  说明  | Lambda将每秒多次轮询DynamoDB流，并使用自上次批处理以来已发布到流的更新批来调用Lambda函数。你可以配置每个调用的批处理大小。|
|  用例案例  | 以应用为中心的工作流，应该在DynamoDB表中发生更改时触发(例如，新用户注册、下单、接受好友请求等)。|
| | 将DynamoDB表复制到另一个区域(用于灾难恢复)或另一个服务(作为日志发送到S3 bucket进行备份或分析)。|

##### Amazon Kinesis Streams

|  |  |
| ------ | --------- |
|  调用模式  | 拉取 |
|  调用类型  | 请求/响应 |
|  说明  | Lambda轮询Kinesis流，每一个流分片一秒一次，并使用分片中的下一条记录调用Lambda函数。你可以为每次交付给函数的记录数量以及并发执行的Lambda函数容器的数量(流分片的数量=并发函数容器的数量)定义批大小。|
|  用例案例  | 大数据管道实时数据处理。|
| | 流日志语句或其它应用程序事件的实时告警/监控。|

### Lambda函数配置

在编写和打包Lambda函数代码之后，除了选择触发函数的事件源之外，还需要设置各种配置项，以定义在Lambda中如何执行代码。

##### 函数内存

要定义分配给执行的Lambda函数的资源，你将获得一个单刻度表盘以增加/减少函数资源的：内存/RAM。可以为Lambda函数分配128MB到1.5GB的RAM。这不仅决定了函数代码在执行过程中可用的内存数量，而且同一个刻度还会影响函数可用的CPU和网络资源。

在优化任何Lambda函数的代价和性能时，选择适当的内存分配是非常重要的一步。请查看本白皮书后续的最佳实践，了解有关优化性能的更多细节。

##### 版本和别名

有时你可能需要引用或将Lambda函数恢复到以前部署的代码。Lambda可以让你**版本化**AWS Lambda函数。每个Lambda函数都有一个内置的默认版本: $LATEST。你可以通过 $LATEST 版本以使用最新上传到Lambda函数的代码。你可以对当前通过 $LATEST引用的代码执行快照，并通过[PublishVersion API](https://docs.aws.amazon.com/lambda/latest/dg/API_PublishVersion.html)创建版本编号。此外，通过[UpdateFunctionCode API](http://docs.aws.amazon.com/lambda/latest/dg/API_UpdateFunctionCode.html)更新函数代码时，还有一个可选的Boolean参数publish。通过在请求中设置publish：true，Lambda将创建一个新的Lambda函数版本，从上一个发布的版本开始递增。

你可以在任何时候独立地调用每个版本的Lambda函数。每个版本都有自己的 Amazon Resource Name(ARN)，如下所示：

    arn:aws:lambda:[region]:[account]:function:[fn_name]:[version]

在调用[Invoke API](https://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html)或为Lambda函数创建事件源时，还可以指定要执行的Lambda函数的特定版本。如果不提供版本号，或者使用不包含版本号的ARN，则默认情况下调用$LATEST。

知道Lambda函数容器特定于函数的某种版本这一点非常重要。因此，例如，如果在Lambda运行时环境中函数版本5的多个函数容器已经部署并可用，则相同函数的版本6将无法在现有版本5的容器之上执行 - 将为每个函数版本安装和管理一组不同的容器。

在测试和运维活动期间，通过版本号调用Lambda函数非常有用。但是，在实际应用流量中，我们不建议你通过指定版本的方式触发Lambda函数。这样做需要你在每次想要更新代码时更新所有调用Lambda函数的触发器和客户端，以指向一个新的函数版本。这里应该使用Lambda**别名**。函数别名允许调用事件源并将其指向特定的Lambda函数版本。

但是，你可以随时更新别名引用的版本。例如，通过别名live调用版本5的事件源和客户端可能会在将live别名更新为指向版本6时切换到函数的版本6。可以在ARN中引用每个别名，类似于引用函数版本号：

    arn:aws:lambda:[region]:[account]:function:[fn_name]:[alias]

以下是Lambda别名的一些示例建议以及如何使用它们：
- live/prod/active - 这可能代表生产触发器或客户端正在集成的Lambda函数版本。
- blue/green – 通过使用别名启用蓝/绿部署模式。
- debug - 如果你已经创建了一个测试堆栈来调试应用，那么当你需要执行更深入的分析时，它可以与这样的别名集成。

为函数别名的使用创建一个良好的、文档化的策略使你能够进行复杂的无服务器部署和操作实践。

##### IAM 角色

AWS身份和访问管理（IAM）提供了创建[IAM策略](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)的功能，这些策略定义了与AWS服务和API交互的权限。策略可以与**IAM角色**相关联。为特定角色生成的任何访问密钥ID和访问密钥都被授权执行附加到该角色的策略中所定义的操作。有关IAM最佳实践的更多信息，请参阅[本文档](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)。

在Lambda的上下文中，你为每个Lambda函数分配一个IAM角色（称为执行角色）。附加到该角色的IAM策略定义了函数代码被授权与哪些AWS服务API交互。有两个好处：
- 源代码不需要执行任何AWS凭据管理或轮换来与AWS API交互。只要使用AWS sdk和默认凭据提供程序，Lambda函数就会自动使用分配给该函数的执行角色相关联的临时凭据。
- 你的源代码与其自身的安全状态是解耦的。如果开发人员试图更改Lambda函数代码，以便与函数无法访问的服务集成，由于分配给函数的IAM角色，则该集成将失败。(除非他们使用了独立于执行角色的IAM凭证，否则您应该使用静态代码分析工具来确保您的源代码中不存在AWS凭证)。
  
为每个Lambda函数分配一个特定的、独立的和最少权限的IAM角色非常重要。这种策略确保每个Lambda函数都可以独立发展，而不需要增加任何其他Lambda函数的授权范围。

##### Lambda函数权限

你可以定义哪些推送模式事件源允许通过被称为**权限**的概念调用Lambda函数。使用权限，您可以声明**功能策略**，列出哪些AWS资源名称（ARN）允许调用函数。

对于拉取模型事件源（例如，Kinesis流和DynamoDB流），你需要确保分配给Lambda函数的IAM执行角色允许进行适当的操作。如果你不想管理所需的权限，AWS提供了一组与每个基于拉取事件源相关联的托管IAM角色。但是，为了确保最少权限IAM策略，你应该使用特定于资源的策略创建自己的IAM角色，以便仅允许访问预期的事件源。

##### 网络配置

通过使用AWS Lambda服务API的Invoke API执行Lambda函数；因此，没有直接的入站网络访问需要函数来管理。但是，函数代码可能需要与外部依赖项集成（内部或公共托管的Web服务，AWS服务，数据库等）。 Lambda函数有两个用于出站网络连接的广泛选项：
- 默认 - Lambda函数从Lambda管理的虚拟私有云(VPC)内部进行通信。它可以连接到internet，但不能连接到你自己的vpc中运行的任何私有部署的资源。
- VPC - Lambda函数通过弹性网络接口(ENI)进行通信，该接口在VPC中提供，你可以在自己的帐户中选择子网。可以为这些ENI分配安全组，并且流量将基于那些ENI所在的子网的路由表进行路由 - 就像将EC2实例放置在同一子网中一样。

如果Lambda函数不需要连接到任何私有部署的资源，我们建议选择默认网络选项。选择VPC选项需要你管理：
- 选择适当的子网以确保使用多个可用区以实现高可用性。
- 为每个子网分配适当数量的IP地址以容量管理。
- 实现VPC网络设计，允许Lambda函数具有所需的连接性和安全性。
- 如果Lambda函数调用模式需要及时创建新的ENI，则Lambda冷启动时间会增加。（当下创建ENI可能需要几秒钟。）

##### 环境变量

软件开发生命周期（SDLC）最佳实践指出开发人员分离他们的代码和配置。你可以通过将环境变量与Lambda一起使用来实现此目的。Lambda函数的环境变量使你能够动态地将数据传递到函数代码和库，而无需更改代码。环境变量是键-值对，你可以在函数配置中创建和修改它们。默认情况下，这些变量将在静止状态被加密。对于将存储为Lambda函数环境变量的任何敏感信息，我们建议在创建函数之前使用AWS Key Management Service（AWS KMS）加密这些值，并将加密的密文存储为变量值。然后让Lambda函数在执行时解密内存中的变量。

下面是一些你可能决定如何使用环境变量的示例：
- 日志设定（FATAL, ERROR, INFO, DEBUG, 等）
- 依赖关系和/或数据库连接字符串和凭据
- 功能标志和开关

Lambda函数的每个版本都可以拥有自己的环境变量值。然而，一旦为一个带编号的Lambda函数版本建立了值，它们就不能被更改。要更改Lambda函数环境变量，可以将它们更改为 $LATEST版本，然后发布包含新环境变量值的新版本。这使你可以始终跟踪哪些环境变量值与函数的先前版本相关联。这在回滚过程中或鉴别应用的过去状态时通常很重要。

##### 死信队列

即使在没有服务器的世界中，异常仍然可能发生。例如，(你可能已经上传了不允许成功解析Lambda事件的新函数代码，或者AWS中存在阻止调用该函数的操作事件)。对于异步事件源(InvocationType事件)，AWS拥有负责调用函数的客户端软件。AWS无法在调用发生时同步通知您调用是否成功。无论调用成功与否，AWS无法在调用发生时同步通知你。如果在这些模型中尝试调用函数时发生异常，则将再次尝试调用两次(在两次重试之间进行回退)。在第三次尝试之后，如果你为该函数配置了一个死信队列，那么该事件要么被丢弃，要么被放置到死信队列中。

SNS主题或SQS队列，是死信队列指定为所有失败调用事件的目的地。如果发生故障事件，使用死信队列只允许保留事件期间未能处理的消息。一旦函数能够再次被调用，就可以在死信队列中针对那些失败的事件进行重新处理。由你来决定重新处理/重试放置在死信队列中的函数调用尝试机制。有关死信队列的更多信息，[请参阅本教程](https://aws.amazon.com/cn/blogs/compute/robust-serverless-application-design-with-aws-lambda-dlq/)。如果对应用很重要的一点是，Lambda函数所有的调用最终都要完成，即使执行延迟，那么你应该使用死信队列。

### 超时

你可以指定在返回超时之前允许单个函数执行完成的最大时间。Lambda函数在发布时的最大超时为300秒，这意味着对Lambda函数的一次调用不能执行超过300秒。你不应该总是将Lambda函数的超时设置为最大值。在许多情况下，应用程序应该快速失败。因为Lambda函数是根据执行时间以100毫秒为增量计费的，所以避免函数冗长超时可以防止函数只是在等待超时被计费（可能是外部依赖不可用，你不小心编写了一个无限循环，或其他类似的情况）。

此外，一旦执行完成或Lambda函数发生超时并返回响应，所有执行都将停止。这包括Lambda函数在执行期间可能生成的任何后台进程，子进程或异步进程。因此，你不应该依赖后台或异步流程来进行关键活动。代码应确保在超时之前完成这些活动或从函数返回响应。