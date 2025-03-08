---
lab:
  title: 实验室：使用 Azure OpenAI 和语义内核 SDK 开发 AI 代理
  module: 'Module 01: Build your kernel'
---

# 实验室：创建 AI 音乐推荐代理
# 学生实验室手册

在本实验中，你将创建 AI 代理的代码，该代理可以管理用户的音乐库并提供个性化的歌曲和音乐会推荐。 你将使用语义内核 SDK 生成该 AI 代理，并将其连接到大型语言模型 (LLM) 服务。 使用语义内核 SDK，能够创建一个可与 LLM 服务交互并为用户提供个性化推荐的智能应用程序。

## 实验室场景

你是某个国际音频流媒体服务的开发人员。 你的任务是将该服务与 AI 集成，以便为用户提供更具个性化的体验。 AI 能够根据用户的听歌历史记录和偏好推荐歌曲以及即将举办的艺术家巡演活动。 你决定使用语义内核 SDK 生成一个可与大型语言模型 (LLM) 服务交互的 AI 代理。

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

1. 若要创建内核，请将以下代码添加到“**Program.cs**”文件中：
    
    ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);

    // Build the kernel
    var kernel = builder.Build();
    ```

1. 若要验证内核和终结点是否正常工作，请输入以下代码：

    ```c#
    var result = await kernel.InvokePromptAsync("Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. 右键单击“**Starter**”文件夹，然后选择“**在集成终端中打开**”。

1. 在终端中，输入 `dotnet run` 以运行代码。

    验证是否看到了来自 Azure Open AI 模型的响应，其中包含全球最著名的 5 位音乐家。

    响应来自传递给内核的 Azure Open AI 模型。 语义内核 SDK 能够连接到大型语言模型 (LLM)，并运行提示。 注意从 LLM 接收响应的速度有多快。 语义内核 SDK 使构建智能应用程序变得简单高效。

## 练习 2：创建自定义音乐库插件

在本练习中，你将为音乐库创建自定义插件。 创建可将歌曲添加到用户最近播放列表、获取最近播放的歌曲列表以及提供个性化歌曲推荐的函数。 你还将创建一个可以根据用户所在位置及其最近播放的歌曲推荐音乐会的函数。

完成练习估计所需时间****：15 分钟

### 任务 1：创建音乐库插件

在此任务中，你将创建一个插件，用于将歌曲添加到用户的最近播放列表，并获取最近播放的歌曲列表。 为简单起见，最近播放的歌曲存储在一个文本文件中。

1. 在“**Plugins**”文件夹中，创建新文件“**MusicLibraryPlugin.cs**”

    首先，创建一些快速函数来获取歌曲并将其添加到用户的“最近播放”列表中。

1. 输入以下代码：

    ```c#
    using System.Text.Json;
    using System.Text.Json.Nodes;
    using Microsoft.SemanticKernel;

    public class MusicLibraryPlugin
    {
        [KernelFunction("GetRecentPlays")]
        public static string GetRecentPlays()
        {
            string content = File.ReadAllText($"Files/RecentlyPlayed.txt");
            return content;
        }
    }
    ```

    在此代码中，你使用 `KernelFunction` 修饰器来声明本机函数。 对函数使用描述性名称，以便 AI 可以正确调用它。 用户最近播放的歌曲列表存储在名为“RecentlyPlayed.txt”的文本文件中。 接下来可以添加用于将歌曲添加到列表中的代码。

1. 将以下代码添加到 `MusicLibraryPlugin` 类：

    ```c#
    [KernelFunction("AddToRecentPlays")]
    public static string AddToRecentlyPlayed(string artist,  string song, string genre)
    {
        // Read the existing content from the file
        string filePath = "Files/RecentlyPlayed.txt";
        string jsonContent = File.ReadAllText(filePath);
        var recentlyPlayed = (JsonArray) JsonNode.Parse(jsonContent)!;

        var newSong = new JsonObject
        {
            ["title"] = song,
            ["artist"] = artist,
            ["genre"] = genre
        };

        // Insert the new song
        recentlyPlayed.Insert(0, newSong);
        File.WriteAllText(filePath, JsonSerializer.Serialize(recentlyPlayed,
            new JsonSerializerOptions { WriteIndented = true }));

        return $"Added '{song}' to recently played";
    }
    ```

    在此代码中，你创建一个函数，接受“艺术家”、“歌曲”和“流派”作为字符串。 “RecentlyPlayed.txt”文件包含用户最近播放的歌曲的 json 格式列表。 此代码从文件中读取现有内容，对其进行分析，然后将新歌曲添加到列表中。 已更新的列表此后会写回到文件中。

1. 使用以下代码更新 Program.cs 文件****：

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();

    // Get chat completion service.
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

    // Create a chat history object
    ChatHistory chatHistory = [];
    ```

    在此代码中，将插件导入内核，并添加聊天完成设置。

1. 添加以下提示以调用插件：

    ```c#
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    chatHistory.AddSystemMessage("The listener has just played the song Danse by Tiara. It falls under these genres: French pop, electropop, pop.");

    ChatMessageContent reply = await chatCompletionService.GetChatMessageContentAsync(
        chatHistory,
        kernel: kernel
    );
    Console.WriteLine(reply.ToString());
    chatHistory.AddAssistantMessage(reply.ToString());
    ```

1. 通过在终端中输入 `dotnet run` 来运行代码。

    应会看到以下输出：

    ```output
    Added 'Danse' to recently played
    ```

    如果打开“Files/RecentlyPlayed.txt”，你应会看到新歌曲已添加到列表中。

### 任务 2：提供个性化歌曲推荐

在此任务中，你将创建一条提示，用于根据用户最近播放的歌曲为他们提供个性化的歌曲推荐。 该提示组合本机函数来生成歌曲推荐。 你还将根据提示创建一个函数，使该提示可重复使用。

1. 在 **MusicLibraryPlugin.cs** 文件中，添加以下函数：

    ```c#
    [KernelFunction("GetMusicLibrary")]
    public static string GetMusicLibrary()
    {
        string dir = Directory.GetCurrentDirectory();
        string content = File.ReadAllText($"Files/MusicLibrary.txt");
        return content;
    }
    ```

    此函数从名为“MusicLibrary.txt”的文件中读取可用音乐列表。 该文件包含可供用户播放的歌曲的 json 格式列表。

1. 使用以下代码更新 Program.cs 文件****：

    ```c#
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    
    string prompt = @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next";

    var result = await kernel.InvokePromptAsync(prompt);
    Console.WriteLine(result);
    ```

    首先，可以移除将歌曲追加到列表中的代码。 然后，将本机插件功能与语义提示相结合。 本机函数将能够检索大型语言模型 (LLM) 无法自行访问的用户数据，而 LLM 则能够基于文本输入生成歌曲建议。

1. 若要测试代码，请在终端中输入 `dotnet run`。

    你应该会看到类似于以下输出的响应：

    ```output 
    Based on the user's recently played music, a suggested song to play next could be "Sabry Aalil" by Yasemin since the user seems to enjoy pop and Egyptian pop music.
    ```

    >[!NOTE] 建议的歌曲可能与此处所示的歌曲不同。

1. 修改代码以根据提示创建函数：

    ```c#
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

1. 添加以下代码以自动调用函数：

    ```c#
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };

    chatHistory.AddUserMessage("What song should I play next?");

    reply = await chatCompletionService.GetChatMessageContentAsync(
        chatHistory,
        kernel: kernel,
        executionSettings: openAIPromptExecutionSettings
    );
    Console.WriteLine(reply.ToString());
    chatHistory.AddAssistantMessage(reply.ToString());
    ```

    在此代码中，你将创建用于启用自动函数调用的设置。 然后，添加将调用函数并检索回复的提示。

### 任务 3：提供个性化音乐会推荐

在此任务中，你将创建一个插件，让 LLM 根据用户最近播放的歌曲和所在位置推荐音乐会。

1. 在 **Program.cs** 文件中，将音乐会插件添加到内核：

    ```c#
    var kernel = builder.Build();    
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    ```

1. 添加代码以根据提示创建函数：

    ```c#
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
    chatHistory.AddUserMessage("Can you recommend a concert for me? I live in Washington");

    reply = await chatCompletionService.GetChatMessageContentAsync(
        chatHistory,
        kernel: kernel,
        executionSettings: openAIPromptExecutionSettings
    );
    Console.WriteLine(reply.ToString());
    chatHistory.AddAssistantMessage(reply.ToString());
    ```

1. 在终端中输入 `dotnet run`

    应会看到类似于以下响应的输出：

    ```output
    I recommend you attend the concert of Lisa Taylor. She will be performing in Seattle, Washington, USA on 2/22/2024. Enjoy the show!
    ```
    
    来自 LLM 的回复可能会有所不同。 尝试调整提示和位置，查看可以生成哪些其他结果。

现在，代理能够根据用户的输入执行不同的操作。 干得漂亮!

### 审阅

在本实验中，你已创建一个 AI 代理，该代理可以管理用户的音乐库并提供个性化的歌曲和音乐会推荐。 你使用语义内核 SDK 生成了该 AI 代理，并将其连接到了大型语言模型 (LLM) 服务。 你为音乐库创建了自定义插件，启用了自动函数调用，使代理动态响应用户的输入。 祝贺你完成本实验！
