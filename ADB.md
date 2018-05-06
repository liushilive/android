# ADB

Android 调试桥 (`adb`) 是多种用途的工具，该工具可以帮助你你管理设备或模拟器的状态。

可以通过下列几种方法进入 `adb`:

* 在设备上运行 shell 命令

* 通过端口转发来管理模拟器或设备

* 从模拟器或设备上拷贝来或拷贝走文件

下面对 `adb` 进行了介绍并描述了常见的使用。

## 概要

Android 调试系统是一个面对客户服务系统，包括三个组成部分：

1. 一个在你用于开发程序的电脑上运行的客户端。你可以通过 shell 端使用 adb 命令启动客户端。

1. 在你用于发的机器上作为后台进程运行的服务器。该服务器负责管理客户端与运行于模拟器或设备上的 `adb 守护程序` (daemon) 之间的通信。

1. 一个以后台进程的形式运行于模拟器或设备上的守护程序 (daemon)。

当你启动一个 adb 客户端，客户端首先确认是否已有一个 adb 服务进程在运行。如果没有，则启动服务进程。当服务器运行， adb 服务器就会绑定本地的 TCP 端口 5037 并监听 adb 客户端发来的命令，所有的 adb 客户端都是用端口 5037 与 adb 服务器对话的。

接着服务器将所有运行中的模拟器或设备实例建立连接。它通过扫描所有 5555 到 5585 范围内的奇数端口来定位所有的模拟器或设备。一旦服务器找到了 adb 守护程序，它将建立一个到该端口的连接。请注意任何模拟器或设备实例会取得两个连续的端口——一个偶数端口用来响应控制台的连接，和一个奇数端口用来响应 adb 连接。比如说：

    模拟器 1，控制台：端口 5554
    模拟器 1，Adb 端口 5555
    控制台：端口 5556
    Adb 端口 5557...

如上所示，模拟器实例通过 5555 端口连接 adb，就如同使用 5554 端口连接控制台一样。

一旦服务器与所有模拟器实例建立连接，就可以使用 adb 命令控制和访问该实例。因为服务器管理模拟器 / 设备实例的连接，和控制处理从来自多个 adb 客户端来的命令，你可以通过任何客户端（或脚本）来控制任何模拟器或设备实例。

以下的部分描述通过命令使用 adb 和管理模拟器 / 设备的状态。

## 发出 adb 命令

发出 Android 命令： 你可以在你的开发机上的命令行或脚本上发布 Android 命令，使用方法：

```bash
adb [-d|-e|-s <serialNumber>] <command>
```

当你发出一个命令，系统启用 Android 客户端。客户端并不与模拟器实例相关，所以如果双服务器 / 设备是运行中的，你需要用 -d 选项去为应被控制的命令确定目标实例。

## 查询模拟器 / 设备实例

在发布 adb 命令之前，有必要知道什么样的模拟器 / 设备实例与 adb 服务器是相连的。可以通过使用`devices`命令来得到一系列相关联的模拟器 / 设备：

```bash
adb devices
```

作为回应，adb 为每个实例都制定了相应的状态信息：
下面是一个展示 devices 命令和输出的例子 :

```bash
$ adb devices
List of devices attached
emulator-5554  device
emulator-5556  device
emulator-5558  device
```

* 序列号——由 adb 创建的一个字符串，这个字符串通过自己的控制端口<type>-<consolePort>唯一地识别一个模拟器 / 设备实例。

* 实例的连接状态有三种状态：

  * offline — 此实例没有与 adb 相连接或者无法响应

  * device — 此实例正与 adb 服务器连接。注意这个状态并不能百分之百地表示在运行和操作 Android 系统，因此这个实例是当系统正在运行的时候与 adb 连接的。然而，在系统启动之后，就是一个模拟器 / 设备状态的正常运行状态了

如果当前没有模拟器 / 设备运行，adb 则返回 `no device`。

## 给特定的模拟器 / 设备实例发送命令

如果有多个模拟器 / 设备实例在运行，在发布 adb 命令时需要指定一个目标实例。 这样做，请使用 -s 选项的命令。在使用的 -s 选项是：

```bash
adb -s <serialNumber> <command>
```

如上所示，给一个命令指定了目标实例，这个目标实例使用由 adb 分配的序列号。你可以使用 `devices` 命令来获得运行着的模拟器 / 设备实例的序列号。

示例如下：

```bash
adb -s emulator-5556 install helloWorld.apk
```

注意这点，如果没有指定一个目标模拟器 / 设备实例就执行 `-s` 这个命令的话，adb 会产生一个错误。

## 远程终端

```bash
adb tcpip 5555
adb connect <device-ip-address>
```

连接到指定的 ip, 这个通常配合 wifidebug，比如 adb connect 127.0.0.1:5037,5037 是默认端口号，海马模拟器是 adb connect 127.0.0.1:26944

断开

```bash
adb disconnect <device-ip-address>
adb disconnect 127.0.0.1:26944﻿
```

## 安装软件

你可以使用 adb 从你的开发电脑上复制一个应用程序，并且将其安装在一个模拟器 / 设备实例。像这样做，使用 `install` 命令。这个 `install` 命令要求你必须指定你所要安装的 `.apk` 文件的路径：

```bash
adb install <path_to_apk>
```

## 从模拟器 / 设备中拷入或拷出文件

可以使用`adbpull`,`push`命令将文件复制到一个模拟器 / 设备实例的数据文件或是从数据文件中复制。`install` 命令只将一个 `.apk` 文件复制到一个特定的位置，与其不同的是，`pull` 和 `push` 命令可令你复制任意的目录和文件到一个模拟器 / 设备实例的任何位置。

从模拟器或者设备中复制文件或目录，使用（如下命）:

```bash
adb pull <remote> <local>
```

将文件或目录复制到模拟器或者设备，使用（如下命令）：

```bash
adb push <local> <remote>
```

在这些命令中， `<local>` 和 `<remote>` 分别指通向自己的发展机（本地）和模拟器 / 设备实例（远程）上的目标文件 / 目录的路径。

下面是一个例子：:

```bash
adb push foo.txt /sdcard/foo.txt
```

## Adb 命令列表

下列表格列出了 adb 支持的所有命令，并对它们的意义和使用方法做了说明。

<TABLE>
    <TBODY>
      <TR>
        <TH>类别</TH>
        <TH>命令</TH>
        <TH>描述</TH>
        <TH>解释</TH>
      </TR>
      <TR>
        <TD rowSpan=3>Options<br/>选择项</TD>
        <TD><CODE>-d</CODE></TD>
        <TD>仅仅通过 USB 接口来管理 abd.</TD>
        <TD>如果不只是用 USB 接口来管理则返回错误。</TD>
      </TR>
      <TR>
        <TD><CODE>-e</CODE></TD>
        <TD>仅仅通过模拟器实例来管理 adb.</TD>
        <TD>如果不是仅仅通过模拟器实例管理则返回错误。</TD>
      </TR>
      <TR>
        <TD><CODE>-s&nbsp;&lt;serialNumber&gt;</CODE></TD>
        <TD>通过模拟器 / 设备的允许的命令号码来发送命令来管理 adb （比如："emulator-5556").</TD>
        <TD>如果没有指定号码，则会报错。</TD>
      </TR>
      <TR>
        <TD rowSpan=3>General<br/>常规的</TD>
        <TD><CODE>devices</CODE></TD>
        <TD>查看所有连接模拟器 / 设备的设施的清单。</TD>
        <TD></TD>
      </TR>
      <TR>
        <TD><CODE>help</CODE></TD>
        <TD>查看 adb 所支持的所有命令。.</TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD><CODE>version</CODE></TD>
        <TD>查看 adb 的版本序列号。</TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD rowSpan=3>Debug<br/>调试</TD>
        <TD><CODE>logcat&nbsp;[&lt;option&gt;] [&lt;filter-specs&gt;]</CODE></TD>
        <TD>将日志数据输出到屏幕上。</TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD><CODE>bugreport</CODE></TD>
        <TD>查看 bug 的报告，如<CODE>dumpsys</CODE>, <CODE>dumpstate</CODE>, 和<CODE>logcat</CODE>信息。 </TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD><CODE>jdwp</CODE></TD>
        <TD>查看指定的设施的可用的 JDWP（调试进程）信息。</TD>
        <TD>可以用 <CODE>forward jdwp:&lt;pid&gt;</CODE>端口映射信息来连接指定的 JDWP 进程。例如： <BR>
          <CODE>adb forward tcp:8000 jdwp:472</CODE><BR>
          <CODE>jdb -attach
          localhost:8000</CODE>
          <P></P></TD>
      </TR>
      <TR>
        <TD rowSpan=3>Data<br/>数据</TD>
        <TD><CODE>install&nbsp;&lt;path-to-apk&gt;</CODE></TD>
        <TD>安装 Android 为（可以模拟器 / 设施的数据文件。apk 指定完整的路径）. </TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD><CODE>pull&nbsp;&lt;remote&gt;&nbsp;&lt;local&gt;</CODE></TD>
        <TD>将指定的文件从模拟器 / 设施的拷贝到电脑上。</TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD><CODE>push&nbsp;&lt;local&gt;&nbsp;&lt;remote&gt;</CODE></TD>
        <TD>将指定的文件从电脑上拷贝到模拟器 / 设备中。</TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD rowSpan=2>Ports and Networking<br>调试端口和网络</TD>
        <TD><CODE>forward&nbsp;&lt;local&gt;&nbsp;&lt;remote&gt;</CODE></TD>
        <TD>用本地指定的端口通过 socket 方法远程连接模拟器 / 设施 </TD>
        <TD>端口需要描述下列信息：
          <UL>
            <LI><CODE>tcp:&lt;portnum&gt;</CODE>
            <LI><CODE>local:&lt;UNIX domain socket name&gt;</CODE>
            <LI><CODE>dev:&lt;character device name&gt;</CODE>
            <LI><CODE>jdwp:&lt;pid&gt;</CODE></LI>
          </UL></TD>
      </TR>
      <TR>
        <TD><CODE>ppp&nbsp;&lt;tty&gt;&nbsp;[parm]...</CODE></TD>
        <TD>通过 USB 运行 ppp：
          <UL>
            <LI><CODE>&lt;tty&gt;</CODE> — the tty for PPP stream. For example <CODE>dev:/dev/omap_csmi_ttyl</CODE>.
            <LI><CODE>[parm]... </CODE>&amp;mdash zero or more PPP/PPPD options,
              such as <CODE>defaultroute</CODE>, <CODE>local</CODE>, <CODE>notty</CODE>, etc.</LI>
          </UL>
          <P>需要提醒你的不能自动启动 PDP 连接。</P></TD>
        <TD></TD>
      </TR>
      <TR>
        <TD rowSpan=3>Scripting<br/>脚本</TD>
        <TD><CODE>get-serialno</CODE></TD>
        <TD>查看 adb 实例的序列号。</TD>
        <TD rowSpan=2></TD>
      </TR>
      <TR>
        <TD><CODE>get-state</CODE></TD>
        <TD>查看模拟器 / 设施的当前状态。</TD>
          </TD>
      </TR>
      <TR>
        <TD><CODE>wait-for-device</CODE></TD>
        <TD>如果设备不联机就不让执行，-- 也就是实例状态是 <CODE>device</CODE>时。</TD>
        <TD>你可以提前把命令转载在 adb 的命令器中，在命令器中的命令在模拟器 / 设备连接之前是不会执行其它命令的。示例如下：
          <PRE>adb wait-for-device shell getprop</PRE>
          需要提醒的是这些命令在所有的系统启动启动起来之前是不会启动 adb 的
          所以在所有的系统启动起来之前你也不能执行其它的命令。比如：运用<CODE>install</CODE> 的时候就需要 Android 包，这些包只有系统完全启动。例如：
          <PRE>adb wait-for-device install &lt;app&gt;.apk</PRE>
          上面的命令只有连接上了模拟器 / 设备连接上了 adb 服务才会被执行，而在 Android 系统完全启动前执行就会有错误发生。</TD>
      </TR>
      <TR>
        <TD rowSpan=2>Server<br/>服务</TD>
        <TD><CODE>start-server</CODE></TD>
        <TD>选择服务是否启动 adb 服务进程。</TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD><CODE>kill-server</CODE></TD>
        <TD>终止 adb 服务进程。</TD>
        <TD>&nbsp;</TD>
      </TR>
      <TR>
        <TD rowSpan=2>Shell<br>壳</TD>
        <TD><CODE>shell</CODE></TD>
        <TD>通过远程 shell 命令来控制模拟器 / 设备实例。</TD>
        <TD rowSpan=2></TD>
      </TR>
      <TR>
        <TD><CODE>shell&nbsp;[&lt;shellCommand&gt;]</CODE></TD>
        <TD>连接模拟器 / 设施执行 shell 命令，执行完毕后退出远程 shell 端 l.</TD>
      </TR>
    </TBODY>
</TABLE>

## 启动 shell 命令

Adb 提供了 shell 端，通过 shell 端你可以在模拟器或设备上运行各种命令。这些命令以 2 进制的形式保存在本地的模拟器或设备的文件系统中：

```bash
/system/bin/...
```

不管你是否完全进入到模拟器 / 设备的 adb 远程 shell 端，你都能 shell 命令来执行命令。

当没有完全进入到远程 shell 的时候，这样使用 shell 命令来执行一条命令：

```bash
adb [-d|-e|-s {<serialNumber>}] shell <shellCommand>
```

在模拟器 / 设备中不用远程 shell 端时，这样使用 shell 命 :

```bash
adb [-d|-e|-s {<serialNumber>}] shell
```

通过操作 `CTRL+D` 或 `exit` 就可以退出 shell 远程连接。

下面一些就将告诉你更多的关于 shell 命令的知识。

## 通过远程 shell 端运行 sqllite3 连接数据库

通过 adb 远程 shell 端，你可以通过 Android 软 sqlite3 命令程序来管理数据库。sqlite3 工具包含了许多使用命令，比如：`.dump` 显示表的内容，`.schema` 可以显示出已经存在的表空间的 SQL CREATE 结果集。Sqlite3 还允许你远程执行 sql 命令。

通过 sqlite3, 按照前几节的方法登陆模拟器的远程 shell 端，然后启动工具就可以使用 sqlite3 命令。当 sqlite3 启动以后，你还可以指定你想查看的数据库的完整路径。模拟器 / 设备实例会在文件夹中保存 SQLite3 数据库。/data/data/<package_name>/databases/.

示例如下：

```bash
$ adb -s emulator-5554 shell
# sqlite3 /data/data/com.android.email/databases/EmailProvider.db
SQLite version 3.3.12
Enter ".help" for instructions
.... enter commands, then quit...
sqlite> .exit
```

当你启动 sqlite3 的时候，你就可以通过 shell 端发送 `sqlite3` 命令了。用 `exit` 或 `CTRL+D` 退出 adb 远程 shell 端。

## 其它的 shell 命令

下面的表格列出了一些 adbshell 命令，如果需要全部的命令和程序，可以启动模拟器实例并且用 adb -help 命令 .

```bash
adb shell ls /system/bin
```

对大部分命令来说，help 都是可用的。

<TABLE>
<TBODY>
  <TR>
    <TH>Shell Command</TH>
    <TH>Description</TH>
    <TH>Comments</TH>
  </TR>
  <TR>
    <TD><CODE>dumpsys</CODE></TD>
    <TD>清除屏幕中的系统数据 n.</TD>
    <TD rowSpan=4> (DDMS) 工具提供了完整的调试</TD>
  </TR>
  <TR>
    <TD><CODE>dumpstate</CODE></TD>
    <TD>清除一个文件的状态。</TD>
  </TR>
  <TR>
    <TD><CODE>logcat&nbsp;[&lt;option&gt;]...&nbsp;[&lt;filter-spec&gt;]...</CODE></TD>
    <TD>启动信息日志并且但因输出到屏幕上。</TD>
  </TR>
  <TR>
    <TD><CODE>dmesg</CODE></TD>
    <TD>输出主要的调试信息到屏幕上。</TD>
  </TR>
  <TR>
    <TD><CODE>pm list packages</CODE></TD>
    <TD>获取已安装的应用的包名。</TD>
  </TR>
  <TR>
    <TD><CODE>pm path phone.android</CODE></TD>
    <TD>获取包名对应的 APK 路径。</TD>
  </TR>
  <TR>
    <TD><CODE>start</CODE></TD>
    <TD>启动或重启一个模拟器 / 设备实例。</TD>
    <TD>&nbsp;</TD>
  </TR>
  <TR>
    <TD><CODE>stop</CODE></TD>
    <TD>关闭一个模拟器 / 设备实例。</TD>
    <TD>&nbsp;</TD>
  </TR>
</TBODY>
</TABLE>

## 启用 logcat 日志

Android 日志系统提供了记录和查看系统调试信息的功能。日志都是从各种软件和一些系统的缓冲区中记录下来的，缓冲区可以通过 logcat 命令来查看和使用。

### 使用 logcat 命令

你可以用 logcat 命令来查看系统日志缓冲区的内容：

```bash
[adb] logcat [<option>] ... [<filter-spec>] ...
```

你也可以在你的电脑或运行在模拟器 / 设备上的远程 adb shell 端来使用 `logcat` 命令，也可以在你的电脑上查看日志输出。

```bash
$ adb logcat
```

你也这样使用：

```bash
# logcat
```

### 过滤日志输出

每一个输出的 Android 日志信息都有一个标签和它的优先级。

日志的标签是系统部件原始信息的一个简要的标志。（比如："View"就是查看系统的标签）.
优先级有下列几种，是按照从低到高顺利排列的：

* V — Verbose （最低优先级）
* D — Debug
* I — Info
* W — Warning
* E — Error
* F — Fatal
* S — Silent （最高优先级，没有任何东西被打印出来）

在运行 logcat 的时候在前两列的信息中你就可以看到 logcat 的标签列表和优先级别，它是这样标出的：`<priority>/<tag>`.

下面是一个 logcat 输出的例子，它的优先级就似乎 I, 标签就是 ActivityManage:

```bash
I/ActivityManager(  585): Starting activity: Intent { action=android.intent.action...}
```

为了让日志输出能体现管理的级别，你还可以用过滤器来控制日志输出，过滤器可以帮助你描述系统的标签等级。

过滤器语句按照下面的格式描 `tag:priority ...` , tag 表示是标签，priority 是表示标签的报告的最低等级。 从上面的 tag 的中可以得到日志的优先级。 你可以在过滤器中多次写 tag:priority 。

这些说明都只到空白结束。下面有一个列子，例子表示支持所有的日志信息，除了那些标签为"ActivityManager"和优先级为"Info"以上的和标签为" MyApp"和优先级为" Debug"以上的。 小等级，优先权报告为 tag.

```bash
adb logcat ActivityManager:I MyApp:D *:S
```

上面表达式的最后的元素 `*:S`，是设置所有的标签为"silent"，所有日志只显示有"View" 与 "MyApp"的，用 `*:S` 的另一个用处是 能够确保日志输出的时候是按照过滤器的说明限制的，也让过滤器也作为一项输出到日志中。

下面的过滤语句指显示优先级为 warning 或更高的日志信息：

```bash
adb logcat *:W
```

如果你电脑上运行 logcat ，相比在远程 adbshell 端，你还可以为环境变量 `ANDROID_LOG_TAGS:` 输入一个参数来设置默认的过滤

```bash
export ANDROID_LOG_TAGS="ActivityManager:I MyApp:D *:S"
```

需要注意的是 `ANDROID_LOG_TAGS` 过滤器如果通过远程 shell 运行 `logcat` 或用 `adb shell logcat` 来运行模拟器 / 设备不能输出日志。

### 控制日志输出格式

日志信息包括了许多元数据域包括标签和优先级。可以修改日志的输出格式，所以可以显示出特定的元数据域。可以通过 -v 选项得到格式化输出日志的相关信息。

* brief — 显示原始进程的 priority/tag 和 PID（默认格式）.
* process — 只显示 PID.
* tag — 只显示 priority/tag.
* thread — 显示进程：只显示 thread 、 priority/tag.
* raw — 显示原始的日志消息，没有其他的元数据字段。
* time — 显示原始进程的日期、调用时间、优先级 / 标记和 PID.
* long — 使用空白行显示所有元数据字段和单独的消息。

当启动了 logcat，你可以通过 -v 选项来指定输出格式：

```bash
[adb] logcat [-v <format>]
```

下面是用 thread 来产生的日志格式：

```bash
adb logcat -v thread
```

需要注意的是你只能 -v 选项来规定输出格式 option.

### 查看可用日志缓冲区

Android 日志系统有循环缓冲区，并不是所有的日志系统都有默认循环缓冲区。为了得到日志信息，你需要通过 -b 选项来启动 logcat。如果要使用循环缓冲区，你需要查看剩余的循环缓冲期：

* radio — 查看缓冲区的相关的信息。
* events — 查看和事件相关的的缓冲区。
* main — 查看主要的日志缓冲区

-b 选项使用方法：

```bash
[adb] logcat [-b <buffer>]
```

下面的例子表示怎么查看日志缓冲区包含 radio 和 telephony 信息：

```bash
adb logcat -b radio
```

### Logcat 命令列表

<TABLE>
    <TBODY>
      <TR>
        <TH>Option</TH>
        <TH>Description</TH>
      </TR>
      <TR>
        <TD><CODE>-b&nbsp;&lt;buffer&gt;</CODE></TD>
        <TD> 加载一个可使用的日志缓冲区供查看，比如<CODE>event</CODE>和<CODE>radio</CODE>.
          默认值是<CODE>main</CODE>。</TD>
      </TR>
      <TR>
        <TD><CODE>-c</CODE></TD>
        <TD>清除屏幕上的日志。</TD>
      </TR>
      <TR>
        <TD><CODE>-d</CODE></TD>
        <TD>输出日志到屏幕上。</TD>
      </TR>
      <TR>
        <TD><CODE>-f&nbsp;&lt;filename&gt;</CODE></TD>
        <TD>指定输出日志信息的<CODE>&lt;filename&gt;</CODE>，默认是<CODE>stdout</CODE>.</TD>
      </TR>
      <TR>
        <TD><CODE>-g</CODE></TD>
        <TD>输出指定的日志缓冲区，输出后退出。</TD>
      </TR>
      <TR>
        <TD><CODE>-n&nbsp;&lt;count&gt;</CODE></TD>
        <TD>设置日志的最大数目<CODE>&lt;count&gt;</CODE>.，默认值是 4，需要和 <CODE>-r</CODE>选项一起使用。</TD>
      </TR>
      <TR>
        <TD><CODE>-r&nbsp;&lt;kbytes&gt;</CODE></TD>
        <TD> 每<CODE>&lt;kbytes&gt;</CODE> 时输出日志，默认值为 16，需要和<CODE>-f</CODE> 选项一起使用。</TD>
      </TR>
      <TR>
        <TD><CODE>-s</CODE></TD>
        <TD>设置默认的过滤级别为 silent. </TD>
      </TR>
      <TR>
        <TD><CODE>-v&nbsp;&lt;format&gt;</CODE></TD>
        <TD>设置日志输入格式，默认的是<CODE>brief</CODE>格式。</TD>
      </TR>
    </TBODY>
</TABLE>

## 停止 adb 服务

在某些情况下，你可能需要终止 Android 调试系统的运行，然后再重新启动它。 例如，如果 Android 调试系统不响应命令，你可以先终止服务器然后再重启，这样就可能解决这个问题。

用 `kill-server` 可以终止 adb server。你可以用 adb 发出的任何命令来重新启动服务器。

## 重启设备

```bash
adb reboot
```
