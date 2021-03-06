# xctool

#### [原文](https://github.com/facebook/xctool/blob/master/README.md) 翻译：[DeveloperLx](http://weibo.com/DeveloperLx)

__xctool__ 是苹果 __xcodebuild__ 的扩展，它可以让测试iOS和Mac的产品更加轻松，尤其对可持续集成非常有帮助。

[![Build Status](https://travis-ci.org/facebook/xctool.png?branch=master)](https://travis-ci.org/facebook/xctool)

[ [特性](#特性) &bull; [需求](#需求) &bull; [用法](#用法) &bull; [可持续集成](#可持续集成) &bull; [报告者](#报告者) &bull; [配置](#配置（.xctool-args）) &bull; [贡献](#贡献) &bull; [了解问题和提示](#了解问题和提示) &bull; [许可证](#许可证) ]

## 特性

__xctool__ 是一个`xcodebuild测试` 的插入式替换方案（drop-in replacement），为之添加了一些额外的特性：

* **更快，并行地测试运行。**

_xctool_ 可以选择性地并行执行你的全部测试bundles，显著地提升你测试执行的速度。在Facebook，我们通过并行执行，提升了两到三倍的速度。

在 _run-tests_ 或 _test_ 使用`-parallelize`选项打开。[查看更多的信息](#parallelizing-test-runs)

* **结构化地输出测试结果。**

_xctool_ 以结构化JSON对象的形式捕获全部的测试结果。如果你正在构建一个可持续集成的系统，这意味着你不再必须去正则解析 _xcodebuild_ 的输出。

尝试用一种[报告者](#报告者)来定制输出，或使用`-reporter json-stream`选项来获取全部的事件流。

* **对人类友好的，ANSI配色的输出。**

_xcodebuild_ 难以置信得啰嗦，它为每个源文件打印全部的编译命令和输出。而默认的，_xctool_ 只有在出了什么错误的时候才会冗长地输出，这会让问题更加容易被识别在哪里。

例如：

![pretty output](https://fpotter_public.s3.amazonaws.com/xctool-uicatalog.gif)

* **用 Objective-C 编写。**

_xctool_ 使用Objective-C编写。Mac OS X 和 iOS 可以在没有学习一门新语言的情况下，轻松地提交新的特性，修复Ta们遇到的问题。我们非常欢迎pull requests！

**注意：** 用xctool build项目已被废弃，并且不会去更新以支持将来版本的Xcode。我们建议将简单的需求移到`xcodebuild`(用[xcpretty](https://github.com/supermarin/xcpretty)) , 牵扯较多的需求移动到 [xcbuild](https://github.com/facebook/xcbuild).xctool将继续支持测试（看上面的）。

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

xctool的命令和选项基本上就是xcodebuild的超集。在大多数情况下，你只需要用__xctool__替换__xcodebuild__，一切就会像如同期望中一般地执行，并给出更有吸引力的输出。

你可以得到帮助和全部选项的列表，通过：

```bash
path/to/xctool.sh -help
```

### 测试

_xctool_有一个__run-tests__的动作，可以理解怎么在你的scheme下执行测试。你可以选择性地限制运行哪些测试，或改变使用的SDK。

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

相对应的，你可以通过前缀匹配类或测试方法，来忽略特定的项目：

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

当运行测试时，你可以有选择地指定`-testTimeout`选项。当遇到一个测试超时时，就不会无穷地等待，而是得到失败的结果。这可以让你测试的执行避免由于不正当的行为造成死锁。

默认的，应用测试将等待最多30秒让模拟器启动。如果你想改变这个超时，使用`-launch-timeout`选项。

#### Building 测试

在执行测试前，你需要build它们。你可以使用__xcodebuild__,  __xcbuild__或__Buck__去做。

例如:

```bash
xcodebuild \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
build-for-testing
```

##### Xcode 7

如果你正在使用Xcode 7来build，你可继续使用xctool来build测试，通过__build-tests__或只是使用__test__来执行测试。

例如：

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

_xctool_可以并行地执行单元测试，更好地利用闲置的CPU内核。在Facebook，我们通过并行地执行测试，看到了两三倍的增速。

要并行地运行测试bundles，使用`-parallelize`选项：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -parallelize
```

以上给了你并行的能力，但是你会受到最慢的测试bundle的约束。例如，如果你有两个测试bundle（'A' 和 'B'）, 但是'B'花费了10倍的时间，由于它包含10倍的测试，这样上面的并行机制就不会有太大帮助。

你可以通过使用`-logicTestBucketSize`选项，把你的测试执行分成若干小的部分，获得更进一步速度的提升：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
run-tests -parallelize -logicTestBucketSize 20
```

上面的将把你的测试执行拆分成一个个的小部分（每个大小为20），它们将被并行地执行。如果一些你的测试bundle比其它的更大，这将提升全部的测试执行，帮助事情的解决。

### Building (仅限 Xcode 7)

**注意：** 使用xctool来build项目的支持已被废弃，在Xcode 8及其之后的版本中已不再使用。我们建议迁移到`xcodebuild`(和[xcpretty](https://github.com/supermarin/xcpretty))来满足简单的需求，或[xcbuild](https://github.com/facebook/xcbuild)满足更多相关的需求。相对应地，你可以使用[Buck](https://buckbuild.com/)。

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

对于在一个可持续集成服务（如[Travis CI](https://travis-ci.org/)或[Jenkins](http://jenkins-ci.org/)）下执行你的测试，xctool是一个优秀的的选项。
为了在一个可持续集成环境下执行你的测试，你必须为你的应用target创建**Shared Schemes**，并确保全部的依赖项（例如CocoaPods）明确地被添加到了Scheme。这么做：

1. 通过选择菜单**Product** > **Schemes** > **Manage Schemes...**，打开**Manage Schemes**表
1. 在列表中找到你的应用target。确保在右侧头部列的表单上的**Shared**的勾选框已选中。
1. 如果你的应用或测试target包含交叉项目的依赖（例如CocoaPods），你就需要确保他们被配置为明确的依赖。像这么做：
1. 点亮你的应用target，然后点击**Edit...**按钮来打开Scheme编辑表单。
1. 点击Scheme编辑器在左侧头部面板的**Build**标签。
1. 点击**+**按钮，添加每个依赖到项目中。CocoaPods将以一个名为**Pods**的静态库的形式出现。
1. 拖拽依赖到你的项目target之上，这样它就会第一个被built。

你将得到一个新的文件在你的Xcode项目下的**xcshareddata/xcschemes**目录中，它是你刚刚配置的共享Scheme。在你的repository中检查这个文件，然后xctool就可以在你下次的CI build时找到和执行你的测试。

### 例如 Travis CI 配置

[Travis CI](https://travis-ci.org/) 是一个非常流行的可持续集成系统，也是一个免费提供的开源项目。它很好地与Github进行了整合，现在，它使用_xctool_作为OC项目默认的build和测试工具。如果你为使用xctool设立了共享的Scheme，你需要配置一个`.travis.yml`文件。

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

你可以通过参考[About OS X Travis CI Environment](http://about.travis-ci.org/docs/user/osx-ci-environment/)文档，为iOS和OS X应用了解更多Travis CI的环境，并通过咨询[Getting Started](http://about.travis-ci.org/docs/user/getting-started/)为配置你的项目找到深入的文档。

## 报告者

xctool拥有可以通过不同格式输出build和测试结果的报告者。如果你自己没有指定任何报告者，xctool将默认使用`pretty`和`user-notifications`报告者。xctool也有这些特定的规则：

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

* __pretty__：默认项，一个基于文本的报告者，使用ANSI配色和统一编码的符号得到较好的输出。
* __plain__：类似于_pretty_，但没有颜色和统一编码。
* __phabricator__：输出一个build或测试的JSON数组，它可以注入到[Phabricator](http://phabricator.org/)这个code-review工具中。
* __junit__：用测试的结果生成一个JUnit/xUnit兼容的XML文件。
* __json-stream__：一个build或测试事件的JSON字典的流，每行一个[(示例输出)](https://gist.github.com/fpotter/82ffcc3d9a49d10ee41b)。
* __json-compilation-database__：输出一个build事件的[“JSON编辑数据库”（JSON Compilation Database）](http://clang.llvm.org/docs/JSONCompilationDatabase.html)，它可以被基于[Clang Tooling](http://clang.llvm.org/docs/LibTooling.html)的工具使用，例如：[OCLint](http://oclint.org)。
* __user-notifications__：当动作完成时，发送通知到通知中心[(示例通知)](https://cloud.githubusercontent.com/assets/1044236/2771974/a2715306-ca74-11e3-9889-fa50607cc412.png)。
* __teamcity__：发送服务信息到[TeamCity](http://www.jetbrains.com/teamcity/)可持续集成服务

### 实现你自己的报告者

你也可以用任何你喜欢的语言实现你自己的报告者。在xctool中，报告者从STDIN读取JSON对象，以及写入格式化的结果到STDOUT是分别执行的。

你可以通过传递它们的全路径给`-reporter`选项来调用报告者：

```bash
path/to/xctool.sh \
-workspace YourWorkspace.xcworkspace \
-scheme YourScheme \
-reporter /path/to/your/reporter \
test
```

例如，这里有一个简单的用python写的报告者，它为每个通过测试输出_._，每个失败的测试输出_!_：

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

如果你要用OC写一个报告者，你可以找到`Reporter`类的帮助 - 查看[Reporter.h](https://github.com/facebook/xctool/blob/master/Common/Reporter.h)。

## 配置 (.xctool-args)

在命令行中，如果你常规地需要传递很多参数给_xctool_，你可以使用一个__.xctool-args__文件加速你的工作流。如果_xctool_在当前目录下发现了__.xctool-args__文件，它就会自动地从这里得到参数。

它的格式就是一个参数的JSON数组：

```json
[
"-workspace", "YourWorkspace.xcworkspace",
"-scheme", "YourScheme",
"-configuration", "Debug",
"-sdk", "iphonesimulator",
"-arch", "i386"
]
```

你传递到命令行的任何额外的参数将会覆盖掉相应在_.xctool-args_文件中的。

## 贡献

欢迎修复bug，改进，以及实现新的报告者。在提交[pull
request](https://help.github.com/articles/using-pull-requests)前，请确认签署了[Facebook贡献者执照协议](https://developers.facebook.com/opensource/cla)。我们只接受签署过的pull request。

#### 工作流 

1. Fork。
2. 创建一个特性分支：__git checkout -b my-feature__
3. 使你的特性保持整洁，每个单独的改变有一个commit（细化是比较有帮助的）。
4. Push你的分支到你的fork：__git push -u origin my-feature__
5. 打开GitHub，在"Your recently pushed branches"下，点击_my-feature_的__Pull Request__。

确认使用单独的分支，为每个单的的特性提交pull request。如果你需要从反馈中做出改变，做出适当的改变而不是依靠大量的提交（使用交互式的rebase来编辑你较早的commit）。然后使用__git push --force origin my-feature__来更新你的pull request。

#### 工作流 (为Facebook员工, 其它提交者)

大多情况下都是一样的，但如果你偏好使用在主xctool repo的分支，也可以。保持事情一致是一个好的方向。

1. 创建一个特性分支：__git checkout -b myusername/my-feature__
2. Push你的分支：__git push -u origin myusername/my-feature__ 
3. 打开GitHub[facebook/xctool](https://github.com/facebook/xctool)，在"Your recently pushed branches"下，点击_myusername/my-feature_的__Pull Request__。

## 了解问题和提示

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

版权 2014年颁发 Facebook

Apache License 2.0版本授权；你必须在这个许可证下使用这个产品。你可以获得一个LICENSE文件的许可证副本，或从这里：

http://www.apache.org/licenses/LICENSE-2.0

本软件是在"AS IS" BASIS许可证下发布的，不接受任何明示或暗示的担保条件，除非有可使用的法律或书面约定。您可以查看许可证中的权限和特定语言下的权限。
