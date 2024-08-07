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
* 使用 Handlebars 规划器自动执行任务

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

1. 在“概述”页上，选择“转到 Azure OpenAI Studio”。********

:::image type="content" source="../media/model-deployments.png" alt-text="Azure OpenAI 部署页的屏幕截图。":::

1. 依次选择“创建新部署”、“部署模型”。********

1. 在“选择模型”下，选择“gpt-35-turbo-16k”。********

    使用默认模型版本

1. 输入部署的名称

1. 部署完成后，导航回 Azure OpenAI 资源。

1. 在“资源管理”下，转到“密钥和终结点”********。

    你将在下一个任务中使用此处提供的数据来生成内核。 请记得将密钥保密并妥善保存！

### 任务 2：生成内核

在本练习中，了解如何生成第一个语义内核 SDK 项目。 了解如何创建新项目，添加语义内核 SDK NuGet 包，以及添加对语义内核 SDK 的引用。 现在就开始吧！

1. 返回到 Visual Studio Code 项目。

1. 通过选择“终端” > “新建终端”来打开“终端”********。

1. 在终端中，运行以下命令以安装语义内核 SDK：

    `dotnet add package Microsoft.SemanticKernel --version 1.2.0`

1. 若要创建内核，请将以下代码添加到“Program.cs”文件中：
    
    ```c#
    using Microsoft.SemanticKernel;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    ```

    务必将占位符替换为 Azure 资源中的值。

1. 若要验证内核和终结点是否正常工作，请输入以下代码：

    ```c#
    var result = await kernel.InvokePromptAsync(
        "Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. 输入 `dotnet run` 以运行代码，检查是否看到了来自 Azure Open AI 模型的响应，其中包含全球最著名的 5 位音乐家。

    响应来自传递给内核的 Azure Open AI 模型。 语义内核 SDK 能够连接到大型语言模型 (LLM)，并运行提示。 注意从 LLM 接收响应的速度有多快。 语义内核 SDK 使构建智能应用程序变得简单高效。

## 练习 2：创建自定义音乐库插件

在本练习中，你将为音乐库创建自定义插件。 创建可将歌曲添加到用户最近播放列表、获取最近播放的歌曲列表以及提供个性化歌曲推荐的函数。 你还将创建一个可以根据用户所在位置及其最近播放的歌曲推荐音乐会的函数。

完成练习估计所需时间****：15 分钟

### 任务 1：创建音乐库插件

在此任务中，你将创建一个插件，用于将歌曲添加到用户的最近播放列表，并获取最近播放的歌曲列表。 为简单起见，最近播放的歌曲存储在一个文本文件中。

1. 在“Lab01-Project”目录中创建一个新文件夹并将其命名为“Plugins”。

1. 在“Plugins”文件夹中，创建新文件“MusicLibrary.cs”

    首先，创建一些快速函数来获取歌曲并将其添加到用户的“最近播放”列表中。

1. 输入以下代码：

    ```c#
    using System.ComponentModel;
    using System.Text.Json;
    using System.Text.Json.Nodes;
    using Microsoft.SemanticKernel;

    public class MusicLibraryPlugin
    {
        [KernelFunction, 
        Description("Get a list of music recently played by the user")]
        public static string GetRecentPlays()
        {
            string content = File.ReadAllText($"Files/RecentlyPlayed.txt");
            return content;
        }
    }
    ```

    在此代码中，你使用 `KernelFunction` 修饰器来声明本机函数。 还将使用 `Description` 修饰器来添加函数功能的说明。 用户最近播放的歌曲列表存储在名为“RecentlyPlayed.txt”的文本文件中。 接下来可以添加用于将歌曲添加到列表中的代码。

1. 将以下代码添加到 `MusicLibraryPlugin` 类：

    ```c#
    [KernelFunction, Description("Add a song to the recently played list")]
    public static string AddToRecentlyPlayed(
        [Description("The name of the artist")] string artist, 
        [Description("The title of the song")] string song, 
        [Description("The song genre")] string genre)
    {
        // Read the existing content from the file
        string filePath = "Files/RecentlyPlayed.txt";
        string jsonContent = File.ReadAllText(filePath);
        var recentlyPlayed = (JsonArray) JsonNode.Parse(jsonContent);

        var newSong = new JsonObject
        {
            ["title"] = song,
            ["artist"] = artist,
            ["genre"] = genre
        };

        recentlyPlayed.Insert(0, newSong);
        File.WriteAllText(filePath, JsonSerializer.Serialize(recentlyPlayed,
            new JsonSerializerOptions { WriteIndented = true }));

        return $"Added '{song}' to recently played";
    }
    ```

    在此代码中，你创建一个函数，接受“艺术家”、“歌曲”和“流派”作为字符串。 除了函数的 `Description` 之外，你还添加输入变量的说明。 “RecentlyPlayed.txt”文件包含用户最近播放的歌曲的 json 格式列表。 此代码从文件中读取现有内容，对其进行分析，然后将新歌曲添加到列表中。 已更新的列表此后会写回到文件中。

1. 使用以下代码更新“Program.cs”文件：

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();

    var result = await kernel.InvokeAsync(
        "MusicLibraryPlugin", 
        "AddToRecentlyPlayed", 
        new() {
            ["artist"] = "Tiara", 
            ["song"] = "Danse", 
            ["genre"] = "French pop, electropop, pop"
        }
    );
    
    Console.WriteLine(result);
    ```

    在此代码中，使用 `ImportPluginFromType` 将 `MusicLibraryPlugin` 导入到内核。 然后使用要调用的插件名称和函数名称调用 `InvokeAsync`。 还可以传入艺术家、歌曲和流派作为参数。

1. 通过在终端中输入 `dotnet run` 来运行代码。

    应会看到以下输出：

    ```output
    Added 'Danse' to recently played
    ```

    如果打开“Files/RecentlyPlayed.txt”，你应会看到新歌曲已添加到列表中。

### 任务 2：提供个性化歌曲推荐

在此任务中，你将创建一条提示，用于根据用户最近播放的歌曲为他们提供个性化的歌曲推荐。 该提示组合本机函数来生成歌曲推荐。 你还将根据提示创建一个函数，使该提示可重复使用。

1. 在 `MusicLibraryPlugin.cs` 文件中，添加以下函数：

    ```c#
    [KernelFunction, Description("Get a list of music available to the user")]
    public static string GetMusicLibrary()
    {
        string dir = Directory.GetCurrentDirectory();
        string content = File.ReadAllText($"Files/MusicLibrary.txt");
        return content;
    }
    ```

    此函数从名为“MusicLibrary.txt”的文件中读取可用音乐列表。 该文件包含可供用户播放的歌曲的 json 格式列表。

1. 使用以下代码更新“Program.cs”文件：

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    
    string prompt = @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next";

    var result = await kernel.InvokePromptAsync(prompt);
    Console.WriteLine(result);
    ```

在此段代码中，你将本机函数与语义提示相结合。 本机函数将能够检索大型语言模型 (LLM) 无法自行访问的用户数据，而 LLM 则能够基于文本输入生成歌曲建议。

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

    var result = await kernel.InvokeAsync(songSuggesterFunction);
    Console.WriteLine(result);
    ```

    在此代码中，你将根据提示创建一个函数，用于向用户建议歌曲。 然后将它添加到内核插件。 最后，告知内核运行该函数。

### 任务 3：提供个性化音乐会推荐

在此任务中，你将创建一个用于检索即将举办的音乐会详细信息的插件。 你还将创建一个插件，让 LLM 根据用户最近播放的歌曲和所在位置推荐音乐会。

1. 在“Plugins”文件夹中，创建一个名为“MusicConcertPlugin.cs”的新文件

1. 在“MusicConcertsPlugin”文件中，添加以下代码：

    ```c#
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    public class MusicConcertsPlugin
    {
        [KernelFunction, Description("Get a list of upcoming concerts")]
        public static string GetConcerts()
        {
            string content = File.ReadAllText($"Files/ConcertDates.txt");
            return content;
        }
    }
    ```

    在此代码中，你将创建一个函数，用于从名为“ConcertDates.txt”文件读取即将举办的音乐会列表。 该文件包含即将举办的音乐会的 json 格式列表。 接下来，需要创建一条提示，让 LLM 推荐音乐会。

1. 在“Prompts”文件夹中，创建一个名为“SuggestConcert”的新文件夹

1. 在“SuggestConcert”文件夹中创建一个“config.json”文件，其中包含以下内容：

    ```json
    {
        "schema": 1,
        "type": "completion",
        "description": "Suggest a concert to the user",
        "execution_settings": {
            "default": {
                "max_tokens": 4000,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "location",
                "description": "The user's location",
                "required": true
            }
        ]
    }
    ```

1. 在“SuggestConcert”文件夹中创建一个“skprompt.txt”文件，其中包含以下内容：

    ```output
    This is a list of the user's recently played songs:
    {{MusicLibraryPlugin.GetRecentPlays}}

    This is a list of upcoming concert details:
    {{MusicConcertsPlugin.GetConcerts}}

    Suggest an upcoming concert based on the user's recently played songs. 
    The user lives in {{$location}}, 
    please recommend a relevant concert that is close to their location.
    ```

    此提示有助于 LLM 筛选用户的输入，并从文本中仅检索目的地。 接下来，调用规划器来创建一个计划，将插件组合在一起以实现目标。

1. 打开“Program.cs”文件并使用以下代码更新它：

    ```c#
    var kernel = builder.Build();    
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
    // code omitted for brevity
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);

    string location = "Redmond WA USA";
    var result = await kernel.InvokeAsync<string>(prompts["SuggestConcert"],
        new() {
            { "location", location }
        }
    );
    Console.WriteLine(result);
    ```

1. 在终端中输入 `dotnet run`

    应会看到类似于以下响应的输出：

    ```output
    Based on the user's recently played songs and their location in Redmond WA USA, a relevant concert recommendation would be the upcoming concert of Lisa Taylor in Seattle WA, USA on February 22, 2024. Lisa Taylor is an indie-folk artist, and her music genre aligns with the user's recently played songs, such as "Loanh Quanh" by Ly Hoa. Additionally, Seattle is close to Redmond, making it a convenient location for the user to attend the concert.
    ```

    尝试调整提示和位置，查看可以生成哪些其他结果。

## 练习 3：使用 Handlebars 计划自动推荐

当你需要执行多个步骤来完成任务时，Handlebars 规划器非常有用。 规划器使用 AI 来选择已注册到内核的插件，并将其组合成一系列步骤来实现目标。 在本练习中，你将使用 Handlebars 规划器生成计划模板，并使用它来自动给出推荐。

完成练习估计所需时间****：10 分钟

### 任务 1：生成计划模板

在此任务中，你将使用 Handlebars 规划器生成计划模板。 该计划模板用于根据用户的输入自动给出推荐。

1. 通过在终端中输入以下内容来安装 Handlebars 规划器：

    `dotnet add package Microsoft.SemanticKernel.Planners.Handlebars --version 1.2.0-preview`

    接下来，替换 SuggestConcert 提示并改用 Handlebars 规划器来执行任务。

1. 在“Program.cs”文件中，将代码更新为以下内容：

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.Planning.Handlebars;
    
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");

    #pragma warning disable SKEXP0060
    var planner = new HandlebarsPlanner(new HandlebarsPlannerOptions() { AllowLoops = true });

    string location = "Redmond WA USA";
    string goal = @$"Based on the user's recently played music, suggest a 
        concert for the user living in ${location}";

    var plan = await planner.CreatePlanAsync(kernel, goal);
    var result = await plan.InvokeAsync(kernel);

    Console.WriteLine($"{result}");
    ```

    >[!NOTE] 由于 Handlebars 包目前处于预览状态，因此可能需要禁止编译器警告来运行代码。

1. 在终端中输入 `dotnet run`

    你应该会看到类似于以下输出的响应：

    ```output
    Based on the user's recently played songs, the artist "Mademoiselle" has an upcoming concert in Seattle WA, USA on February 22, 2024, which is close to Redmond WA. Therefore, the recommended concert for the user would be Mademoiselle's concert in Seattle.
    ```

    接下来，更改代码以输出 Handlebars 计划模板。

1. 在“Program.cs”文件中，将代码更新为以下内容：

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, goal);
    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

    现在，你可以看到生成的计划。 接下来，修改计划以包含歌曲推荐或将歌曲添加到用户的最近播放列表。

1. 使用以下片段扩展代码：

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, 
        @$"If add song:
        Add a song to the user's recently played list.
        
        If concert recommendation:
        Based on the user's recently played music, suggest a concert for 
        the user living in a given location.

        If song recommendation:
        Suggest a song from the music library to the user based on their 
        recently played songs.");

    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

1. 在终端中输入 `dotnet run` 以查看所创建计划的输出。

    你应会看到类似于以下输出的模板：

    ```output
    Plan:
    {{!-- Step 1: Identify Key Values --}}
    {{set "location" location}}
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" concertRecommendation}}

    {{!-- Step 2: Use the Right Helpers --}}
    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set "songAdded" (MusicLibraryPlugin-AddToRecentlyPlayed artist=artist song=song genre=genre)}}  
        {{json songAdded}}
    {{/if}}

    {{#if concertRecommendation}}
        {{set "concertSuggested" (Prompts-SuggestConcert location=location recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json concertSuggested}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "songSuggested" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json songSuggested}}
    {{/if}}

    {{!-- Step 3: Output the Result --}}
    {{json "Goal achieved"}}
    ```

     请注意 `{{#if ...}}` 语法。 此语法充当 Handlebars 规划器可以使用的条件语句，类似于 C# 中的传统 `if`-`else` 块。 `if` 语句必须以 `{{/if}}` 结尾。

    接下来，使用这个生成的模板创建你自己的 Handlebars 计划。 

1. 在“文件存储”目录中，使用以下文本创建名为“handlebarsTemplate.txt”的新文件：

    ```output
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" songRecommendation}}

    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set addedSong (MusicLibraryPlugin-AddToRecentlyPlayed artist song genre)}}  
        Output The following content, do not make any modifications:
        {{json addedSong}}     
    {{/if}}

    {{#if concertRecommendation}}
        {{set "location" location}}
        {{set "concert" (Prompts-SuggestConcert location)}}
        Output The following content, do not make any modifications:
        {{json concert}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "recentlyPlayedSongs" (MusicLibraryPlugin-GetRecentPlays)}}
        {{set "musicLibrary" (MusicLibraryPlugin-GetMusicLibrary)}}
        {{set "song" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs musicLibrary)}}
        Output The following content, do not make any modifications:
        {{json song}}
    {{/if}}
    ```

    在此模板中，你将向 LLM 添加一条指令，告诉它不要执行任何文本生成，以确保输出由插件严格管理。 现在让我们试用该模板！

### 任务 2：使用 Handlebars 规划器自动给出推荐

在此任务中，你将基于 Handlebars 计划模板创建一个函数，并使用该函数根据用户的输入自动给出推荐。

1. 通过修改现有代码移除 handlebars 计划：

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.PromptTemplates.Handlebars;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");
    
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"Based on the user's recently played music:
        {{$recentlyPlayedSongs}}
        recommend a song to the user from the music library:
        {{$musicLibrary}}",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

1. 添加用于读取模板文件并创建函数的代码：

    ```c#
    string template = File.ReadAllText($"Files/HandlebarsTemplate.txt");

    var handlebarsPromptFunction = kernel.CreateFunctionFromPrompt(
        new() {
            Template = template,
            TemplateFormat = "handlebars"
        }, new HandlebarsPromptTemplateFactory()
    );
    ```

    在此代码中，你将一个 `Template` 对象连同 `TemplateFormat` 一起传递给内核方法 `CreateFunctionFromPrompt`。 `CreateFunctionFromPrompt` 还接受一个 `IPromptTemplateFactory` 类型，它告知内核如何分析给定模板。 由于你使用的是 Handlebars 模板，你使用的就是 `HandlebarsPromptTemplateFactory` 类型。

    接下来，让我们使用一些参数来运行函数并查看结果！

1. 将以下代码添加到 `Program.cs` 文件：

    ```c#
    string location = "Redmond WA USA";
    var templateResult = await kernel.InvokeAsync(handlebarsPromptFunction,
        new() {
            { "location", location },
            { "concertRecommendation", true },
            { "songRecommendation", false },
            { "addSong", false },
            { "artist", "" },
            { "song", "" },
            { "genre", "" }
        });

    Console.WriteLine(templateResult);
    ```

1. 在终端中输入 `dotnet run` 以查看规划器模板的输出。

    你应该会看到类似于以下输出的响应：

    ```output
    Based on the user's recently played songs, Ly Hoa seems to be a relevant artist. The closest concert to Redmond WA, USA would be in Portland OR, USA on April 16th, 2024.  
    ```

    该提示能够根据用户最近播放的音乐列表及其所在位置为用户推荐音乐会。 你还可以尝试将其他变量设置为 true，看看会发生什么情况！

现在，你的代码能够根据用户的输入执行不同的操作。 干得漂亮!

### 审阅

在本实验中，你已创建一个 AI 代理，该代理可以管理用户的音乐库并提供个性化的歌曲和音乐会推荐。 你使用语义内核 SDK 生成了该 AI 代理，并将其连接到了大型语言模型 (LLM) 服务。 你为音乐库创建了自定义插件，使用 Handlebars 规划器自动给出了推荐，并基于 Handlebars 计划模板创建了一个函数，以根据用户的输入自动给出推荐。 祝贺你完成本实验！
