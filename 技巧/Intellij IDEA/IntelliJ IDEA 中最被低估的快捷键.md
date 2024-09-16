![Sue](media/Sue.jpg)

[Sue](https://blog.jetbrains.com/zh-hans/author/sue-zhu-jetbrains-com)

2022年11月29日

Read this post in other languages:

-   [English](https://blog.jetbrains.com/idea/2022/10/top-underrated-shortcuts/)
,-   [Français](https://blog.jetbrains.com/fr/idea/2022/12/selection-de-raccourcis-clavier-pour-intellij-idea-qui-gagnent-a-etre-connus/)
,-   [한국어](https://blog.jetbrains.com/ko/idea/2022/11/top-underrated-shortcuts/)

快捷键根植于 IntelliJ IDEA 的 DNA：所有可用操作均触手可及，即使是那些您认为不需要的操作。

我们经常与用户分享实用的快捷键，所以到现在为止，您很可能已经成为快捷键专家。 但信不信由你，可能会有一些连您都不知道的快捷键！

本文涵盖了 IntelliJ IDEA 支持的一些被低估的组合键。 快来了解它们并试一试！ 如果您还使用任何其他鲜为人知的快捷键，请在下面的评论中与我们分享。

# 被低估的快捷键

## ⌥ “x” 或 Alt+“x”：关闭所有非活动选项卡

如果您的编辑器中打开的选项卡过多，只需按一个键并点击一下即可解决此问题。 操作方法就是按 _⌥_ (macOS) 或 _Alt_ (Windows/Linux) 并点击活动选项卡上的“_x_”。 然后，IDE 将一次性关闭所有非活动状态的选项卡，而您当前正在使用的选项卡将保持打开状态。

![](media/CloseTabs.gif)

## ⇧⌘V 或 Shift+Ctrl+V：选择要粘贴的内容

IntelliJ IDEA 允许您在剪贴板中保留多个最近复制的条目，并借助 _⇧⌘V_ (macOS) 或 _Shift+Ctrl+V_ (Windows/Linux) 来粘贴其中的任意条目。 这意味着您不必担心最近复制的任意一段代码丢失。

![](media/CopyRecent.gif)

## ⌥/ 或 Alt+/：使用 Hippie 补全

您可能已经习惯了 IntelliJ IDEA 的代码补全机制，它会根据上下文提供合适的建议。 但在某些情况下，您可能只想重用在当前文件乃至项目中曾使用过的单词，而不考虑其上下文。

[_Hippie 补全_](https://www.jetbrains.com/help/idea/auto-completing-code.html#hippie_completion)就可以在这种情况中派上用场。 您可以通过将文本光标放置在所需位置并按 _⌥/_ (macOS) 或 _Alt+/_ (Windows/Linux) 来使用此功能。

![](media/HippieCompletion.gif)

## ⌥⌘L 或 Ctrl+Alt+L：重新格式化代码

在团队中工作时，你们常常需要共享统一的代码格式。 因此，如果项目中的格式不符合您所使用的约定，请根据您的设置选择一段要重新格式化的代码，然后按 _⌥⌘L_ (macOS) 或 _Ctrl+Alt+L_ (Windows/Linux)。

![](media/FormatCode.gif)

## ⌘F12 或 Ctrl+F12：按文件结构导航

如果您正在寻找另一种方式来快速访问文件中使用的方法，此快捷键非常适合您。 与文本搜索不同的是，您可以通过按 _⌘F12_ (macOS) 或 _Ctrl+F12_ (Windows/Linux) 打开您的文件结构。 然后，在弹出的结构窗口中，开始输入方法名称，使用上下箭头导航到正确的变体，然后按 Enter 键。 就是这样！

![](media/NavigateFileStructure.gif)

## ⌃G 或 Alt+J：选择多次出现的内容

如果您需要快速选择和编辑相同的代码段，那么此快捷键将是您的最佳选择。 要使用它，请将文本光标放置在所需符号上，然后按 _⌃G_ (macOS) 或 _Alt+J_ (Windows/Linux)。 然后，当代码元素高亮显示时，按住此快捷键即可在整个文件中移动。 您会注意到文本光标变成多个，并出现在每个高亮显示的代码元素实例旁。 如果您开始编辑高亮显示的代码，将在每个实例中自动更改此代码。

![](media/MultiplePiecesOfCode.gif)

## ⌘⇧↑/↓ 或 Alt +Shift+↑/↓：移动行

这个快捷键非常简单，却十分高效。 按 _⌘⇧_ (macOS) 或 _Alt+Shift_ (Windows/Linux)，然后使用上下箭头键即可将文本光标当前所在行移动到所需的位置。

但是，使用此快捷键时请小心，因为它可能会将行移到范围外。

如果您担心不小心将行移动到语句之外，请使用移动语句的快捷键：_⇧⌘↑/↓_ (macOS) 或 _Ctrl+Shift+↑/↓_ (Windows/Linux)。

![](media/MoveLines.gif)

## ⌘D 或 Ctrl+D：复制行

当您需要添加几行相似但具有不同形参的代码时，此快捷键非常实用。 要试用该快捷键，请将文本光标放置在需要复制的行上，然后按 _⌘D_ (macOS) 或 _Ctrl+D_ (Windows/Linux)。

![](media/DublicateLine.gif)

## ⌥ F3 或 Ctrl+F11：添加助记书签

要快速访问文件夹、文件、项目条目和代码行，您可以为它们添加书签并以数字 (0-9) 或字母 (A-Z) 标注。 分配后，IDE 将在间距中显示助记符。

要将此类书签添加到代码行，请将文本光标放置在该行上，然后按 _⌥ F3_ (macOS) 或 _Ctrl+F11_ (Windows)。 要为文件、软件包、文件夹或模块添加书签，请在 _Project_（项目）列表中选择所需的项目，并按相同的快捷键。

然后，在弹出窗口中选择要用作此书签标识符的数字或字母。 轻松搞定！

要跳转到书签，请按住 _^_ (macOS) 或 _Ctrl_ (Windows)，然后在键盘上按预定义的数字或字母。

![](media/MnemonicBookmark.gif)

## ^` 或 Ctrl+`：更改视图模式

IntelliJ IDEA 提供了多种视图模式以满足您在编码时的个人需求。 它们包括 _Presentation Mode_、_Distraction Free Mode_、_Full Screen_ 和 _Zen Mode_。 要激活这些模式，请执行以下操作：

-   按 _^`_ (macOS) 或 _Ctrl+`_ (Windows/Linux)。
-   在出现的 _Switch_（切换）对话框中，按 5。
-   使用上/下箭头键选择所需的模式或按相应的数字。

要返回默认视图，请重复相同的步骤。

_Distraction Free Mode_ 是我们最喜欢的视图之一。 它会通过隐藏所有 UI 元素并保持源代码居中，使您 100% 专注于任务。 一探究竟！

![](media/DistractionFree.gif)

## ⌘L 或 Ctrl+G：转到行:列

当您需要快速导航到代码中的内容并且知道其确切位置时，此快捷键非常有效。 按 _⌘L_ (macOS) 或 _Ctrl+G_ (Windows/Linux) 以打开 _Go to Line:Column_（转到行:列）对话框并输入所需的行或列编号。 您也可以输入二者，用冒号 (:) 分隔。 点击 OK（确定），然后就会转到所需的位置。

![](media/GoToLine.gif)

## ⌘-/+ 或 Ctrl -/+：收起和展开代码块

使用此快捷键可以更清晰地查看文件结构或快速导航到类的特定部分。 您可以按 _⌘-_ (macOS) 或 _Ctrl +_ – (Windows/Linux) 来收起所有可展开的代码块。 要重新展开，请同时按 _+_ 和 _⌘_ 或 _Ctrl_。

![](media/CollapseCode.gif)

## ⌘⇧A 或 Ctrl+Shift+A：查找将照片添加为背景的操作

此快捷键可在 IDE 中调用适用于不同情况的各种操作。 但是，今天我们只想分享其中一项操作 – 您可以使用该操作为您的 IDE 添加可爱的个性化风格。

您想在编码时看到自己最喜欢的人物、动物、事物或风景吗？ 这很简单 – 只需设置一张背景照片即可。

为此，请按 _⌘⇧A_ (macOS) 或 _Ctrl+Shift+A_（Windows 和 Linux）以调出 _Find Actions_（查找操作）对话框，然后输入“background image”（背景图像）。 然后，在出现的对话框中，选择照片的路径并根据您的喜好配置其他可用设置。 例如，您可以更改图像的不透明度。

要返回标准编辑器背景，请点击上一个对话框中的 _Clear and Close_（清除并关闭）按钮。

![](media/SetPhoto-2.gif)

# 总结

我们希望您喜欢我们精选的内容，愿您能发现一些此前从未听说过而又意想不到的实用快捷键。 如果您觉得还有其他非常实用却被人们忽视的快捷键，请在评论中告诉我们。 让我们一起成为键盘大师！

本博文英文原作者：

![Sue](media/Sue-1.jpg)

#### Irina Maryasova