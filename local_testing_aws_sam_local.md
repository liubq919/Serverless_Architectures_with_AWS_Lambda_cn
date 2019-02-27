#### 本地测试 - AWS SAM Local

与AWS SAM一起，[AWS SAM Local](https://github.com/awslabs/aws-sam-cli)提供了其他命令行工具，可以将这些工具添加到AWS SAM，以便在将无服务器函数和应用部署到AWS之前对其进行本地测试。AWS SAM Local使用Docker使你能够使用常用事件源（例如，Amazon S3，DynamoDB等）快速测试开发的Lambda函数。在API网关中创建之前，可以在SAM模板中本地测试定义的API。你还可以验证创建的AWS SAM模板。通过启用这些功能，可以驻留在开发人员工作站中运行Lambda函数，你可以在本地执行诸如查看日志、在调试器中单步执行代码以及快速迭代更改，而无需将新代码包部署到AWS。