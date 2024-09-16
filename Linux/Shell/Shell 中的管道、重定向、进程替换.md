# Shell 中的管道、重定向、进程替换

发表于 2021/09/15 更新于 2022/12/30

作者 _[Nichts Hsu](https://nihil.cc/)_ _13 分钟_阅读

## 管道[](https://nihil.cc/posts/shell-redirect/#%E7%AE%A1%E9%81%93)

相信绝大部分 Linux 用户对管道（Pipe）都不陌生，其语法是：

`   |   | |---| ||   ```   |   | |---| |command1 \| command2|  它的作用是将 `command1` 的标准输出作为 `command2` 的标准输入来使用。一个非常典型的例子是 `grep`，相信大部分人都有使用过形如 `command | grep xxx` 的命令。  > 管道的两个命令分别在两个不同的子 shell 中执行，并且没有规定哪个子 shell 会被保留继续使用  ## 重定向[](https://nihil.cc/posts/shell-redirect/#%E9%87%8D%E5%AE%9A%E5%90%91)   ``

重定向（Redirect）用于将输入或输出重定向到文件或流。在开始本节之前，我们需要先了解三个特殊的 Linux 文件描述符，它们分别是 STDIN（0），STDOUT（1），STDERR（2），它们会起到一些特殊的作用。

### 输出重定向[](https://nihil.cc/posts/shell-redirect/#%E8%BE%93%E5%87%BA%E9%87%8D%E5%AE%9A%E5%90%91)

#### 基本形式[](https://nihil.cc/posts/shell-redirect/#%E5%9F%BA%E6%9C%AC%E5%BD%A2%E5%BC%8F)

`   |   | |---| ||   ```   |   | |---| |command > output_file|  将 `command` 的输出保存在文件 `output_file` 中而不是输出到终端。若文件不存在则创建该文件，若文件已有内容则覆盖原有内容。  例如，[在 Ubuntu 通过命令行安装 vscode 时](https://code.visualstudio.com/docs/setup/linux)，就需要你执行这样一条命令：   ```   |   | |---| ||   ```   |   | |---| |echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list|  它就是将 echo 后面这一段字符串输入到文件 `/etc/apt/sources.list.d/vscode.list` 中。  #### 追加形式[](https://nihil.cc/posts/shell-redirect/#%E8%BF%BD%E5%8A%A0%E5%BD%A2%E5%BC%8F)   ```   |   | |---| ||   ```   |   | |---| |command >> output_file|  使用两个 `>` 符号用以表示追加模式，这个模式下若文件已有内容，则在原有内容的后面追加新内容，而不是覆盖。  #### 使用文件描述符[](https://nihil.cc/posts/shell-redirect/#%E4%BD%BF%E7%94%A8%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6)   ``

通常，我们使用 `> output_file` 时，它实际上是指 `1> output_file`，这意味着将 `STDOUT` 重定向到文件 `output_file`。如果我们要将 `STDERR` 也重定向到 `output_file`，除了多写一个 `2> output_file` 外，也可以将 `STDERR` 重定向到 `STDOUT`：

`   |   | |---| ||   ```   |   | |---| |command > output_file 2>&1|  注意到**在 `>` 和 `1` 之间使用了一个 `&`**，用以表示 `1` 是文件描述符而不是一个名为 `1` 的文件。  举个例子，我们在使用 `nohup xxx &` 命令启用一个后台程序时，通常会将其输出重定向到 `/dev/null` 以避免污染当前终端：   ```   |   | |---| ||   ``   |   | |---| |nohup command > /dev/null 2>&1 &|  > 重定向的顺序很重要  如果写成：   ``   |   | |---| ||   ```   |   | |---| |command 2>&1 > output_file|  仅仅只有 `STDOUT` 指向 `output_file`，而 `STDERR` 仍然指向重定向之前的 `STDOUT`。  由于 `2>&1` 是一个非常常用的重定向，因此大多数 shell 都提供了下面这种简单写法：   ```   |   | |---| ||   ```   |   | |---| |command &> output_file|  这意味着同时将 `STDOUT` 和 `STDERR` 重定向到 `output_file`，语义上与 `command > output_file 2>&1` 完全相同。另一方面，`>& output_file` 也是等义的写法，但是不被提倡。  除此之外，也可以使用符号 `-` 来表示关闭通道：   ```   |   | |---| ||   ```   |   | |---| |command >&-|  这意味着关闭 `STDOUT`，只有 `STDERR` 保留输出。  文件描述符对追加重定向也适用：   ```   |   | |---| ||   ``|   | |---| |command 1>> file1 2>> file2|`
`   |   | |---| ||   ``   |   | |---| |command &>> output_file|  也可以配合管道使用：   ``   |   | |---| ||   ```   |   | |---| |command1 2>&1 \| command2|  意味着将 `command1` 的标准输出和标准错误输出都作为 `command2` 的标准输入。  同样的，大多数现代 shell 都提供了 `|&` 来作为 `2>&1 |` 的替代：   ```   |   | |---| ||   ``   |   | |---| |command1 \|& command2|  除了 0，1，2 外，也可以使用其他文件描述符，虽然不会有实际意义，但是可以作为临时值，例如：   ``   |   | |---| ||   ```   |   | |---| |{<br>    err=$(command 2>&1 1>&3)<br>} 3>&1|  这条命令首先将 `STDERR` 重定向到 `STDOUT`，然后将 `STDOUT` 重定向到新引入的文件描述符 `3`，完成变量 `err` 的赋值之后，再将 `3` 重定向到 `STDOUT`。最后的结果是：`STDERR` 的输出被保存到变量 `err` 中，而 `STDOUT` 的输出仍然打印在终端。  > GNU 建议不要使用大于 `9` 的文件描述符，因为这可能和 shell 分配的文件描述符冲突  #### 强制覆盖[](https://nihil.cc/posts/shell-redirect/#%E5%BC%BA%E5%88%B6%E8%A6%86%E7%9B%96)   ``

对于大部分人而言，这种形式的重定向十分的陌生：

`   |   | |---| ||   ```   |   | |---| |command >\| output_file|  在几乎所有个人电脑上，`>|` 和 `>` 没有任何区别。  大多数现代 shell 都提供一个 noclobber 的选项，它的作用是保护已经存在的文件不会被重定向破坏，但是默认都是关闭的，你可以使用 `set -o noclobber` 来启用该功能，或者 `set +o noclobber` 禁用该功能。启用 noclobber 后，如果重定向的文件已经存在，那么重定向就会失败，此时使用 `>|` 就可以显式覆盖已有文件。  ### 输入重定向[](https://nihil.cc/posts/shell-redirect/#%E8%BE%93%E5%85%A5%E9%87%8D%E5%AE%9A%E5%90%91)   ``

#### 基本形式[](https://nihil.cc/posts/shell-redirect/#%E5%9F%BA%E6%9C%AC%E5%BD%A2%E5%BC%8F-1)

`   |   | |---| ||   ```   |   | |---| |command < input_file|  将文件 `input_file` 的内容作为命令 `command` 的标准输入，而不是在终端中使用键盘输入。输入重定向也可以指定要重定向的文件描述符，或者重定向 `STDIN` 到某个文件描述符，通常用于临时值。  #### Here document[](https://nihil.cc/posts/shell-redirect/#here-document)   ``

Here document 是被视为一个单独的文件的字符流，其通常形式是：

`   |   | |---| ||   ````   |   | |---| |command << END<br>line1<br>line2<br>line3<br>END|  其中 `END` 是自定义的用以表示文档结束的标识符，不会作为字符流输出到 `STDIN` 中。行内行为与双引号的内容基本相同——变量名替换为其值，反引号（即 `` ` ``）内的命令被其执行结果替代等。这可以通过将结束符引用来阻止：   ````   |   | |---| ||   ```   |   | |---| |command << 'END'<br>line1<br>line2<br>line3<br>END|  如果使用 `<<-` 替代 `<<`，则 here document 会忽视所有制表符，这会方便在脚本中进行排版。  #### Here string[](https://nihil.cc/posts/shell-redirect/#here-string)   ``

与 here document 不同，here string 不使用结束符来标识文档的结束，而是使用单引号或双引号：

`   |   | |---| ||   ```   |   | |---| |command <<< "Here String"|  同时，当 here string 只有一个词时，不需要任何引号或双引号。  ### `exec` 与重定向[](https://nihil.cc/posts/shell-redirect/#exec-%E4%B8%8E%E9%87%8D%E5%AE%9A%E5%90%91)   ``

`exec` 是 shell 的一个内置函数，用于不产生子进程地执行一个新的命令，原进程将被替换。这也就是说，在 shell 中使用 `exec command`，当 `command` 退出后，shell 也会随之退出。但这并不是重点，今天要讨论的是 `exec` 的一个特殊用法：如果未指定 `command`，则任何重定向都会在当前 shell 中生效。例如，执行 `exec 2>&1` 会导致之后在 shell 内执行的所有命令都会把 `STDERR` 重定向到 `STDOUT`。

#### 倒带[](https://nihil.cc/posts/shell-redirect/#%E5%80%92%E5%B8%A6)

有没有想过这样一个问题，在已经消耗了 STDIN 的情况下，如何才能重新获得 STDIN 的数据再次处理？这里涉及到一个小技巧，被称为倒带（Rewind）：

`|   |   | |---|---| ||[nihil@zsh]$ {<br>>     cat;                # 第一次消耗 STDIN<br>>     exec < /dev/stdin   # 倒带<br>>     cat;                # 再次消耗 STDIN<br>> } <<< "Hello World"<br>Hello World<br>Hello World|`

上述命令将会输出两次 `Hello World`。需要注意的是，倒带仅限于输入文件，如果你尝试使用管道作为 `STDIN`，那么倒带会失败。

### 输入输出重定向[](https://nihil.cc/posts/shell-redirect/#%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%E9%87%8D%E5%AE%9A%E5%90%91)

其语法是：

`   |   | |---| ||   ```   |   | |---| |command n<> file|  其作用是将文件描述符的输入和输出都重定向到文件 `file`。不指定 n 时，默认为 0。通常，这是一个很少用到的语法，举个例子：   ```   |   | |---| ||   ```   |   | |---| |OUT=./out.txt;<br>exec 3<> $OUT;<br>echo "Hello World" >&3;<br>cat <&3;|  这将重定向文件描述符 `3` 到文件 `$OUT`，然后将输入和输出重定向到文件描述符 `3`。  ## 进程替换[](https://nihil.cc/posts/shell-redirect/#%E8%BF%9B%E7%A8%8B%E6%9B%BF%E6%8D%A2)   ``

进程替换的语法：

`   |   | |---| ||   ``   |   | |---| |<(command)<br>>(command)|  我们来看看它究竟是什么：   `

`|   |   | |---|---| ||[nihil@zsh]$ echo <(ls)<br>/proc/self/fd/11<br>[nihil@zsh]$ echo >(xxd)<br>/proc/self/fd/12|`

可以看到，其实质是将一个命令的标准输出当做一个临时文件来使用。对于管道和输入重定向，都是从 `STDIN` 输入数据的，但是许多命令并不支持从 `STDIN` 读取数据，必须要指定文件作为参数，这种情况就可以使用进程替换来实现，例如：

`|   |   | |---|---| ||[nihil@zsh]$ rustc <(echo "<br>> fn main() {<br>>     println!(\"hello world\");<br>> }<br>> ") -o ./helloworld<br>[nihil@zsh]$ ./helloworld<br>hello world|`

`rustc` 是 Rust 语言的编译器，将输入的文件编译为可执行文件，这里使用了进程替换，将临时文件传递给 `rustc` 编译，最终执行 `./helloworld` 将会在控制台打印 `hello world`。

同样的，如果有什么命令需要指定一个文件参数作为输出文件的话，也可以使用 `command1 >(command2)` 将 `command1` 的输出作为 `command2` 的标准输入。

除此之外，进程替换也可以用在重定向中，例如：

`|   |   | |---|---| ||[nihil@zsh]$ echo hello world > >(xxd)<br>00000000: 6865 6c6c 6f20 776f 726c 640a            hello world.<br>[nihil@zsh]$ xxd < <(echo hello world)<br>00000000: 6865 6c6c 6f20 776f 726c 640a            hello world.|`

这两种写法的效果是一样的，都是将 `echo` 的输出保存到临时文件，然后将 `xxd` 的 `STDIN` 重定向到该文件。这种写法的效果有点类似于使用管道：

`|   |   | |---|---| ||[nihil@zsh]$ echo hello world \| xxd<br>00000000: 6865 6c6c 6f20 776f 726c 640a            hello world.|`

但与管道不同的是，管道的两个命令在不同的子 shell 中执行，而进程替换的两个命令在同一个 shell 中。

zsh 还额外提供了 `=(command)` 语法，该语法几乎和 `<(command)` 完全一样，不同点是其在 `/tmp` 下创建文件：

`|   |   | |---|---| ||[nihil@zsh]$ echo =(ls)<br>/tmp/zsh7b9nb3|`

## 参考[](https://nihil.cc/posts/shell-redirect/#%E5%8F%82%E8%80%83)

[Redirections (Bash Reference Manual) - GNU.org](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)

[Process Substitution (Bash Reference Manual) - GNU.org](https://www.gnu.org/software/bash/manual/html_node/Process-Substitution.html)

[Opening the file descriptors for reading and writing](https://bash.cyberciti.biz/guide/Opening_the_file_descriptors_for_reading_and_writing)

[Shell Tips: tip1](http://linux-ip.net/misc/madlug/shell-tips/tip-1.txt)

[教程](https://nihil.cc/categories/%E6%95%99%E7%A8%8B/), [Shell](https://nihil.cc/categories/%E6%95%99%E7%A8%8B-shell/)

[linux](https://nihil.cc/tags/linux/) [shell](https://nihil.cc/tags/shell/) [教程](https://nihil.cc/tags/%E6%95%99%E7%A8%8B/)

本文由作者按照 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 进行授权