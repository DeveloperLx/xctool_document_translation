# xctool

[原文](https://github.com/facebook/xctool/blob/master/README.md) 翻译：[DeveloperLx](http://weibo.com/DeveloperLx)

__xctool__ 是苹果 __xcodebuild__ 的扩展，它可以让测试iOS和Mac的产品更加轻松。它尤其对可持续集成非常有用。

[![Build Status](https://travis-ci.org/facebook/xctool.png?branch=master)](https://travis-ci.org/facebook/xctool)

[ [特性](#特性) &bull; [需求](#需求) &bull; [用法](#用法)
&bull; [可持续继承](#可持续继承)
&bull; [报告者](#报告者) &bull;
[配置xctool参数](#配置xctool参数) &bull; 
[贡献](#贡献) &bull; [需了解的问题和提示](#需了解的问题和提示) &bull; [许可证](#许可证) ]

## 特性

__xctool__ 是一个`xcodebuild test` 的插入式替换方案（drop-in replacement），可以为之添加一些额外的特性：

* **快速，并行地测试运行。**

_xctool_ 可以选择性地并行执行全部你的测试bundles，显著地提升你测试执行的速度。在Facebook，我们通过并行地执行提升了两到三倍的速度。

在 _run-tests_ 或 _test_ 使用`-parallelize`选项来打开。

[查看更多的信息](#parallelizing-test-runs)

* **结构化地输出测试。**

_xctool_ 用结构化JSON对象的形式捕获全部的测试结果。如果你正在构建一个可持续集成的系统，这意味着你不再必须去正则解析 _xcodebuild_ 的输出。

尝试报告者之一来定制输出，或使用`-reporter json-stream`选项来获取全部的事件流。

* **对人类友好的，ANSI配色的输出。**

_xcodebuild_ 难以置信得啰嗦，为每个源文件打印全部的编译命令和输出。默认的 _xctool_ 只有在出了什么错误的时候才会冗长地输出，这样可以让问题更容易被识别。

例如：

![pretty output](https://fpotter_public.s3.amazonaws.com/xctool-uicatalog.gif)

* **用 Objective-C 编写。**

_xctool_ 使用Objective-C编写。Mac OS X 和 iOS 可以轻松地在没有学习一门新语言的情况下，提交新的特性和修复Ta们遇到的问题。我们非常欢迎pull requests！

**注意：** 用xctool build项目已被废弃，并且不会去更新来支持将来版本的Xcode。我们建议将简单的需求移到 `xcodebuild` (用[xcpretty](https://github.com/supermarin/xcpretty)) , 牵扯较多的需求移动到 [xcbuild](https://github.com/facebook/xcbuild).
xctool 将继续支持测试（看上面的）.

## 需求

* Xcode 7 或更高的版本
* 你需要安装Xcode命令行工具。在Xcode中通过 _Xcode &rarr; Preferences &rarr; Downloads_ 来安装。

## 安装

xctool 可以用homebrew安装通过
```bash
brew install xctool
```
或通过xctool.sh命令下载和安装。

## 用法

xctool的命令和选项基本上是xcodebuild的超集。在大多数情况下，你只需要用 __xctool__ 替换 __xcodebuild__ ，事情就会像如同期望中一般地执行，并给出更有魅力的输出。

你可以得到帮助和全部选项的列表，通过：

```bash
path/to/xctool.sh -help
```

### 测试

_xctool_ 有一个 __run-tests__ 动作，可以理解怎么在你的scheme下执行测试。你可以有选择地限制运行测试，或改变使用的SDK。

在你的scheme下执行全部的测试，你可以用：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests
```

仅仅在一个指定的target来执行测试，使用`-only`选项：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -only SomeTestTarget
```

你可以更进一步地只执行一个指定的测试类：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -only SomeTestTarget:SomeTestClass
```

甚至再进一步，你可以只执行一个单独的测试类：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -only SomeTestTarget:SomeTestClass/testSomeMethod
```

你也可以为类或测试方法指定前缀：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -only SomeTestTarget:SomeTestClassPrefix*,SomeTestClass/testSomeMethodPrefix*
```

相对应的，你可以通过前缀匹配类或方法忽略特定的项目：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -omit SomeTestTarget:SomeTestClass/testSomeMethodPrefix*
```

您还可以针对不同的SDK运行测试：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -test-sdk iphonesimulator5.1
```

当运行测试时，你可有有选择地指定`-testTimeout`选项。当遇到一个测试超时时，就会得到失败的结果而不是无穷地等待。这可以让你的测试执行 避免由于不当的行为造成死锁。

默认的，应用测试将等待最多30秒让模拟器启动。如果你想改变这个超时，使用`-launch-timeout`选项。

#### Building 测试

在执行测试前你需要build它们。你可以使用__xcodebuild__,  __xcbuild__ 或 __Buck__来做。

例如:

```bash
xcodebuild \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
build-for-testing
```



##### Xcode 7

如果你正在使用Xcode 7来build，你可继续使用xctool来build测试，通过 __build-tests__ 或只是使用 __test__ 来执行测试。

例如:

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
build-tests
```

你可以选择性地用`-only`选项只build一个单独的测试target。

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
build-tests -only SomeTestTarget
```

#### 并行测试执行

_xctool_ 可以并行地执行单元测试，更好地利用闲置的CPU内核。在Facebook，我们通过并行地执行测试，看到了两三倍的增速。

要并行地运行测试bundles，使用`-parallelize`选项：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -parallelize
```

以上给了你并行的能力，但是你会受到最慢的测试bundle的约束。例如，如果你有两个测试bundle（'A' 和 'B'）, 但是'B'花费了10倍的时间，由于它包含10倍的测试，这样上面的并行机制就不会有太大帮助。

你可以通过使用`-logicTestBucketSize`选项，把你的测试执行分成若干小的部分，获得更进一步的速度提升：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -parallelize -logicTestBucketSize 20
```

上面的将把你的测试执行拆分成一个个的小部分（每个大小为20），它们将被并行地执行。如果一些你的测试bundle比其它的更大，这将提升全部的测试执行，帮助事情的解决。

### Building (仅限 Xcode 7)

**注意：** 使用xctool来build项目的支持已被废弃，在Xcode 8及其之后的版本中已不再使用。我们建议迁移到`xcodebuild`(和[xcpretty](https://github.com/supermarin/xcpretty))满足简单的需求，或[xcbuild](https://github.com/facebook/xcbuild)满足更多相关的需求。相对应地，你可以使用[Buck](https://buckbuild.com/)。

使用_xctool_来build项目与使用_xcodebuild_是等价的。

如果你使用workspaces和schemes：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
build
```

如果你使用projects和schemes：

```bash
path/to/xctool.sh \
-project YourProject.xcodeproj \
-scheme YourScheme \
build
```

全部的通常的选项`-configuration`, `-sdk`, `-arch`作用就像它们在_xcodebuild_中一样。

注意：_xctool_不支持直接使用`-target`build target；你必须使用schemes。

## 可持续集成

对于在一个可持续集成服务（如[Travis CI](https://travis-ci.org/)或[Jenkins](http://jenkins-ci.org/)）下执行你的测试，xctool是一个杰出的选项。
为了在一个可持续集成环境下执行你的测试，你必须为你的应用target创建**Shared Schemes**，并确保全部的依赖项（例如CocoaPods）被明确地添加到了Scheme。这么做：

1. 通过选择菜单**Product** > **Schemes** > **Manage Schemes...**，打开**Manage Schemes**表
1. 在列表中找到你的应用target。确保在右侧头部列的表单上的**Shared**的勾选框已选中。
1. 如果你的应用或测试target包含交叉项目的依赖（例如CocoaPods），你就需要确保他们被配置为明确的依赖。像这么做：
1. 点亮你的应用target，然后点击**Edit...**按钮来打开Scheme编辑表单。
1. 点击Scheme编辑器在左侧头部面板的**Build**标签。
1. 点击**+**按钮，添加每个依赖到项目中。CocoaPods将以一个名为**Pods**的静态库的形式出现。
1. 拖拽依赖到你的项目target之上，这样它就会第一个被built。

你将有一个新的文件在你的Xcode项目下，**xcshareddata/xcschemes**目录中。这是你刚刚配置的共享Scheme。检查这个文件是否在你的repository中，这样xctool就能找到和执行你的执行在下一次可持续集成的build时。

### 例如 Travis CI 配置

[Travis CI](https://travis-ci.org/) 是一个非常流行可持续集成系统，是一个免费提供的开源项目。它很好地与Github进行过整合，现在，它使用_xctool_作为OC项目默认的build和测试工具。一旦你为使用xctool设立了共享的Scheme，你需要配置一个`.travis.yml`文件。

如果你使用workspaces，你的`.travis.yml`可能是：

```yaml
language: objective-c
xcode_workspace: path/to/YourApp.xcworkspace
xcode_scheme: YourApp
```

如果你使用projects，你的`.travis.yml`可能是：

```yaml
language: objective-c
xcode_project: path/to/YourApp.xcodeproj
xcode_scheme: YourApp
```

更灵活的你也可以控制Travis怎么安装和调用xctools：

```yaml
language: objective-c
before_install:
- brew update
- brew install xctool
script: xctool -workspace MyApp.xcworkspace -scheme MyApp test
```

你可以通过参考[About OS X Travis CI
Environment](http://about.travis-ci.org/docs/user/osx-ci-environment/)文档，为iOS和OS X应用了解更多Travis CI的环境，并通过咨询[Getting
Started](http://about.travis-ci.org/docs/user/getting-started/)为配置你的项目找到深入的文档。

## 报告者

xctool有一个可以用不同格式输出build和测试结果的报告者。如果你自己没有指定任何报告者，xctool将默认使用`pretty`和`user-notifications`报告者。xctool也有这些特定的规则：

* 当xctool没有检测到TTY（显示终端名）时，重写是不可以在`pretty`报告者上使用的。这个可以通过在环境中设置`XCTOOL_FORCE_TTY`来覆盖。

* `user-notifications`将不会被使用

如果xctool检测到build正在被Travis CI, CircleCI, TeamCity, 或Jenkins执行（也就是说，`TRAVIS=true`, `CIRCLECI=true`, `TEAMCITY_VERSION`, 或在环境中`JENKINS_URL`）。
你可以通过`-reporter`选项使用你自己的报告者：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
-reporter plain \
build
```

默认的，报告者输出到标注输出中，但你也可以通过在报告者名称后添加`:OUTPUT_PATH`，来指导它输入到一个文件中：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
-reporter plain:/path/to/plain-output.txt \
build
```

你可以依照自己的喜好使用多个报告者；只需使用`-reporter`选项多次。

### 已包含的报告者

* __pretty__: a text-based reporter that uses ANSI colors and unicode
symbols for pretty output (the default).
* __plain__: like _pretty_, but with no colors or unicode.
* __phabricator__: outputs a JSON array of build/test results which can
be fed into the [Phabricator](http://phabricator.org/) code-review tool.
* __junit__: produces a JUnit/xUnit compatible XML file with test
results.
* __json-stream__: a stream of build/test events as JSON dictionaries,
one per line [(example
output)](https://gist.github.com/fpotter/82ffcc3d9a49d10ee41b).
* __json-compilation-database__: outputs a [JSON Compilation Database](http://clang.llvm.org/docs/JSONCompilationDatabase.html) of build events which can be used by [Clang Tooling](http://clang.llvm.org/docs/LibTooling.html) based tools, e.g. [OCLint](http://oclint.org).
* __user-notifications__: sends notification to Notification Center when action is completed [(example notifications)](https://cloud.githubusercontent.com/assets/1044236/2771974/a2715306-ca74-11e3-9889-fa50607cc412.png).
* __teamcity__: sends service messages to [TeamCity](http://www.jetbrains.com/teamcity/) Continuous Integration Server

### 实施你自己的报告者

You can also implement your own reporters using whatever language you
like.  Reporters in xctool are separate executables that read JSON
objects from STDIN and write formatted results to STDOUT.

You can invoke reporters by passing their full path via the `-reporter`
option:

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
-reporter /path/to/your/reporter \
test
```

For example, here's a simple reporter in Python that outputs a _period_
for every passing test and an _exclamation mark_ for every failing test:

```python
#!/usr/bin/python

import fileinput
import json
import sys

for line in fileinput.input():
obj = json.loads(line)

if obj['event'] == 'end-test':
if obj['succeeded']:
sys.stdout.write('.')
else:
sys.stdout.write('!')

sys.stdout.write('\n')
```

If you're writing a reporter in Objective-C, you'll find the
`Reporter` class helpful - see [Reporter.h](https://github.com/facebook/xctool/blob/master/Common/Reporter.h).


## 配置 (.xctool-args)

If you routinely need to pass many arguments to _xctool_ on the
command-line, you can use an __.xctool-args__ file to speed up your workflow.
If _xctool_ finds an __.xctool-args__ file in the current directory, it
will automatically pre-populate its arguments from there.

The format is just a JSON array of arguments:

```json
[
"-workspace", "YourWorkspace.xcworkspace",
"-scheme", "YourScheme",
"-configuration", "Debug",
"-sdk", "iphonesimulator",
"-arch", "i386"
]
```

Any extra arguments you pass on the command-line will take precedence 
over those in the _.xctool-args_ file.

## 贡献

Bug fixes, improvements, and especially new
[Reporter](#reporters)
implementations are welcome.  Before submitting a [pull
request](https://help.github.com/articles/using-pull-requests), please
be sure to sign the [Facebook
Contributor License
Agreement](https://developers.facebook.com/opensource/cla).  We can't
accept pull requests unless it's been signed.

#### 工作流 

1. Fork.
2. Make a feature branch: __git checkout -b my-feature__
3. Make your feature.  Keep things tidy so you have one commit per self-contained change (squashing can help).
3. Push your branch to your fork: __git push -u origin my-feature__
4. Open GitHub, under "Your recently pushed branches", click __Pull
Request__ for _my-feature_.

Be sure to use a separate feature branch and pull request for every
self-contained feature.  If you need to make changes from feedback, make
the changes in place rather than layering on commits (use interactive
rebase to edit your earlier commits).  Then use __git push --force
origin my-feature__ to update your pull request.

#### 工作流 (为Facebook员工, 其它提交者)

Mostly the same, but use branches in the main xctool repo if you prefer.
It's a nice way to keep things together.

1. Make a feature branch: __git checkout -b myusername/my-feature__
2. Push your branch: __git push -u origin myusername/my-feature__
3. Open GitHub to [facebook/xctool](https://github.com/facebook/xctool),
under "Your recently pushed branches", click __Pull Request__ for
_myusername/my-feature_.

## 需了解的问题和提示

* __使用共享的scheme，并关掉自动创建Scheme特性。__

Xcode有两种scheme：共享的和某用户的。默认是某用户的scheme，它们保存在一个名为`USERNAME.xcuserdatad`的目录下，大多数人正确地将其添加到了_.gitignore_中。

使用共享的scheme替代，并将其提交到你的repo上。这样你团队中的每个人（和你的build服务器）就可以使用相同的信息工作，并以相同的方式build。

![example](https://fpotter_public.s3.amazonaws.com/xctool-shared-schemes.png)

* __确保模拟器运行在图形界面工具下__。

如果你运行`xctool`在可持续集成，你的用户账号调用`xctool`**必须**有一个活动的图形界面工具的上下文。

如果不是这样，模拟器不能成功地启动，并给出神秘的警告像这样：

```
Tried to install the test host app 'com.myapp.test' but failed.
Preparing test environment failed.
-[TEST_BUNDLE FAILED_TO_START] 
There was a problem starting the test bundle: Simulator 'iPhone 6' was not prepared: Failed for unknown reason.
Test did not run: Simulator 'iPhone 6' was not prepared: Failed for unknown reason.
2015-01-21 12:02:19.296 xcodebuild[35135:875297]  iPhoneSimulator: Timed out waiting 120 seconds for simulator to boot, current state is 1.
Testing failed:
Test target MyProjectTests encountered an error (Timed out waiting 120 seconds for simulator to boot, current state is 1.  
```

注意这对于`xcodebuild`也是相同的...这不是`xctool`所特有的。

关于更多的信息，查看[Jason Jarrett的帖子](http://staxmanade.com/2015/01/setting-jenkins-up-to-run-xctool-and-xcode-simulator-tests/)。

## 许可证

版权 2014-present Facebook

Apache License 2.0版本授权；你必须在这个许可证下使用这个产品。你可以获得一个LICENSE文件的许可证副本，或从这里：

http://www.apache.org/licenses/LICENSE-2.0

本软件是在"AS IS" BASIS许可证下发布的，不接受任何明示或暗示的担保条件，除非有可使用的法律或书面约定。您可以查看许可证中的权限和特定语言下的权限。
