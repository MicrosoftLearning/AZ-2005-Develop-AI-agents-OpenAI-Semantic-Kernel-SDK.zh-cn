---
lab:
  title: 实验室：使用 Azure OpenAI 和语义内核 SDK 开发 AI 代理
  module: 'Module 01: Build your kernel'
---

# 实验室：创建 AI 音乐推荐助手
# 学生实验室手册

在本实验室中，你将为 AI 助手创建代码，该助手可以管理用户的音乐库并提供个性化歌曲和音乐会推荐。 你将使用语义内核 SDK 生成该 AI 助手，并将其连接到大型语言模型 (LLM) 服务。 使用语义内核 SDK，能够创建一个可与 LLM 服务交互并为用户提供个性化推荐的智能应用程序。

## 实验室场景

你是某个国际音频流媒体服务的开发人员。 你的任务是将该服务与 AI 集成，以便为用户提供更具个性化的体验。 AI 能够根据用户的听歌历史记录和偏好推荐歌曲以及即将举办的艺术家巡演活动。 你决定使用语义内核 SDK 生成可与大型语言模型 (LLM) 服务交互的 AI 助手。

## 目标

通过本实验，你将完成以下任务：

* 为大型语言模型 (LLM) 服务创建终结点
* 生成语义内核对象
* 使用语义内核 SDK 运行提示
* 创建语义内核函数和插件
* 启用自动函数调用以自动执行任务

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
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip`

1. 通过单击位于页面右上角的 `...` 按钮或按 <kbd>Ctrl</kbd>+<kbd>Shift+</kbd><kbd>S</kbd> 来下载 zip 文件。

1. 将 zip 文件的内容提取到容易查找和记住的位置，例如桌面上的文件夹。

1. 打开 Visual Studio Code，选择“文件” > “打开文件夹”********。

1. 导航到提取的 Starter**** 文件夹并选择“选择文件夹”****。

1. 在代码编辑器中打开“Program.cs”文件。

> [!NOTE]
> 如果系统提示你信任该文件夹，请选择**是，我信任作者**。 

## 练习 1：使用语义内核 SDK 运行提示

在本练习中，将为大型语言模型 (LLM) 服务创建终结点。 语义内核 SDK 使用此终结点连接到 LLM 并运行提示。 语义内核 SDK 支持 HuggingFace、OpenAI 和 Azure Open AI LLM。 在本例中将使用 Azure Open AI。

完成练习估计所需时间****：10 分钟

### 任务 1：创建 Azure OpenAI 资源

1. 导航到 [https://portal.azure.com](https://portal.azure.com)。

1. 使用默认设置创建新的 Azure OpenAI 资源。

    > [!NOTE]
    > 如果已有 Azure OpenAI 资源，则可以跳过此步骤。

1. 创建资源后，选择“转到资源”****。

1. 在“**概述**”页上，选择“**转到 Azure AI Foundry 门户**”。

1. 依次选择“**新建部署**”、“**从基础模型**”。

1. 在模型列表中，选择 **gpt-35-turbo-16k**。

1. 选择“确认”

1. 输入部署的名称并保留默认选项。

1. Azure 门户部署完成后，导航回 Azure 门户中的 Azure OpenAI 资源。

1. 在“资源管理”下，转到“密钥和终结点”********。

    你将在下一个任务中使用此处提供的数据来生成内核。 请记得将密钥保密并妥善保存！

### 任务 2：生成内核

在本练习中，了解如何生成第一个语义内核 SDK 项目。 了解如何创建新项目，添加语义内核 SDK NuGet 包，以及添加对语义内核 SDK 的引用。 现在就开始吧！

1. 返回到 Visual Studio Code 项目。

1. 打开 **appsettings.json** 文件，并使用 Azure OpenAI 服务模型 ID、终结点和 API 密钥更新值。

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. 通过选择“终端” > “新建终端”来打开“终端”********。

1. 在终端中，运行以下命令以安装语义内核 SDK：

    `dotnet add package Microsoft.SemanticKernel --version 1.30.0`

1. 1. 在 **Program.cs** 文件中添加以下 `using` 指令：

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.ChatCompletion;
    using Microsoft.SemanticKernel.Connectors.OpenAI;
    ```

1. 在注释“**使用 Azure OpenAI 聊天补全创建内核生成器**”下，添加以下代码：

    ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    ```

1. 在注释“**生成内核**”下添加此代码以生成内核：

    ```c#
    // Build the kernel
    var kernel = builder.Build();
    ```

1. 在注释“**验证终结点并运行提示**”下，添加以下代码：

    ```c#
    // Verify the endpoint and run a prompt
    var result = await kernel.InvokePromptAsync("Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. 右键单击“**Starter**”文件夹，然后选择“**在集成终端中打开**”。

1. 在终端中，输入 `dotnet run` 以运行代码。

    验证是否看到了来自 Azure Open AI 模型的响应，其中包含全球最著名的 5 位音乐家。

    响应来自传递给内核的 Azure Open AI 模型。 语义内核 SDK 能够连接到大型语言模型 (LLM)，并运行提示。 注意从 LLM 接收响应的速度有多快。 语义内核 SDK 使构建智能应用程序变得简单高效。

确认响应后，可以移除验证码。

## 练习 2：创建自定义音乐库插件

在本练习中，你将为音乐库创建自定义插件。 创建可将歌曲添加到用户最近播放列表、获取最近播放的歌曲列表以及提供个性化歌曲推荐的函数。 你还将创建一个可以根据用户所在位置及其最近播放的歌曲推荐音乐会的函数。

完成练习估计所需时间****：15 分钟

### 任务 1：创建音乐库插件

在此任务中，你将创建一个插件，用于将歌曲添加到用户的最近播放列表，并获取最近播放的歌曲列表。 为简单起见，最近播放的歌曲存储在一个文本文件中。

1. 在“**插件**”文件夹中，打开 **MusicLibraryPlugin.cs** 文件

1. 在注释“**创建内核函数以获取最近播放歌曲**”下，添加内核函数修饰器：


    ```c#
    // Create a kernel function to get recently played songs
    [KernelFunction("GetRecentPlays")]
    public static string GetRecentPlays()
    ```

    `KernelFunction`修饰器声明本机函数。 对函数使用描述性名称，以便 AI 可以正确调用它。 用户最近播放的歌曲列表存储在名为“RecentlyPlayed.txt”的文本文件中。

1. 在注释**创建内核函数将歌曲添加到最近播放列表**下，添加内核函数修饰器：

    ```c#
    // Create a kernel function to add a song to the recently played list
    [KernelFunction("AddToRecentPlays")]
    public static string AddToRecentlyPlayed(string artist,  string song, string genre)
    ```

    现在，将此插件类添加到内核时，它将能够识别和调用函数。

1. 导航到 **Program.cs** 文件。

1. 在注释“**Import plugins to the kernel**”下添加以下代码：

    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    ```

1. 在注释“**创建提示执行设置**”下，添加以下代码以自动调用函数：

    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    使用此设置将允许内核自动调用函数，而无需在提示符中指定它们。

1. 在注释“**获取聊天补全服务**”下添加以下代码：

    ```c#
    // Get chat completion service.
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. 在注释“**创建帮助程序函数以等待并输出来自聊天补全服务的回复**”下，添加以下代码：

    ```c#
    // Create a helper function to await and output the reply from the chat completion service
    async Task GetAssistantReply() {
        ChatMessageContent reply = await chatCompletionService.GetChatMessageContentAsync(
            chatHistory,
            kernel: kernel,
            executionSettings: openAIPromptExecutionSettings
        );
        chatHistory.AddAssistantMessage(reply.ToString());
        Console.WriteLine(reply.ToString());
    }
    ```


1. 在“**向聊天添加系统消息**”注释下，添加以下代码：

    ```c#
    // Add system messages to the chat
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    chatHistory.AddSystemMessage("The listener has just played the song Danse by Tiara. It falls under these genres: French pop, electropop, pop.");
    await GetAssistantReply();
    ```

1. 通过在终端中输入 `dotnet run` 来运行代码。

    添加的系统消息提示应调用插件函数，并应看到以下输出：

    ```output
    Added 'Danse' to recently played
    ```

    如果打开 **Files/RecentlyPlayed.txt**，你应会看到新歌曲已添加到列表中。

### 任务 2：提供个性化歌曲推荐

在此任务中，你将创建一条提示，用于根据用户最近播放的歌曲为他们提供个性化的歌曲推荐。 该提示组合本机函数来生成歌曲推荐。 你还将根据提示创建一个函数，使该提示可重复使用。

1. 在 **Program.cs** 文件的注释“**使用提示创建歌曲建议器函数**”下，添加以下代码：

    ```c#
    // Create a song suggester function using a prompt
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

    在此代码中，你将从提示创建一个函数，并将其添加到内核插件。

1. 在注释 **使用用户提示调用歌曲建议器函数***下，添加以下代码：

    ```c#
    // Invoke the song suggester function with a prompt from the user
    chatHistory.AddUserMessage("What song should I play next?");
    await GetAssistantReply();
    ```

    现在，应用程序可以根据用户的请求自动调用插件函数。 让我们扩展此代码，以便根据用户的信息提供音乐会建议。

### 任务 3：提供个性化音乐会推荐

在此任务中，你将创建一个插件，让 LLM 根据用户最近播放的歌曲和所在位置推荐音乐会。

1. 在 **Program.cs** 文件的“**将插件导入内核**”注释下，导入音乐会插件：

    ```c#
    // Import plugins to the kernel 
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    ```

1. 在注释“**使用提示创建音乐会建议器函数**”下，添加以下代码：

    ```c#
    // Create a concert suggester function using a prompt
    var concertSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of the user's recently played songs:
        {{MusicLibraryPlugin.GetRecentPlays}}

        This is a list of upcoming concert details:
        {{MusicConcertsPlugin.GetConcerts}}

        Suggest an upcoming concert based on the user's recently played songs. 
        The user lives in {{$location}}, 
        please recommend a relevant concert that is close to their location.",
        functionName: "SuggestConcert",
        description: "Suggest a concert to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestConcertPlugin", [concertSuggesterFunction]);
    ```

    此函数提示会获取音乐库和即将推出的音乐会信息以及用户的位置，并提供建议。

1. 添加以下提示以调用新插件函数：

    ```c#
    // Invoke the concert suggester function with a prompt from the user
    chatHistory.AddUserMessage("Can you recommend a concert for me? I live in Washington");
    await GetAssistantReply();
    ```

1. 在终端中输入 `dotnet run`

    应会看到类似于以下响应的输出：

    ```output
    I recommend you attend the concert of Lisa Taylor. She will be performing in Seattle, Washington, USA on 2/22/2024. Enjoy the show!
    ```
    
    来自 LLM 的回复可能会有所不同。 尝试调整提示和位置，查看可以生成哪些其他结果。

现在，助手能够根据用户的输入自动执行不同的操作。 干得漂亮!

### 审阅

在本实验室中，你已创建一个 AI 助手，该助手可以管理用户的音乐库并提供个性化歌曲和音乐会推荐。 你使用语义内核 SDK 生成了该 AI 助手，并将其连接到大型语言模型 (LLM) 服务。 你为音乐库创建了自定义插件，启用了自动函数调用，使助手动态响应用户的输入。 祝贺你完成本实验！
