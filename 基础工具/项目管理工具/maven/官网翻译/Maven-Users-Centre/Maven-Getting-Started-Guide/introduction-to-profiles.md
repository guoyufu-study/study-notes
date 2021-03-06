## 介绍构建 profiles

> https://maven.apache.org/guides/introduction/introduction-to-profiles.html

Apache Maven 竭尽全力确保构建是可移植的。除其他外，这意味着允许在 POM 中进行构建配置，避免**所有**文件系统引用（在继承、依赖项和其他地方），并更加依赖本地存储库来存储实现这一点所需的元数据。

然而，有时可移植性并非完全可行。在某些情况下，插件可能需要配置本地文件系统路径。在其他情况下，将需要稍微不同的依赖集，并且项目的工件名称可能需要稍微调整。在其他时候，您甚至可能需要在构建生命周期中包含整个插件，具体取决于检测到的构建环境。

为了解决这些情况，Maven 支持构建 profiles。Profiles 是使用 POM 本身中可用元素的子集（加上一个额外的部分）指定的，并以多种方式中的任何一种触发。它们在构建时修改 POM，旨在用于互补集，为一组目标环境提供等效但不同的参数（例如，提供开发、测试和生产环境中的 appserver 根的路径）。因此，profiles 很容易导致团队中不同成员的不同构建结果。但是，如果使用得当，可以使用 profiles ，同时仍然保持项目的可移植性。这也将最大限度地减少使用 maven 的 `-f` 选项，它允许用户创建另一个具有不同参数或配置的 POM 来构建，这使得它更易于维护，因为它仅使用一个 POM 运行。

### 有哪些不同类型的 profile？每个定义在哪里？

- 每个项目
  
  - 在 POM 本身中定义`(pom.xml)`。

- 每个用户
  
  - 在[Maven-settings](https://maven.apache.org/ref/current/maven-settings/settings.html) `(%USER_HOME%/.m2/settings.xml)`中定义。

- 全球的
  
  - 在[全局 Maven-settings](https://maven.apache.org/ref/current/maven-settings/settings.html) `(${maven.home}/conf/settings.xml)`中定义。

- 配置文件描述符
  
  - 位于[项目 basedir`(profiles.xml)`](https://maven.apache.org/ref/2.2.1/maven-profile/profiles.html)中的描述符（Maven 3.0 及更高版本不再支持；请参阅[Maven 3 兼容性说明](https://cwiki.apache.org/confluence/display/MAVEN/Maven+3.x+Compatibility+Notes#Maven3.xCompatibilityNotes-profiles.xml)）

### 如何触发 profile？这如何根据所使用的 profile 类型而变化？

可以通过多种方式激活 profile：

- 从命令行
- 通过 Maven 设置
- 基于环境变量
- 操作系统设置
- 存在或丢失文件

#### 有关 profile 激活的详细信息

可以使用 `-P` 命令行标志显式指定 profiles。

此标志后跟要使用的 profile IDs 的逗号分隔列表。选项中指定的 profile(s) 与通过其激活配置或 `settings.xml` 中的 `<activeProfiles>` 部分激活的 profiles 一起被激活。从 Maven 4 开始，Maven 将拒绝激活或停用无法解析的 profile。为防止这种情况，请在 profile 标识符前加上`?`，将其标记为可选：

``` 
mvn groupId:artifactId:goal -P profile-1,profile-2,?profile-3
```

可以在 Maven 设置中通过 `<activeProfiles>` 部分激活 Profiles 。本节采用一个 `<activeProfile>` 元素列表，每个元素都包含一个 profile-id。

``` xml
<settings>
    ...
    <activeProfiles>
        <activeProfile>profile-1</activeProfile>
    </activeProfiles>
    ...
</settings>
```

每次项目使用它时，默认情况下都会激活 `<activeProfiles>` 标签中列出的 profiles。

可以根据检测到的构建环境状态自动触发 Profiles 。这些触发器是通过 profile 本身的一个 `<activation>` 部分指定的。目前，此检测仅限于 JDK 版本的前缀匹配、系统属性的存在或系统属性的值。这里有些例子。

以下配置将在 JDK 的版本以 “1.4” 开头时触发 profile（例如“1.4.0_08”、“1.4.2_07”、“1.4”）：

``` xml
<profiles>
    <profile>
        <activation>
            <jdk>1.4</jdk>
        </activation>
        ...
    </profile>
</profiles>
```

从 Maven 2.1 开始也可以使用范围（有关更多信息，请参阅 [Enforcer 版本范围语法](https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html)）。以下为版本 1.3、1.4 和 1.5。

``` xml
<profiles>
    <profile>
        <activation>
            <jdk>[1.3,1.6)</jdk>
        </activation>
        ...
    </profile>
</profiles>
```

*注意：*上限 `,1.5]` 可能不包括 1.5 的大多数版本，因为它们将有一个额外的“补丁”版本，例如 `_05` 上述范围内未考虑的版本。

下一个将根据操作系统设置激活。有关操作系统值的更多详细信息，请参阅 [Maven Enforcer 插件](https://maven.apache.org/enforcer/enforcer-rules/requireOS.html)。

``` xml
<profiles>
    <profile>
        <activation>
            <os>
                <name>Windows XP</name>
                <family>Windows</family>
                <arch>x86</arch>
                <version>5.1.2600</version>
            </os>
        </activation>
        ...
    </profile>
</profiles>
```

当系统属性 “debug” 被指定为任何值时，下面的 profile 将被激活：

``` xml
<profiles>
    <profile>
        <activation>
            <property>
                <name>debug</name>
            </property>
        </activation>
        ...
    </profile>
</profiles>
```

当系统属性“debug”根本没有定义时，将激活以下 profile ：

``` xml
<profiles>
    <profile>
        <activation>
            <property>
                <name>!debug</name>
            </property>
        </activation>
        ...
    </profile>
</profiles>
```

当未定义系统属性“debug”或使用非“true”值定义时，将激活以下 profile 。

``` xml
<profiles>
    <profile>
        <activation>
            <property>
                <name>debug</name>
                <value>!true</value>
            </property>
        </activation>
        ...
    </profile>
</profiles>
```

要激活它，您可以在命令行上键入其中一个：

``` powershell
mvn groupId:artifactId:goal
mvn groupId:artifactId:goal -Ddebug=false
```

下一个示例将在使用值“test”指定系统属性“environment”时触发 profile ：

``` xml
<profiles>
    <profile>
        <activation>
            <property>
                <name>environment</name>
                <value>test</value>
            </property>
        </activation>
        ...
    </profile>
</profiles>
```

要激活它，您可以在命令行上键入：

``` xml
mvn groupId:artifactId:goal -Denvironment=test
```

从 Maven 3.0 开始，POM 中的 profile 也可以根据 `settings.xml` 中的活动 profiles 的属性来激活。

**注意**： 环境变量，如 `FOO` ，可以以 `env.FOO` 的形式使用。进一步注意，环境变量名称在 Windows 上被规范化为全部大写。

这个示例将在生成的文件`target/generated-sources/axistools/wsdl2java/org/apache/maven` 丢失时触发 profile 。

``` xml
<profiles>
    <profile>
        <activation>
            <file>
                <missing>target/generated-sources/axistools/wsdl2java/org/apache/maven</missing>
            </file>
        </activation>
        ...
    </profile>
</profiles>
```

从 Maven 2.0.9 开始，可以对标签 `<exists>` 和 `<missing>` 进行插值。支持的变量是系统属性，如 `${user.home}` ，和环境变量，如`${env.HOME}`。请注意，POM 本身中定义的属性和值在这里不能用于插值，例如上面的示例激活器不能使用`${project.build.directory}`但需要硬编码路径 `target`。

Profiles 也可以在默认情况下激活，使用如下配置：

``` xml
<profiles>
    <profile>
        <id>profile-1</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        ...
    </profile>
</profiles>
```

除非使用上述方法之一激活同一 POM 中的另一个 profile ，否则这个 profile 将自动对所有构建处于活动状态。当在命令行或通过其激活配置激活 POM 中的 profile  时，默认情况下处于活动状态的所有 profiles 都会自动停用。

#### 停用 profile 

从 Maven 2.0.10 开始，可以使用命令行禁用一个或多个 profiles ，方法是在其标识符前加上字符 “!” 或 '-' 如下图所示：

```
mvn groupId:artifactId:goal -P !profile-1,!profile-2,!?profile-3
```

或者

```
mvn groupId:artifactId:goal -P -profile-1,-profile-2,-?profile-3
```

这可用于停用标记为 `activeByDefault` 的 profiles 或将通过其激活配置激活的 profiles 。

### 每种类型的 profile 可以自定义 POM 的哪些区域？为什么？

既然我们已经讨论了在何处指定 profiles 以及如何激活它们，那么讨论您可以在 profile 中指定*什么内容*将会很有用。与 profile 配置的其他方面一样，这个答案并不简单。

根据您选择配置 profile 的位置，您将可以访问不同的 POM 配置选项。

#### 外部文件中的 profiles 

在外部文件（即 在 `settings.xml` 或 `profiles.xml`）中指定的 profiles 在最严格的意义上是不可移植的。任何似乎有很大可能改变构建结果的东西都被限制在 POM 中的内联 profiles 中。诸如存储库列表之类的东西可能只是已批准工件的私有存储库，并且不会改变构建的结果。因此，您将只能修改 `<repositories>` 和 `<pluginRepositories>` 部分，以及一个额外的 `<properties>` 部分。

`<properties>` 部分允许您指定自由格式的键值对，这些键值对将包含在 POM 的插值过程中。这允许您以 `${profile.provided.path}` 的形式指定插件配置。

#### POM 中的 profiles

另一方面，如果您的 profiles 可以在 POM 中合理地指定*，*那么您有更多的选择。当然，这样做的代价是您只能修改该项目及其子模块。由于这些 profiles 是内联指定的，因此更有可能保持可移植性，因此可以合理地说您可以向它们添加更多信息，而不会有其他用户无法使用该信息的风险。

POM 中指定的 profiles 可以修改[以下 POM 元素](https://maven.apache.org/ref/current/maven-model/maven.html)：

- `<repositories>`
- `<pluginRepositories>`
- `<dependencies>`
- `<plugins>`
- `<properties>`（实际上在主 POM 中不可用，但在幕后使用）
- `<modules>`
- `<reports>`
- `<reporting>`
- `<dependencyManagement>`
- `<distributionManagement>`
- `<build>` 元素的子集，包括：
  - `<defaultGoal>`
  - `<resources>`
  - `<testResources>`
  - `<directory>`
  - `<finalName>`
  - `<filters>`
  - `<pluginManagement>`
  - `<plugins>`

#### `<profiles>` 之外的 POM 元素

我们不允许在 POM profiles 之外修改某些 POM 元素，因为这些运行时修改不会在 POM 部署到存储库系统时分发，从而使该人对该项目的构建与其他人完全不同。虽然您可以在一定程度上使用外部 profiles 的选项来实现这一点，但是危险是有限的。另一个原因是这个 POM 信息有时会从父 POM 重用。

外部文件，例如 `settings.xml` 和 `profiles.xml`，也不支持 POM profiles 之外的元素。让我们以这个场景进行详细说明。当有效的 POM 部署到远程存储库时，任何人都可以从存储库中获取其信息并使用它直接构建 Maven 项目。现在，想象一下，如果我们可以在依赖项中设置 profiles，依赖项对构建非常重要，或者可以在 `settings.xml` 中 POM profiles 之外的任何其他元素中设置 profiles ，那么很可能我们不能期望其他人使用存储库中的 POM 并能够构建它。我们还必须考虑如何和其他人分享`settings.xml`。请注意，要配置的文件过多会非常混乱且难以维护。底线是，由于这是构建数据，它应该在 POM 中。Maven 2 的目标之一是将运行构建所需的所有信息整合到单个文件或文件层次结构中，即 POM。

### Profile 顺序

来自活动 profiles 的 POM 中的所有 profile 元素都会覆盖与 POM 具有相同名称的全局元素，或者在集合的情况下扩展这些元素。如果多个 profiles 在同一个 POM 或外部文件中处于活动状态，则**稍后**定义的那些优先于**之前**定义的那些（与其 profile ID 和激活顺序无关）。

示例：

``` xml
<project>
    ...
    <repositories>
        <repository>
            <id>global-repo</id>
            ...
        </repository>
    </repositories>
    ...
    <profiles>
        <profile>
            <id>profile-1</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <repositories>
                <repository>
                    <id>profile-1-repo</id>
                    ...
                </repository>
            </repositories>
        </profile>
        <profile>
            <id>profile-2</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <repositories>
                <repository>
                    <id>profile-2-repo</id>
                    ...
                </repository>
            </repositories>
        </profile>
        ...
    </profiles>
    ...
</project>
```

这导致存储库列表：`profile-2-repo, profile-1-repo, global-repo`.

###  profiles 陷阱

我们已经提到，将 profiles 添加到构建中可能会破坏项目的可移植性。我们甚至强调了 profiles 可能会破坏项目可移植性的情况。但是，作为更连贯的讨论的一部分，有必要重申这些观点，以便在使用 profiles 时避免一些陷阱。

使用 profiles 时需要牢记两个主要问题领域。首先是外部属性，通常用于插件配置。这些构成破坏项目可移植性的风险。另一个更微妙的领域是自然 profiles 集的不完整规范。

#### 外部属性

外部属性定义涉及在 `pom.xml` 外部定义但未在其内部的相应 profile 中定义的任何属性值。POM 中属性最明显的用法是在插件配置中。虽然没有属性当然有可能破坏项目的可移植性，但这些小生物可能会产生微妙的影响，导致构建失败。例如，当团队中的其他用户尝试在没有类似的 `settings.xml` 时，在 `settings.xml` 中指定的 profile 中指定 appserver 路径可能会导致您的集成测试插件失败。考虑以下 Web 应用程序项目的 `pom.xml` 代码片段：

``` xml
<project>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.myco.plugins</groupId>
                <artifactId>spiffy-integrationTest-plugin</artifactId>
                <version>1.0</version>
                <configuration>
                    <appserverHome>${appserver.home}</appserverHome>
                </configuration>
            </plugin>
            ...
        </plugins>
    </build>
    ...
</project>
```

现在，在您的本地 `${user.home}/.m2/settings.xml`，您有：

``` xml
<settings>
    ...
    <profiles>
        <profile>
            <id>appserverConfig</id>
            <properties>
                <appserver.home>/path/to/appserver</appserver.home>
            </properties>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>appserverConfig</activeProfile>
    </activeProfiles>
    ...
</settings>
```

当您构建**集成测试**生命周期阶段时，您的集成测试通过，因为您提供的路径允许测试插件安装和测试此 Web 应用程序。

*但是*，当您的同事尝试构建**集成测试**时，他的构建失败了，抱怨它无法解析插件配置参数 `<appserverHome>`，或者更糟糕的是，该参数的值，从字面上看`${appserver.home}`，是无效的（如果它警告你的话） .

恭喜，您的项目现在是不可移植的。在你的 `pom.xml` 文件中内联这个 profile 可以帮助缓解这个问题，明显的缺点是每个项目层次结构（允许继承的影响）现在必须指定这个信息。由于 Maven 为项目继承提供了良好的支持，因此可以将这种配置粘贴在团队级 POM 或类似的 `<pluginManagement>` 部分中，并简单地继承路径。

另一个不太吸引人的答案可能是开发环境的标准化。然而，这往往会损害 Maven 能够提供的生产力增益。

#### 自然 profiles 集的不完整规范

除了上述可移植性破坏者之外，您的 profiles 很容易无法涵盖所有情况。当你这样做时，你通常会让你的一个目标环境变得干涸。让我们再看一次上面的示例 `pom.xml` 片段：

``` xml
<project>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.myco.plugins</groupId>
                <artifactId>spiffy-integrationTest-plugin</artifactId>
                <version>1.0</version>
                <configuration>
                    <appserverHome>${appserver.home}</appserverHome>
                </configuration>
            </plugin>
            ...
        </plugins>
    </build>
    ...
</project>
```

现在，考虑以下 profile，它将在 `pom.xml` 中内联指定：

``` xml
<project>
    ...
    <profiles>
        <profile>
            <id>appserverConfig-dev</id>
            <activation>
                <property>
                    <name>env</name>
                    <value>dev</value>
                </property>
            </activation>
            <properties>
                <appserver.home>/path/to/dev/appserver</appserver.home>
            </properties>
        </profile>

        <profile>
            <id>appserverConfig-dev-2</id>
            <activation>
                <property>
                    <name>env</name>
                    <value>dev-2</value>
                </property>
            </activation>
            <properties>
                <appserver.home>/path/to/another/dev/appserver2</appserver.home>
            </properties>
        </profile>
    </profiles>
    ..
</project>
```

此 profile 看起来与上一个示例中的 profile 非常相似，但有一些重要的例外：它显然是面向开发环境的，添加了一个名为 `appserverConfig-dev-2` 的新 profile ，并且它有一个激活部分，当系统属性包含 “ env=dev" 用于名为 `appserverConfig-dev` 的 profile，"env=dev-2" 用于名为`appserverConfig-dev-2`. 所以，执行：

``` 
mvn -Denv=dev-2 integration-test
```

将导致成功构建，应用名为 `appserverConfig-dev-2` 的 profile 的属性。当我们执行

``` 
mvn -Denv=dev integration-test
```

它将导致成功构建应用名为 `appserverConfig-dev` 的 profile 给出的属性。但是，执行：

``` 
mvn -Denv=production integration-test
```

不会成功构建。为什么？因为，生成的非插值文字值`${appserver.home}`将不是部署和测试 Web 应用程序的有效路径。在编写 profile 时，我们没有考虑生产环境的情况。“production”环境（env=production）与 “test” 甚至可能是 “local” 一起构成了一组自然的目标环境，我们可能希望为其构建集成测试生命周期阶段。这个自然集的不完整规范意味着我们有效地将我们的有效目标环境限制在开发环境中。你的队友——可能还有你的经理——不会看到其中的幽默。当您构建 profile 以处理此类情况时，请务必解决整个目标排列集。

顺便说一句，用户特定的 profiles 可能会以类似的方式行事。这意味着当团队添加新的开发人员时，用于处理不同环境的用户 profiles 可以发挥作用。虽然我认为这*可以*作为对新手有用的培训，但以这种方式将它们扔给狼群并不好。同样，一定要考虑*整套 profiles 。

### 我如何知道在构建过程中哪些 profiles  有效？

确定活动 profiles 将帮助用户了解在构建期间执行了哪些特定 profiles。我们可以使用[Maven 帮助插件](https://maven.apache.org/plugins/maven-help-plugin/)来了解在构建过程中哪些 profiles 有效。

``` 
mvn help:active-profiles
```

让我们用一些小示例来帮助我们更好地理解该插件的 *active-profiles* 目标。

从 `pom.xml` 中的 profiles 的最后一个示例中，您会注意到有两个 profiles 被命名 `appserverConfig-dev` 和 `appserverConfig-dev-2`，并且被赋予了不同的属性值。如果我们继续执行：

```
mvn help:active-profiles -Denv=dev
```

结果将是具有 “env = dev” 激活属性的 profile  ID的项目符号列表以及声明它的源。请参阅下面的示例。

```
The following profiles are active:

- appserverConfig-dev (source: pom)
```

现在，如果我们在 `settings.xml` 中声明了一个 profile（请参阅 `settings.xml` 中的 profile 示例）并且已设置为活动 profile 并执行：

``` 
mvn help:active-profiles
```

结果应该是这样的

```
The following profiles are active:

- appserverConfig (source: settings.xml)
```

尽管我们没有激活属性，但 profile 已被列为活动。为什么？就像我们之前提到的，在 `settings.xml` 中设置为活动 profile 的 profile 会自动激活。

现在，如果我们有类似于 `settings.xml` 中一个 profile 的东西，它已被设置为活动 profile ，并且还触发了 POM 中的 profile 。您认为哪个 profile 会对构建产生影响？

``` xml
mvn help:active-profiles -P appserverConfig-dev
```

这将列出激活的 profiles ：

```
The following profiles are active:

- appserverConfig-dev (source: pom)
- appserverConfig (source: settings.xml)
```

即使它列出了两个活动 profiles ，我们也不确定其中哪一个已被应用。要查看对构建的影响，请执行：

``` 
mvn help:effective-pom -P appserverConfig-dev
```

这将将此构建配置的有效 POM 打印到控制台。请注意，`settings.xml` 中的 profiles 比 POM 中的 profiles 具有更高的优先级。所以这里应用的 profiles 是 `appserverConfig`，不是 `appserverConfig-dev`.

如果要将插件的输出重定向到名为 `effective-pom.xml` 的文件，请使用命令行选项 `-Doutput=effective-pom.xml`。

### 命名约定

到目前为止，您已经注意到 profiles 是解决针对不同目标环境的不同构建配置要求问题的一种自然方式。上面，我们讨论了解决这种情况的“自然集” profiles 的概念，以及考虑所需的整个 profiles 集的重要性。

然而，如何组织和管理该集合的演变的问题也很重要。正如一个优秀的开发人员努力编写自我文档化的代码一样，您的 profile ID 为他们的预期用途提供提示很重要。一种很好的方法是使用公共系统属性触发器作为 profile 名称的一部分。对于由系统属性 **env** 触发的profiles ，这可能会产生类似 **env-dev**、**env-test** 和 **env-prod** 的名称。这样的系统为如何激活针对特定环境的构建留下了非常直观的提示。因此，要激活测试环境的构建，您需要通过发出以下命令来激活 **env-test ：**

``` 
mvn -Denv=test <phase>
```

只需将 profile ID 中的“=”替换为“-”即可获得正确的命令行选项。

