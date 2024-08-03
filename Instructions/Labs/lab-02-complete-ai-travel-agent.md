---
lab:
  title: 实验室：使用 Azure OpenAI 和语义内核 SDK 开发 AI 代理
  module: 'Module 01: Build your kernel'
---

# 实验室：完成 AI 旅行服务代理
# 学生实验室手册

在本实验中，你将使用语义内核 SDK 完成一个 AI 旅行服务代理。 你将为大型语言模型 (LLM) 服务创建一个终结点，创建语义内核函数，并使用语义内核 SDK 的自动函数调用功能将用户意图路由到相应的插件，包括现有的一些预生成插件。 你还将使用对话历史记录为 LLM 提供上下文，并允许用户继续对话。

## 实验室场景

你是某家旅行社的开发人员，专门为客户打造个性化的旅行体验。 你的任务是创建一个 AI 旅行服务代理，以帮助客户详细了解旅行目的地并规划行程活动。 该 AI 旅行服务代理能够换算货币金额、推荐目的地和活动、以不同的语言提供有用短语以及翻译短语。 该 AI 旅行服务代理还能够利用对话历史记录为用户请求提供区分上下文的响应。

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
* [最新的 .NET 7.0 SDK。](https://dotnet.microsoft.com/download/dotnet/7.0)
* Visual Studio Code 的 [C# 扩展](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

### 准备开发环境

在这些练习中，你可以使用初学者项目。 使用以下步骤设置初学者项目：

> [!IMPORTANT]
> 必须安装 .NET Framework 8.0 以及 C# 和 NuGet 包管理器的 VS Code 扩展。

1. 下载位于 `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip` 的 zip 文件。

1. 将 zip 文件的内容提取到容易查找和记住的位置，例如桌面上的文件夹。

1. 打开 Visual Studio Code，选择“文件” > “打开文件夹”********。

1. 导航到提取的 Starter**** 文件夹并选择“选择文件夹”****。

1. 在代码编辑器中打开“Program.cs”文件。

## 练习 1：使用语义内核 SDK 创建插件

在本练习中，将为大型语言模型 (LLM) 服务创建终结点。 语义内核 SDK 使用此终结点连接到 LLM 并运行提示。 语义内核 SDK 支持 HuggingFace、OpenAI 和 Azure Open AI LLM。 在本例中将使用 Azure Open AI。

完成练习估计所需时间****：10 分钟

### 任务 1：创建 Azure OpenAI 资源

1. 导航到 [https://portal.azure.com](https://portal.azure.com)。

1. 使用默认设置创建新的 Azure OpenAI 资源。

    > [!NOTE]
    > 如果已有 Azure OpenAI 资源，则可以跳过此步骤。

1. 创建资源后，选择“转到资源”****。

1. 在“概述”页上，选择“转到 Azure OpenAI Studio”。********

:::image type="content" source="../media/model-deployments.png" alt-text="Azure OpenAI 部署页的屏幕截图。":::

1. 选择“创建新部署”****，然后选择“+创建新部署”****。

1. 在**部署模型**弹出窗口中，选择 **gpt-35-turbo-16k**。

    使用默认模型版本

1. 输入部署的名称

1. 部署完成后，导航回 Azure OpenAI 资源。

1. 在“资源管理”下，转到“密钥和终结点”********。

    在下一个任务中，你将使用这些值来生成内核。 请记得将密钥保密并妥善保存！

1. 在 Visual Studio Code 中返回到 Program.cs 文件****。

1. 使用 Azure Open AI 服务部署名称、API 密钥、终结点更新以下变量

    ```csharp
    string yourDeploymentName = "";
    string yourEndpoint = "";
    string yourApiKey = "";
    ```

    > [!NOTE]
    > 部署模型必须是“gpt-35-turbo-16k”，某些语义内核 SDK 功能才能正常工作。

### 任务 2：创建本机函数

在此任务中，你将创建一个本机函数，用于将基准货币金额换算成目标货币金额。

1. 在 Plugins/ConvertCurrency 文件夹中创建一个名为 `CurrencyConverter.cs` 的新文件****

1. 在 `CurrencyConverter.cs` 文件中，添加以下代码以创建插件函数：

    ```c#
    using AITravelAgent;
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    class CurrencyConverter
    {
        [KernelFunction, 
        Description("Convert an amount from one currency to another")]
        public static string ConvertAmount()
        {
            var currencyDictionary = Currency.Currencies;
        }
    }
    ```

    在此段代码中，你将使用 `KernelFunction` 修饰器来声明本机函数。 还将使用 `Description` 修饰器来添加函数功能的说明。 你可以使用 `Currency.Currencies` 来获取货币字典及其汇率。 接下来，添加一些逻辑来实现从一种货币到另一种货币的给定金额换算。

1. 使用以下代码修改 `ConvertAmount` 函数：

    ```c#
    [KernelFunction, Description("Convert an amount from one currency to another")]
    public static string ConvertAmount(
        [Description("The target currency code")] string targetCurrencyCode, 
        [Description("The amount to convert")] string amount, 
        [Description("The starting currency code")] string baseCurrencyCode)
    {
        var currencyDictionary = Currency.Currencies;
        
        Currency targetCurrency = currencyDictionary[targetCurrencyCode];
        Currency baseCurrency = currencyDictionary[baseCurrencyCode];

        if (targetCurrency == null)
        {
            return targetCurrencyCode + " was not found";
        }
        else if (baseCurrency == null)
        {
            return baseCurrencyCode + " was not found";
        }
        else
        {
            double amountInUSD = Double.Parse(amount) * baseCurrency.USDPerUnit;
            double result = amountInUSD * targetCurrency.UnitsPerUSD;
            return @$"${amount} {baseCurrencyCode} is approximately 
                {result.ToString("C")} in {targetCurrency.Name}s ({targetCurrencyCode})";
        }
    }
    ```

    在此段代码中，你将使用 `Currency.Currencies` 字典来获取目标和基准货币的 `Currency` 对象。 然后使用 `Currency` 对象进行从基准货币到目标货币的金额换算。 最后，返回一个包含换算后的金额的字符串。 接下来，让我们测试插件。

1. 在 `Program.cs` 文件中，请使用以下代码导入并调用新插件函数：

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync("CurrencyConverter", 
        "ConvertAmount", 
        new() {
            {"targetCurrencyCode", "USD"}, 
            {"amount", "52000"}, 
            {"baseCurrencyCode", "VND"}
        }
    );

    Console.WriteLine(result);
    ```

    在此段代码中，你将使用 `ImportPluginFromType` 方法来导入插件。 然后使用 `InvokeAsync` 方法来调用插件函数。 `InvokeAsync` 方法将采用插件名称、函数名称和参数字典。 最后，你会将结果打印到控制台。 接下来，请运行代码以确保其正常工作。

1. 在终端中输入 `dotnet run`。 应会看到以下输出：

    ```output
    $52000 VND is approximately $2.13 in US Dollars (USD)
    ```

    插件正常运行后，让我们创建一个可以检测用户想要换算的货币和金额的自然语言提示。

### 任务 3：使用提示分析用户输入

在此任务中，你将创建一个提示来分析用户的输入，以确定目标货币、基准货币和要换算的金额。

1. 在“Prompts”**** 文件夹中创建一个名为 `GetTargetCurrencies` 的新文件夹

1. 在 `GetTargetCurrencies` 文件夹中，创建一个名为 `config.json` 的新文件

1. 在 `config.json` 文件中输入以下文本：

    ```output
    {
        "schema": 1,
        "type": "completion",
        "description": "Identify the target currency, base currency, and amount to convert",
        "execution_settings": {
            "default": {
                "max_tokens": 800,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "input",
                "description": "Text describing some currency amount to convert",
                "required": true
            }
        ]
    }
    ```

1. 在 `GetTargetCurrencies` 文件夹中，创建名为 `skprompt.txt` 的新文件

1. 在 `skprompt.txt` 文件中输入以下文本：

    ```html
    <message role="system">Identify the target currency, base currency, and 
    amount from the user's input in the format target|base|amount</message>

    For example: 

    <message role="user">How much in GBP is 750.000 VND?</message>
    <message role="assistant">GBP|VND|750000</message>

    <message role="user">How much is 60 USD in New Zealand Dollars?</message>
    <message role="assistant">NZD|USD|60</message>

    <message role="user">How many Korean Won is 33,000 yen?</message>
    <message role="assistant">KRW|JPY|33000</message>

    <message role="user">{{$input}}</message>
    <message role="assistant">target|base|amount</message>
    ```

### 任务 4：检查你的工作

在此任务中，你将运行应用程序并验证代码是否正常工作。 

1. 请使用以下代码来更新 `Program.cs` 文件，以测试新提示：

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync(prompts["GetTargetCurrencies"],
        new() {
            {"input", "How many Australian Dollars is 140,000 Korean Won worth?"}
        }
    );

    Console.WriteLine(result);
    ```

1. 在终端中输入 `dotnet run`。 应会看到以下输出：

    ```output
    AUD|KRW|140000
    ```

    > [!NOTE]
    > 如果代码未生成预期的输出，可以在“Solution”**** 文件夹中查看代码。 你可能需要调整 `skprompt.txt` 文件中的提示，以生成更精确的结果。

现在，你有一个可以实现从一种货币到另一种货币的金额换算的插件，以及可用于将用户的输入分析为可供 `ConvertAmount` 函数使用的格式的提示。 这样，用户就可以使用你的 AI 旅行服务代理轻松换算货币金额。

## 练习 2：根据用户意图自动选择插件

在本练习中，你会检测用户意向，并将对话路由到所需的插件。 可以使用提供的插件检索用户意向。 现在就开始吧！

完成练习估计所需时间****：10 分钟

### 任务 1：使用 GetIntent 插件

1. 使用以下代码更新`Program.cs`文件：

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();

    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );

    ```

    在此代码中，你使用`GetIntent`提示检测用户意向。 然后，将意向存储在名为`intent`的变量中。 接下来，将意向路由到`CurrencyConverter`插件。

1. 将以下代码添加到 `Program.cs` 文件：

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            var currencyText = await kernel.InvokeAsync<string>(
                prompts["GetTargetCurrencies"], 
                new() {{ "input",  input }}
            );
            var currencyInfo = currencyText!.Split("|");
            var result = await kernel.InvokeAsync("CurrencyConverter", 
                "ConvertAmount", 
                new() {
                    {"targetCurrencyCode", currencyInfo[0]}, 
                    {"baseCurrencyCode", currencyInfo[1]},
                    {"amount", currencyInfo[2]}, 
                }
            );
            Console.WriteLine(result);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    `GetIntent`插件返回以下值：ConvertCurrency, SuggestDestinations, SuggestActivities, Translate, HelpfulPhrases, Unknown. 使用`switch`语句将用户意向路由到相应的插件。 
    
    如果用户意向是兑换货币，则使用`GetTargetCurrencies`提示检索货币信息。 然后，使用`CurrencyConverter`插件兑换金额。

    接下来，添加一些事例以处理其他意向。 现在，让我们使用语义内核 SDK 的自动函数调用功能将意向路由到可用的插件。

1. 将以下代码添加到`Program.cs`文件以创建自动函数调用设置：

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    OpenAIPromptExecutionSettings settings = new()
    {
        ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
    };

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();
    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );
    ```

    接下来，将事例添加到 switch 语句中以实现其他意向。

1. 使用以下代码更新`Program.cs`文件：

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            // ...Code you entered previously...
            break;
        case "SuggestDestinations":
        case "SuggestActivities":
        case "HelpfulPhrases":
        case "Translate":
            var autoInvokeResult = await kernel.InvokePromptAsync(input!, new(settings));
            Console.WriteLine(autoInvokeResult);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    在此代码中，使用`AutoInvokeKernelFunctions`设置自动调用内核中引用的函数和提示。 如果用户意向是兑换货币，`CurrencyConverter`插件会执行其任务。 
    
    如果用户意向是获取目标或活动建议、翻译短语或获取语言的有用短语，`AutoInvokeKernelFunctions`设置会自动调用项目代码中包含的现有插件。

    如果代码不属于上述任何意向，还可以添加代码以运行用户输入，作为大型语言模型 (LLM)。

1. 使用以下代码更新默认用例：

    ```c#
    default:
        Console.WriteLine("Sure, I can help with that.");
        var otherIntentResult = await kernel.InvokePromptAsync(input!, new(settings));
        Console.WriteLine(otherIntentResult);
        break;
    ```

    现在，如果用户有不同的意向，LLM 可以处理用户的请求。 让我们尝试一下吧！

### 任务 2：检查你的工作

在此任务中，你会运行应用程序并验证代码是否正常工作。 

1. 在终端中输入 `dotnet run`。 出现提示时，输入类似于以下提示的一些文本：

    ```output
    What would you like to do?
    How many TTD is 50 Qatari Riyals?    
    ```

1. 应会看到类似于以下响应的输出：

    ```output
    $50 QAR is approximately $93.10 in Trinidadian Dollars (TTD)
    ```

1. 在终端中输入 `dotnet run`。 出现提示时，输入类似于以下提示的一些文本：

    ```output
    What would you like to do?
    I want to go somewhere that has lots of warm sunny beaches and delicious, spicy food!
    ```

1. 应会看到类似于以下响应的输出：

    ```output
    Based on your preferences for warm sunny beaches and delicious, spicy food, I have a few destination recommendations for you:

    1. Thailand: Known for its stunning beaches, Thailand offers a perfect combination of relaxation and adventure. You can visit popular beach destinations like Phuket, Krabi, or Koh Samui, where you'll find crystal-clear waters and white sandy shores. Thai cuisine is famous for its spiciness, so you'll have plenty of mouthwatering options to try, such as Tom Yum soup, Pad Thai, and Green Curry.

    2. Mexico: Mexico is renowned for its beautiful coastal regions and vibrant culture. You can explore destinations like Cancun, Playa del Carmen, or Tulum, which boast stunning beaches along the Caribbean Sea. Mexican cuisine is rich in flavors and spices, offering a wide variety of dishes like tacos, enchiladas, and mole sauces that will satisfy your craving for spicy food.

    ...

    These destinations offer a perfect blend of warm sunny beaches and delicious, spicy food, ensuring a memorable trip for you. Let me know if you need any further assistance or if you have any specific preferences for your trip!
    ```

1. 在终端中输入 `dotnet run`。 出现提示时，输入类似于以下提示的一些文本：

    ```output
    What would you like to do?
    Can you give me a recipe for chicken satay?

1. You should see a response similar to the following response:

    ```output
    Sure, I can help with that.
    Certainly! Here's a recipe for chicken satay:

    ...
    ```

    意图应该路由到默认案例，LLM 应该处理鸡肉沙爹食谱的请求。

    > [!NOTE]
    > 如果代码未生成预期的输出，可以在**解决方案**文件夹中查看代码。

接下来，让我们修改路由逻辑，以为某些插件提供一些聊天历史记录。 提供历史记录允许插件检索对用户请求的更多上下文相关响应。

### 任务 3：完成插件路由

在本练习中，你将使用对话历史记录向大型语言模型 (LLM) 提供上下文。 你还将调整代码，以便用户能够继续相应对话，就像使用真正的聊天机器人一样。 现在就开始吧！

1. 修改代码以使用 `do`-`while` 循环接受用户的输入：

    ```c#
    string input;

    do 
    {
        Console.WriteLine("What would you like to do?");
        input = Console.ReadLine();

        // ...
    }
    while (!string.IsNullOrWhiteSpace(input));
    ```

    现在，你可以使对话持续进行下去，直到用户输入一个空白行。

1. 通过修改 `SuggestDestinations` 案例捕获有关用户行程的详细信息：

    ```c#
    case "SuggestDestinations":
        chatHistory.AppendLine("User:" + input);
        var recommendations = await kernel.InvokePromptAsync(input!);
        Console.WriteLine(recommendations);
        break;
    ```

1. 将 `SuggestActivities` 案例中的行程详细信息与以下代码一起使用：

    ```c#
     case "SuggestActivities":
        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});
        break;
    ```

    在此段代码中，你将使用内置 `SummarizeConversation` 函数来汇总与用户的聊天。 接下来，让我们使用汇总内容来提供有关目的地活动的建议。

1. 使用以下代码扩展 `SuggestActivities` 案例：

    ```c#
    var activities = await kernel.InvokePromptAsync(
        input,
        new () {
            {"input", input},
            {"history", chatSummary},
            {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
    });

    chatHistory.AppendLine("User:" + input);
    chatHistory.AppendLine("Assistant:" + activities.ToString());
    
    Console.WriteLine(activities);
    break;
    ```

    在此段代码中，你会将 `input` 和 `chatSummary` 添加为内核参数。 然后，内核将调用提示并将其路由到 `SuggestActivities` 插件。 你还可以将用户的输入和助手的响应追加到聊天历史记录并显示结果。 接下来，需要将 `chatSummary` 变量添加到 `SuggestActivities` 插件。

1. 导航到 Prompts/SuggestActivities/config.json**** 并在 Visual Studio Code 中打开文件

1. 在 `input_variables` 下，添加对应聊天历史记录的变量：

    ```json
    "input_variables": [
      {
          "name": "history",
          "description": "Some background information about the user",
          "required": false
      },
      {
          "name": "destination",
          "description": "The destination a user wants to visit",
          "required": true
      }
   ]
   ```

1. 导航到 Prompts/SuggestActivities/skprompt.txt**** 并打开该文件

1. 将提示的前半部分替换为以下使用聊天历史记录变量的提示：

    ```html 
    You are an experienced travel agent. 
    You are helpful, creative, and very friendly. 
    Consider the traveler's background: {{$history}}
    ```

    按原样保留提示的其余部分。 现在，插件将使用聊天历史记录向 LLM 提供上下文。

### 任务 4：检查你的工作

在此任务中，你将运行应用程序并验证代码是否正常工作。

1. 将更新的 switch 案例与以下代码进行比较：

    ```c#
    case "SuggestDestinations":
            chatHistory.AppendLine("User:" + input);
            var recommendations = await kernel.InvokePromptAsync(input!);
            Console.WriteLine(recommendations);
            break;
    case "SuggestActivities":

        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});

        var activities = await kernel.InvokePromptAsync(
            input!,
            new () {
                {"input", input},
                {"history", chatSummary},
                {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
        });

        chatHistory.AppendLine("User:" + input);
        chatHistory.AppendLine("Assistant:" + activities.ToString());
        
        Console.WriteLine(activities);
        break;
    ```

1. 在终端中输入 `dotnet run`。 出现提示时，请输入类似于以下内容的一些文本：

    ```output
    What would you like to do?
    How much is 60 USD in new zealand dollars?
    ```

1. 应该会收到类似于以下内容的一些输出：

    ```output
    $60 USD is approximately $97.88 in New Zealand Dollars (NZD)
    What would you like to do?
    ```

1. 输入用于获取目的地建议的提示且其中包含一些上下文指示，例如：

    ```output
    What would you like to do?
    I'm planning an anniversary trip with my spouse, but they are currently using a wheelchair and accessibility is a must. What are some destinations that would be romantic for us?
    ```

1. 应该会收到一些输出，其中包含有关可游览的目的地的建议。

1. 输入用于获取活动建议的提示，例如：

    ```output
    What would you like to do?
    What are some things to do in Barcelona?
    ```

1. 应该会收到符合先前的上下文的建议，例如方便在巴塞罗那参加的活动，类似于以下内容：

    ```output
    1. Visit the iconic Sagrada Família: This breathtaking basilica is an iconic symbol of Barcelona's architecture and is known for its unique design by Antoni Gaudí.

    2. Explore Park Güell: Another masterpiece by Gaudí, this park offers stunning panoramic views of the city, intricate mosaic work, and whimsical architectural elements.

    3. Visit the Picasso Museum: Explore the extensive collection of artworks by the iconic painter Pablo Picasso, showcasing his different periods and styles.
    ```

    > [!NOTE]
    > 如果代码未生成预期的输出，可以在“Solution”**** 文件夹中查看代码。

你可以使用不同的提示和上下文指示来继续测试应用程序。做得不错！ 你已成功向 LLM 提供上下文指示，并调整了代码以允许用户继续对话。

### 审阅

在本实验中，你为大型语言模型 (LLM) 服务创建了一个终结点，生成了一个语义内核对象，使用语义内核 SDK 运行了提示，创建了语义内核函数和插件，并使用语义内核 SDK 的自动函数调用功能将用户意图路由到了相应的插件。 你还使用对话历史记录为 LLM 提供了上下文，并允许用户继续对话。 祝贺你完成本实验！
