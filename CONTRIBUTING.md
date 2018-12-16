# <a name="contributing-to-the-entity-framework-documentation"></a>参与实体框架文档

本文档介绍了参与[实体框架文档站点](https://docs.microsoft.com/ef)上托管的文章和代码示例的过程。 参与内容可简（更正拼写错误），可繁（编写新文章）。

## <a name="how-to-make-a-simple-correction-or-suggestion"></a>如何提出简单的更正或建议

文章作为 Markdown 文件存储在存储库中。 通过点击浏览器窗口右上角的“编辑”链接，可以在浏览器中对 Markdown 文件的内容进行简单更改。 （在狭窄的浏览器窗口中，可能需要展开“选项”栏，以查看“编辑”链接。）按照说明创建拉取请求 (PR)。 EF 团队将对拉取请求进行审核并接受相关请求或提出更改建议。

## <a name="how-to-make-a-more-complex-submission"></a>如何提出更复杂的提交

需要对 [Git 和 GitHub.com](https://guides.github.com/activities/hello-world/) 有基本的理解。

* 打开[问题](https://github.com/aspnet/EntityFramework.Docs/issues/new)，描述想要执行的操作，例如更改现有文章或创建一个新文章。 请等待 EF 团队批准后再投入时间参与进来。
* 对 [aspnet/EntityFramework.Docs](https://github.com/aspnet/EntityFramework.Docs/) 存储库进行分叉，并为所作更改创建分支。
* 将拉取请求 (PR) 与更改一起提交到主干。
* 对 PR 请求反馈进行响应。

## <a name="markdown-syntax"></a>Markdown 语法

文章采用 [DocFx 风格的 Markdown](http://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html) 编写，它是 [GitHub 风格的 Markdown (GFM)](https://guides.github.com/features/mastering-markdown/) 的超集。 有关 EF 文档中常用的 UI 功能的 DFM 语法示例，请参阅 .NET Core 存储库风格指南中的[元数据和 Markdown 模板](https://github.com/dotnet/docs/blob/master/styleguide/template.md)。 

## <a name="folder-structure-conventions"></a>文件夹结构约定

图像和其他静态内容存储在站点的每个区域/文件夹中的 `_static` 文件夹。

代码示例存储在 `samples` 根文件夹。 它们采用模仿文档结构（位于 `entity-framework` 根文件夹下）的文件夹结构。

## <a name="code-snippets"></a>代码片段

文章经常使用代码片段来说明要点。 DFM 允许将代码复制到 Markdown 文件或引用单独的代码文件。 请尽可能使用单独的代码文件，以最大限度地减少代码中出错的可能性。 代码文件应存储在使用示例项目的上面所述的文件夹结构的存储库中。

下面是一些 [DFM 代码片段语法](http://dotnet.github.io/docfx/spec/docfx_flavored_markdown.html#code-snippet)示例。

将整个代码文件呈现为代码片段：

``` none
[!code-csharp[Main](../../../samples/core/saving/Program.cs)]
```

使用行号将文件的一部分呈现为代码片段：

``` none
[!code-csharp[Main](../../../samples/core/saving/Program.cs?range=1-10]
```

有关 C# 代码片段，请参阅 [C# 区域](https://msdn.microsoft.com/library/9a1ybwek.aspx)。 请尽可能使用区域而不是行号，因为代码文件中的行号往往会更改，并与 Markdown 中引用的行号不同步。 可以嵌套 C# 区域，如果引用外部区域，内部 `#region` 和 `#endregion` 指令不会在代码片段中呈现。

呈现名为 “snippet_Example” 的 C# 区域：

``` none
[!code-csharp[Main](../../../samples/core/saving/Program.cs?name=snippet_Example)]
```

突出显示呈现的代码段中选定的行（通常呈现为黄色背景）：

``` none
[!code-csharp[Main](../../../samples/core/saving/Program.cs?name=snippet_Example&highlight=1-3,10,20-25)]
```

## <a name="test-your-changes-with-docfx"></a>使用 DocFX 测试更改

使用 [DocFX 命令行工具](https://dotnet.github.io/docfx/tutorial/docfx_getting_started.html#2-use-docfx-as-a-command-line-tool)测试更改，这将会创建站点的本地托管版本。 DocFX 不呈现为 docs.microsoft.com 创建的样式和站点扩展。

DocFX 需要使用 .NET Framework（对于 Windows）或 Mono（对于 Linux 或 macOS）。

### <a name="windows-instructions"></a>Windows 说明

* 从 [DocFX 发布](https://github.com/dotnet/docfx/releases)下载并解压缩 “docfx.zip”。
* 将 DocFX 添加到路径。
* 在命令行窗口中，导航到克隆存储库（其中包含“docfx.json”文件），然后运行以下命令：

   ``` console
   docfx -t default --serve
   ```

* 在浏览器中导航到 `http://localhost:8080`。

### <a name="mono-instructions"></a>Mono 说明

* 使用 Homebrew 安装 Mono - `brew install mono`。
* 下载 [DocFX 的最新版本](https://github.com/dotnet/docfx/releases/tag/v2.7.2)。
* 提取到 `\bin\docfx`。
* 为 “docfx” 创建一个别名：

  ``` console
  function docfx {
    mono $HOME/bin/docfx/docfx.exe
  }

  function docfx-serve {
    mono $HOME/bin/docfx/docfx.exe serve _site
  }
  ```

* 在克隆存储库中运行“docfx”以生成站点，并在 `http://localhost:8080` 上运行“docfx-serve”以查看站点。

## <a name="voice-and-tone"></a>语气和语调

我们的目标是编写被广泛受众所理解的易懂文档。 为此，我们编写了写作风格指南，请参与者遵守。 有关详细信息，请参阅 .NET Core 存储库中的 [语气和语调指南](https://github.com/dotnet/docs/blob/master/styleguide/voice-tone.md)。
