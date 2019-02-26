#### 卓越运维最佳实践

创建无服务器应用程序消除了传统应用程序带来的许多运维负担。这并不意味着你应该减少对卓越运维的关注。这意味着你可以将操作重点缩小到更少的责任，并有望实现更高级别的卓越运维。

##### 日志

Lambda的每种语言运行时都为函数提供了一种机制，可以将日志语句交付给CloudWatch Logs。对于Lambda和无服务器架构来说，充分使用日志并不是什么新鲜事。尽管今天并不认为这是最佳实践，但是许多运维团队依赖于查看日志，因为日志是在部署应用的服务器上生成的。很显然在Lambda中是不可能的，因为没有服务器。你现在也无法“逐步”执行实时运行的Lambda函数代码（尽管可以在部署之前使用[AWS SAM Local](https://github.com/awslabs/aws-sam-local)执行此操作）。对于已部署的功能，你在很大程度上依赖于创建的日志来指引函数行为的调查。 因此，特别重要的是，创建的日志能够找到详细程度的平衡，以帮助跟踪/分类问题，而不需要太多额外的计算时间来创建它们。

我们建议你使用Lambda环境变量来创建一个函数可以引用的日志级变量，这样它就可以确定在运行时创建哪些日志语句。适当地使用日志级别可以确保你能够仅在运维分类期间选择性地产生额外的计算成本和存储成本。