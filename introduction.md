## 介绍 - 什么是无服务器？

无服务器通常指无服务器应用。无服务器应用不需要你提供或管理任何服务器。你可以专注于核心产品和业务逻辑，而不是诸如操作系统（OS）访问控制，操作系统修补，供给，合适大小，扩展和可用性等职责。通过在无服务器平台上构建应用，平台可以为你管理这些职责。

对于被视为无服务器的服务或平台，它应提供以下功能：
- 无服务器管理 - 不必提供或维护任何服务器。没有要安装、维护或管理的软件或运行时。
- 灵活的伸缩 - 可以自动伸缩应用，或者通过切换消费单元(例如吞吐量、内存)而不是单独服务器的单元来调整其容量。
- 高可用 - 无服务器应用具有内置的可用性和容错能力。
- 无闲置容量 - 你无需为闲置容量付费，也没有必要为计算和存储之类的设备预先准备或过度准备容量。只要代码不运行就无需付费。

AWS Cloud提供许多不同的服务，这些服务可以是无服务器应用的组件。这些服务的功能包括：
- 计算 - [AWS Lambda](https://aws.amazon.com/lambda)
- APIs - [Amazon API Gateway](https://aws.amazon.com/api-gateway/)
- 存储 - [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/)
- 数据库 - [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)
- 进程间通信 - [ Amazon Simple Notification Service (Amazon SNS)](https://aws.amazon.com/sns/)和[Amazon Simple Queue Service (Amazon SQS)](https://aws.amazon.com/sqs/)
- 编排 - [AWS Step Functions](https://aws.amazon.com/step-functions/) and [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html)
- 分析 - [Amazon Kinesis](https://aws.amazon.com/kinesis/)

这篇白皮书将重点聚焦AWS Lambda，即执行无服务器应用代码的的计算层，以及在使用Lambda构建和维护无服务器应用时支持最佳实践的AWS开发人员工具和服务。
