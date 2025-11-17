---
lab:
  title: 使用语义内核创建 AI 助手
  description: 了解如何使用语义内核生成可执行 DevOps 任务的生成式 AI 助手。
---

# 使用语义内核创建 AI 助手

在本实验室中，你将开发 AI 支持的助手的代码，旨在自动执行开发运营并帮助简化任务。 你将使用语义内核 SDK 生成该 AI 助手，并将其连接到大型语言模型 (LLM) 服务。 使用语义内核 SDK，能够创建一个可与 LLM 服务交互、对自然语言查询作出响应并为用户提供个性化见解的智能应用程序。 在本练习中，提供了模拟函数来表示典型的 DevOps 任务。 现在就开始吧！

此练习大约需要 **30** 分钟。

## 部署聊天完成模型

1. 导航到 [https://portal.azure.com](https://portal.azure.com)。

1. 使用默认设置创建新的 Azure OpenAI 资源****。

1. 创建资源后，选择“转到资源”****。

1. 在“**概述**”页上，选择“**转到 Azure Foundry 门户**”。

    Azure AI Foundry 门户应在一个新标签页中打开。

1. 在左侧导航窗格中，选择“部署”****。

1. 选择“部署模型”，然后选择“部署基础模型”********。

1. 在模型列表上搜索“gpt-4o”，选择它，然后选择确认********。

    随即应会显示一个对话框，用于配置向 Azure OpenAI 资源的部署。

1. 查看设置，然后选择“部署”****。

    部署完成时，“部署详细信息”页将会出现。

1. 在“终结点”下，观察“目标 URI”和“密钥”************。

    你将在下一个任务中使用此处的值来生成内核。 请记得将密钥保密并妥善保存！

## 准备应用程序配置

1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。

    关闭任何欢迎通知以查看 Azure 门户主页。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中新建 Cloud Shell，选择订阅中不含存储的 ***PowerShell*** 环境。

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。 可以调整此窗格的大小或最大化此窗格，以便更易于使用。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
    rm -r semantic-kernel -f
    git clone https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK semantic-kernel
    ```

    > **提示**：将命令粘贴到 Cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含聊天应用程序代码文件的文件夹：

    > **备注**：按照所选编程语言的步骤操作。

    **Python**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/python
    ```

    **C#**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/c-sharp
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令以安装将要使用的库：

    **Python**
    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    **C#**
    ```
    dotnet add package Microsoft.Extensions.Configuration
    dotnet add package Microsoft.Extensions.Configuration.Json
    dotnet add package Microsoft.SemanticKernel
    dotnet add package Microsoft.SemanticKernel.PromptTemplates.Handlebars
    ```

1. 输入以下命令以编辑已提供的配置文件：

    **Python**
    ```
    code .env
    ```

    **C#**
    ```
    code appsettings.json
    ```

    该文件已在代码编辑器中打开。

1. 更新 Azure OpenAI 模型部署中的值：

    **Python**
    ```python
    MODEL_ENDPOINT=""
    API_KEY="
    MODEL_DEPLOYMENT_NAME=""
    ```

    **C#**
    ```json
    {
        "openai_endpoint": "",
        "api_key": "",
        "model_deployment_name": "",
    }
    ```

> **注意**：如果使用 C#，请使用资源“主页”页面上的“Azure OpenAI”终结点 URL 作为 `openai_endpoint` 值********。

1. 更新值后，使用 **CTRL+S** 命令保存更改，然后使用 **CTRL+Q** 命令关闭代码编辑器，同时保持 Cloud Shell 命令行打开状态。

## 创建语义内核插件

1. 输入以下命令以编辑已提供的代码文件：

    **Python**
    ```
    code devops.py
    ```

    **C#**
    ```
    code Program.cs
    ```

1. 在注释“**使用 Azure OpenAI 聊天补全创建内核生成器**”下，添加以下代码：

    **Python**
    ```python
    # Create a kernel builder with Azure OpenAI chat completion
    kernel = Kernel()
    chat_completion = AzureChatCompletion(
        deployment_name=model_name,
        api_key=api_key,
        base_url=endpoint,
    )
    kernel.add_service(chat_completion)
    ```
    **C#**
     ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelName, endpoint, apiKey);
    var kernel = builder.Build();
    ```

1. 在文件底部附近的“DevopsPlugin”类中，查找注释“创建内核函数以生成阶段环境”，并添加以下代码以创建将生成过渡环境的模拟插件 functin：********

    **Python**
    ```python
    # Create a kernel function to build the stage environment
    @kernel_function(name="BuildStageEnvironment")
    def build_stage_environment(self):
        return "Stage build completed."
    ```

    **C#**
    ```c#
    // Create a kernel function to build the stage environment
    [KernelFunction("BuildStageEnvironment")]
    public string BuildStageEnvironment() 
    {
        return "Stage build completed.";
    }
    ```

    `KernelFunction`修饰器声明本机函数。 对函数使用描述性名称，以便 AI 可以正确调用它。 

1. 在 main 方法中，导航到注释“将插件导入内核”，并添加以下代码以使用已完成的插件类：********

    **Python**
    ```python
    # Import plugins to the kernel
    kernel.add_plugin(DevopsPlugin(), plugin_name="DevopsPlugin")
    ```

    **C#**
    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<DevopsPlugin>();
    ```

1. 在注释“**创建提示执行设置**”下，添加以下代码以自动调用函数：

    **Python**
    ```python
    # Create prompt execution settings
    execution_settings = AzureChatPromptExecutionSettings()
    execution_settings.function_choice_behavior = FunctionChoiceBehavior.Auto()
    ```

    **C#**
    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    使用此设置将允许内核自动调用函数，而无需在提示符中指定它们。

1. 在注释“**创建聊天历史记录**”下添加以下代码：

    **Python**
    ```python
    # Create chat history
    chat_history = ChatHistory()
    ```

    **C#**
    ```c#
    // Create chat history
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. 取消对位于注释“**用户交互逻辑**”之后的代码块的注释

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。

## 运行 DevOps 助手代码

1. 在 Cloud Shell 命令行窗格中，输入以下命令以登录到 Azure。

    ```
    az login
    ```

    **<font color="red">必须登录到 Azure - 即使 Cloud Shell 会话已经过身份验证。</font>**

    > **备注**：在大多数情况下，仅使用 *az login* 就足够了。 但是，如果在多个租户中有订阅，则可能需要使用 *--tenant* 参数指定租户。 有关详细信息，请参阅[使用 Azure CLI 以交互方式登录到 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。

1. 出现提示时，请按照说明在新选项卡中打开登录页，并输入提供的验证码和 Azure 凭据。 然后在命令行中完成登录过程，并在出现提示时选择包含 Azure AI Foundry 中心的订阅。

1. 登录后，输入以下命令来运行应用程序：


    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 出现提示时，输入以下提示`Please build the stage environment`

1. 你应该会看到类似于以下输出的响应：

    ```output
    Assistant: The stage environment has been successfully built.
    ```

1. 接下来，输入以下提示`Please deploy the stage environment`

1. 你应该会看到类似于以下输出的响应：

    ```output
    Assistant: The staging site has been deployed successfully.
    ```

1. 按 <kbd>Enter</kbd> 结束程序。

## 根据提示创建内核函数

1. 在注释`Create a kernel function to deploy the staging environment`下，添加以下代码

     **Python**
    ```python
    # Create a kernel function to deploy the staging environment
    deploy_stage_function = KernelFunctionFromPrompt(
        prompt="""This is the most recent build log:
        {{DevopsPlugin.ReadLogFile}}

        If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function""",
        function_name="DeployStageEnvironment",
        description="Deploy the staging environment"
    )

    kernel.add_function(plugin_name="DeployStageEnvironment", function=deploy_stage_function)
    ```

    **C#**
    ```c#
    // Create a kernel function to deploy the staging environment
    var deployStageFunction = kernel.CreateFunctionFromPrompt(
    promptTemplate: @"This is the most recent build log:
    {{DevopsPlugin.ReadLogFile}}

    If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function",
    functionName: "DeployStageEnvironment",
    description: "Deploy the staging environment"
    );

    kernel.Plugins.AddFromFunctions("DeployStageEnvironment", [deployStageFunction]);
    ```

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。

1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 出现提示时，输入以下提示`Please deploy the stage environment`

1. 你应该会看到类似于以下输出的响应：

    ```output
    Assistant: The stage environment cannot be deployed because the earlier stage build failed due to unit test errors. Deploying a faulty build to stage may cause eventual issues and compromise the environment.
    ```

    来自 LLM 的响应可能会有所不同，但仍会阻止你部署暂存站点。 
    
1. 按 <kbd>Enter</kbd> 结束程序。

## 创建 Handlebars 提示

1. 在注释“**创建 handlebars 提示**”下添加以下代码：

    **Python**
    ```python
    # Create a handlebars prompt
    hb_prompt = """<message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">"""
    ```

    **C#**
    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    在此代码中，你将使用 Handlebars 模板格式创建少样本提示。 提示将引导模型在创建新的分支之前从用户检索更多信息。

1. 在注释“**使用 handlebars 格式创建提示模板配置**”下添加以下代码：

    **Python**
    ```python
    # Create the prompt template config using handlebars format
    hb_template = HandlebarsPromptTemplate(
        prompt_template_config=PromptTemplateConfig(
            template=hb_prompt, 
            template_format="handlebars",
            name="CreateBranch", 
            description="Creates a new branch for the user",
            input_variables=[
                InputVariable(name="input", description="The user input", is_required=True)
            ]
        ),
        allow_dangerously_set_content=True,
    )
    ```

    **C#**
    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "CreateBranch",
    };
    ```

    此代码从提示中创建 Handlebars 模板配置。 可以使用它创建插件函数。

1. 在注释“**从提示创建插件函数**”下添加以下代码： 

    **Python**
    ```python
    # Create a plugin function from the prompt
    prompt_function = KernelFunctionFromPrompt(
        function_name="CreateBranch",
        description="Creates a branch for the user",
        template_format="handlebars",
        prompt_template=hb_template,
    )
    kernel.add_function(plugin_name="BranchPlugin", function=prompt_function)
    ```

    **C#**
    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var branchPlugin = kernel.CreatePluginFromFunctions("BranchPlugin", [promptFunction]);
    kernel.Plugins.Add(branchPlugin);
    ```

    此代码为提示创建一个插件函数，并将其添加到内核。 现在，你已准备就绪，可调用函数。

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。

1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 出现提示时，输入以下文本：`Please create a new branch`

1. 你应该会看到类似于以下输出的响应：

    ```output
    Assistant: Could you please provide the following details?

    1. The name of the new branch.
    2. The base branch from which the new branch should be created.
    ```

1. 输入以下文本 `feature-login main`

1. 你应该会看到类似于以下输出的响应：

    ```output
    Assistant: The new branch `feature-login` has been successfully created from `main`.
    ```

1. 按 <kbd>Enter</kbd> 结束程序。

## 要求用户同意操作

1. 在文件底部附近，找到注释“**创建函数筛选器**”，并添加以下代码：

    **Python**
    ```python
    # Create a function filter
    async def permission_filter(context: FunctionInvocationContext, next: Callable[[FunctionInvocationContext], Awaitable[None]]) -> None:
        await next(context)
        result = context.result
        
        # Check the plugin and function names
    ```

    **C#**
    ```c#
    // Create a function filter
    class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

1. 在注释“**检查插件和函数名称**”下添加以下代码，以检测何时调用 `DeployToProd` 函数：

     **Python**
    ```python
    # Check the plugin and function names
    if context.function.plugin_name == "DevopsPlugin" and context.function.name == "DeployToProd":
        # Request user approval
        
        # Proceed if approved
    ```

    **C#**
    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "DevopsPlugin" && context.Function.Name == "DeployToProd"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    此代码使用 `FunctionInvocationContext` 对象，以确定调用的插件和函数。

1. 添加以下逻辑，请求用户的权限以继续进行操作：

    请务必保持正确的缩进级别。

    **Python**
    ```python
    # Request user approval
    print("System Message: The assistant requires approval to complete this operation. Do you approve (Y/N)")
    should_proceed = input("User: ").strip()

    # Proceed if approved
    if should_proceed.upper() != "Y":
        context.result = FunctionResult(
            function=result.function,
            value="The operation was not approved by the user",
        )
    ```

    **C#**
    ```c#
    // Request user approval
    Console.WriteLine("System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)");
    Console.Write("User: ");
    string shouldProceed = Console.ReadLine()!;

    // Proceed if approved
    if (shouldProceed != "Y")
    {
        context.Result = new FunctionResult(context.Result, "The operation was not approved by the user");
        return;
    }
    ```

1. 导航到注释“**将筛选器添加到内核**”，然后添加以下代码：

    **Python**
    ```python
    # Add filters to the kernel
    kernel.add_filter('function_invocation', permission_filter)
    ```

    **C#**
    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。

1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. 输入将生成部署到生产环境的提示。 你应该看到如下所示的响应：

    ```output
    User: Please deploy the build to prod
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to proceed with the deployment.
    ```

1. 按 <kbd>Enter</kbd> 结束程序。

### 审阅

在此实验室中，为大型语言模型 (LLM) 服务创建了一个终结点，构建了一个语义内核对象，并使用语义内核 SDK 运行提示。 还创建了插件，并利用系统消息来引导模型。 祝贺你完成本实验！