---
lab:
  title: 实验室：使用 Azure OpenAI 和语义内核 SDK 开发 AI 代理
  module: 'Module 01: Build your kernel'
---

# 实验室：完成 AI 旅行助手
# 学生实验室手册

在本实验中，你将使用语义内核 SDK 完成 AI 旅行助手。 你将为大型语言模型 (LLM) 服务创建一个终结点，创建语义内核函数，并使用语义内核 SDK 的自动函数调用功能将用户意图路由到相应的插件，包括现有的一些预生成插件。 你还将使用对话历史记录为 LLM 提供上下文，并允许用户继续对话。

## 实验室场景

你是某家旅行社的开发人员，专门为客户打造个性化的旅行体验。 你的任务是创建 AI 旅行助手，以帮助客户详细了解旅行目的地并规划行程活动。 该 AI 旅行助手能够换算货币金额、推荐目的地和活动、提供不同语言的有用短语并翻译短语。 AI 旅行助手还应该能够利用对话历史记录，针对用户的请求提供与上下文相关的回复。

## 目标

通过本实验，你将完成以下任务：

* 为大型语言模型 (LLM) 服务创建终结点
* 生成语义内核对象
* 使用语义内核 SDK 运行提示
* 创建语义内核函数和插件
* 使用语义内核 SDK 的自动函数调用功能

## 实验室教学设置

### 先决条件

要完成本练习，您需要在系统中安装以下设备：

* [Visual Studio Code](https://code.visualstudio.com)
* [最新的 .NET 8.0 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
* Visual Studio Code 的 [C# 扩展](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

### 准备开发环境

在这些练习中，你可以使用初学者项目。 使用以下步骤设置初学者项目：

> [!IMPORTANT]
> 必须安装 .NET Framework 8.0 以及 C# 和 NuGet 包管理器的 VS Code 扩展。

1. 将链接粘贴到新的浏览器窗口中：
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip`

1. 单击位于页面右上角的 <kbd>...</kbd> 按钮，或按 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd> 下载 zip 文件。

1. 将 zip 文件的内容提取到容易查找和记住的位置，例如桌面上的文件夹。

1. 打开 Visual Studio Code，选择“文件” > “打开文件夹”********。

1. 导航到提取的 Starter**** 文件夹并选择“选择文件夹”****。

1. 在代码编辑器中打开“Program.cs”文件。

> [!NOTE]
> 如果系统提示你信任该文件夹，请选择**是，我信任作者**。

## 练习 1：使用语义内核 SDK 创建插件

在本练习中，将为大型语言模型 (LLM) 服务创建终结点。 语义内核 SDK 使用此终结点连接到 LLM 并运行提示。 语义内核 SDK 支持 HuggingFace、OpenAI 和 Azure Open AI LLM。 在本例中将使用 Azure Open AI。

完成练习估计所需时间****：10 分钟

### 任务 1：创建 Azure OpenAI 资源

1. 导航到 [https://portal.azure.com](https://portal.azure.com)。

1. 使用默认设置创建新的 Azure OpenAI 资源。

    > [!NOTE]
    > 如果已有 Azure OpenAI 资源，则可以跳过此步骤。

1. 创建资源后，选择“转到资源”****。

1. 在“**概述**”页上，选择“**转到 Azure Foundry 门户**”。

1. 依次选择“**新建部署**”、“**从基础模型**”。

1. 在模型列表中，选择 **gpt-35-turbo-16k**。

1. 选择“确认”

1. 输入部署的名称并保留默认选项。

1. Azure 门户部署完成后，导航回 Azure 门户中的 Azure OpenAI 资源。

1. 在“资源管理”下，转到“密钥和终结点”********。

    你将在下一个任务中使用此处提供的数据来生成内核。 请记得将密钥保密并妥善保存！

### 任务 2：创建本机插件

在此任务中，你将创建一个本机函数插件，可以实现从基准货币到目标货币的金额换算。

1. 返回到 Visual Studio Code 项目。

1. 打开 **appsettings.json** 文件，并使用 Azure OpenAI 服务模型 ID、终结点和 API 密钥更新值。

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. 导航至 **Plugins** 文件夹中名为 **CurrencyConverterPlugin.cs** 的文件

1. 在 **CurrencyConverterPlugin.cs** 文件中，在注释“**Create a kernel function that gets the exchange rate**”下添加以下代码：

    ```c#
    // Create a kernel function that gets the exchange rate
    [KernelFunction("convert_currency")]
    [Description("Converts an amount from one currency to another, for example USD to EUR")]
    public static decimal ConvertCurrency(decimal amount, string fromCurrency, string toCurrency)
    {
        decimal exchangeRate = GetExchangeRate(fromCurrency, toCurrency);
        return amount * exchangeRate;
    }
    ```

    在此段代码中，你将使用 **KernelFunction** 修饰器来声明本机函数。 还将使用**说明**修饰器来添加函数功能的说明。 接下来，添加一些逻辑来实现从一种货币到另一种货币的给定金额换算。

1. 打开 Program.cs 文件

1. 在注释“**将插件添加到内核**”下导入货币转换器插件：

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    ```

    接下来，让我们测试插件。

1. 右键单击 **Program.cs** 文件，然后单击“在集成终端中打开”

1. 在终端中输入 `dotnet run`。 

    输入兑换货币的提示请求，例如“在香港，10 美元是多少？”

    应会看到类似于下面的信息：

    ```output
    Assistant: 10 USD is equivalent to 77.70 Hong Kong dollars (HKD).
    ```

## 练习 2：创建 Handlebars 提示

在本练习中，你将从 Handlebars 提示中创建一个函数。 该函数将提示 LLM 为用户创建旅行行程。 现在就开始吧！

完成练习估计所需时间****：10 分钟

### 任务 1：从 Handlebars 提示中创建函数

1. 在 **Program.cs** 文件中添加以下 `using` 指令：

    `using Microsoft.SemanticKernel.PromptTemplates.Handlebars;`

1. 在注释“**创建 handlebars 提示**”下添加以下代码：

    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before providing the user with a travel itinerary, ask how many days their trip is</message>
        <message role="user">I'm going to {{city}}. Can you create an itinerary for me?</message>
        <message role="assistant">Sure, how many days is your trip?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    在此代码中，你将使用 Handlebars 模板格式创建少样本提示。 提示将引导模型在创建旅行行程之前从用户检索更多信息。

1. 在注释“**使用 handlebars 格式创建提示模板配置**”下添加以下代码：

    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "GetItinerary",
    };
    ```

    此代码从提示中创建 Handlebars 模板配置。 可以使用它创建插件函数。

1. 在注释“**从提示创建插件函数**”下添加以下代码： 

    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var itineraryPlugin = kernel.CreatePluginFromFunctions("TravelItinerary", [promptFunction]);
    kernel.Plugins.Add(itineraryPlugin);
    ```

    此代码为提示创建一个插件函数，并将其添加到内核。 现在，你已准备就绪，可调用函数。

1. 在终端中，输入 `dotnet run` 以运行代码。

    请尝试以下输入，提示 LLM 提供行程。

    ```output
    Assistant: How may I help you?
    User: I'm going to Hong Kong, can you create an itinerary for me?
    Assistant: Sure! How many days will you be staying in Hong Kong?
    User: 10
    Assistant: Great! Here's a 10-day itinerary for your trip to Hong Kong:
    ...
    ```

    现在，你有了 AI 旅行助手的雏形！ 让我们使用提示和插件来添加更多功能

1.  在注释“**将插件添加到内核**”下添加航班预订插件：

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    kernel.ImportPluginFromType<FlightBookingPlugin>();
    ```

    此插件使用带有模拟详细信息的 **flights.json** 文件模拟外部测试版预订。 接下来，向助手添加一些其他系统提示。

1.  在注释“**向聊天添加系统消息**”下添加以下代码：

    ```c#
    // Add system messages to the chat
    history.AddSystemMessage("The current date is 01/10/2025");
    history.AddSystemMessage("You are a helpful travel assistant.");
    history.AddSystemMessage("Before providing destination recommendations, ask the user about their budget.");
    ```

    这些提示将有助于创建流畅的用户体验，并帮助模拟航班预订插件。 现在，可以测试代码。

1. 在终端中输入 `dotnet run`。

    请尝试输入以下一些提示：

    ```output
    1. Can you give me some destination recommendations for Europe?
    2. I want to go to Barcelona, can you create an itinerary for me?
    3. How many Euros is 100 USD?
    4. Can you book me a flight to Barcelona?
    ```

    尝试其他输入并查看旅行助理的响应方式。

## 练习 3：要求用户同意操作

在本练习中，你将添加一个筛选器调用函数，该函数将在允许助手代表用户预订航班之前请求用户的批准。 现在就开始吧！

### 任务 1：创建函数调用筛选器

1. 创建名为 **PermissionFilter.cs**的新文件。

1. 在新单元格中，输入以下代码：

    ```c#
    #pragma warning disable SKEXP0001 
    using Microsoft.SemanticKernel;
    
    public class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

    >[!NOTE] 
    > 在语义内核 SDK 版本 1.30.0 中，函数筛选器可能会更改，需要警告抑制。 

    在此代码中，将实现 `IFunctionInvocationFilter` 接口。 每当从 AI 助手调用函数时，始终调用 `OnFunctionInvocationAsync` 方法。

1. 添加以下代码以检测何时调用 `book_flight` 函数：

    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "FlightBookingPlugin" && context.Function.Name == "book_flight"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    此代码使用 `FunctionInvocationContext` 确定调用的插件和函数。

1. 添加以下逻辑以请求用户预订外部测试版的权限：

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

1. 导航到 **Program.cs** 文件。

1. 在注释“**将筛选器添加到内核**”下添加以下代码：

    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. 在终端中输入 `dotnet run`。

    输入预订外部测试版的提示。 应该看到类似于以下示例的响应：

    ```output
    User: Find me a flight to Tokyo on the 19
    Assistant: I found a flight to Tokyo on the 19th of January. The flight is with Air Japan and the price is $1200.
    User: Y
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to book the flight for you.
    ```

    在继续任何预订之前，助手应要求用户审批。

### 审阅

在此实验室中，为大型语言模型 (LLM) 服务创建了一个终结点，构建了一个语义内核对象，并使用语义内核 SDK 运行提示。 还创建了插件，并利用系统消息来引导模型。 祝贺你完成本实验！
