## 查看进程信息
`ps -ef | grep nginx` 是一个在 Unix/Linux 系统上常用的命令组合，用于查看与 `nginx` 相关的进程信息。让我们逐个部分解释这个命令：

1. `ps`：进程状态（process status）的缩写，是一个用来列出当前系统中的所有进程的命令。
2. `-e`：`ps` 命令的一个选项，表示显示所有进程。
3. `-f`：`ps` 命令的另一个选项，表示用全格式（full-format）显示进程信息。这将显示更多详细信息，包括进程的 UID（用户 ID）、PID（进程 ID）、PPID（父进程 ID）、C（CPU 使用率）、STIME（启动时间）、TTY（终端）、TIME（累计 CPU 时间）和 CMD（命令）。
4. `|`：管道符号，将前一个命令的输出作为下一个命令的输入。
5. `grep nginx`：`grep` 是一个文本搜索工具，用来在输入的文本中搜索匹配的模式。在这里，`grep nginx` 表示在 `ps -ef` 的输出中搜索包含 `nginx` 字符串的行。


### 示例
假设在系统中有正在运行的 `nginx` 进程，运行 `ps -ef | grep nginx` 可能会得到如下输出：
```
root     12345     1  0 12:34 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data 12346 12345  0 12:34 ?        00:00:00 nginx: worker process
user     12348  5678  0 12:35 pts/0    00:00:00 grep --color=auto nginx
```

解释：
- 第一行显示 `nginx` 的主进程（master process）。
- 第二行显示 `nginx` 的一个工作进程（worker process）。
- 第三行是 `grep nginx` 自身的进程，因为它也包含 `nginx` 字符串。

### 小提示
为了避免 `grep` 命令本身出现在结果中，可以使用如下命令：
```
ps -ef | grep nginx | grep -v grep
```
这里的 `grep -v grep` 表示排除包含 `grep` 字符串的行，从而去除 `grep` 进程自身的输出。


## 查看日志文件内容
`tail -f text.log` 是一个在 Unix/Linux 系统上用于查看日志文件内容的命令。让我们逐个部分解释这个命令：
1. `tail`：这是一个命令，用来显示文件的最后几行内容。默认情况下，它会显示文件的最后 10 行。
2. `-f`：`tail` 命令的一个选项，表示以“跟随”模式（follow mode）运行。使用 `-f` 选项后，`tail` 会持续监视文件的更新，并在文件末尾增加新内容时将其显示出来。这个选项特别适合用来实时查看日志文件的更新。
3. `text.log`：这是要查看的文件名。在这个示例中，`text.log` 是一个日志文件的名称。

`tail -f text.log` 命令的作用是显示 `text.log` 文件的最后几行内容，并且当文件有新的内容追加时，自动将新的内容显示出来。这在实时监控日志文件（如 Web 服务器日志、应用程序日志等）时非常有用。

### 示例
假设 `text.log` 文件的内容如下：
```
Line 1: Starting application
Line 2: Loading configuration
Line 3: Initializing modules
Line 4: Application started
Line 5: Listening on port 8080
```

运行 `tail -f text.log` 可能会显示：
```
Line 1: Starting application
Line 2: Loading configuration
Line 3: Initializing modules
Line 4: Application started
Line 5: Listening on port 8080
```

如果在 `text.log` 文件中追加新的内容，例如：
```
Line 6: Received request from client
```

则终端中的 `tail -f text.log` 会自动更新并显示：
```
Line 6: Received request from client
```

### 小提示
- 使用 `Ctrl+C` 可以中断并退出 `tail -f` 命令。
- `tail -f` 可以和其他命令结合使用，如 `grep`，以过滤显示特定的日志内容。例如，`tail -f text.log | grep ERROR` 会实时显示包含“ERROR”字符串的日志行。