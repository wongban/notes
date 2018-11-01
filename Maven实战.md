# Maven实战

## 生命周期和插件

&emsp;&emsp;Maven的生命周期就是为了对所有的构建过程进行抽象和统一。这个生命周期包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有构建步骤。

&emsp;&emsp;生命周期是抽象的，实际任务由插件完成。这种思想与设计模式中的模板方法（Template Method）非常相似。

### 生命周期

&emsp;&emsp;Maven拥有三套相互独立的生命周期，它们分别为clean、default和site。

+ clean: 清理项目
+ default: 定义了真正构建时所需要执行的所有步骤，它是所有生命周期中最核心的部分
+ site: 建立和发布项目站点，Maven能够基于POM所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息

&emsp;&emsp;每个生命周期包含一些阶段（phase），这些阶段是有顺序的，并且后面的阶段依赖于前面的阶段，用户和Maven最直接的交互方式就是调用这些生命周期阶段。`mvn package`表示执行默认生命周期阶段`package`

### 插件目标

&emsp;&emsp;Maven的核心仅仅定义了抽象的生命周期，具体的任务交由插件完成。为了能够复用代码，插件往往能够完成多个任务。这些功能聚集在一个插件里，每个功能就是一个插件目标`Plugin Goal`。

+ `maven-compiler-plugin`的`compile`目标：`compiler:compile`
+ `maven-surefire-plugin`的`test`目标：`surefire:test`

### 插件绑定

生命周期的阶段与插件的目标相互绑定，以完成某个具体的构建任务。

生命周期|阶段`phase`|插件目标`goal`(打包类型`jar`)|作用
-|-|-|-
clean|`pre-clean`<br>`clean`<br>`post-clean`|-<br>`maven-clean-plugin:clean`<br>-|清理前需要完成的工作<br>清理上一次构建生成的文件<br>清理后需要完成的工作
default|`validate`<br>`initialize`<br>`generate-sources`<br>`process-sources`<br>`generate-resources`<br>`process-resources`<br>`compile`<br>`process-classes`<br>`generate-test-sources`<br>`process-test-sources`<br>`generate-test-resources`<br>`process-test-resources`<br>`test-compile`<br>`process-test-classes`<br>`test`<br>`prepare-package`<br>`package`<br>`pre-integration-test`<br>`integration-test`<br>`post-integration-test`<br>`verify`<br>`install`<br>`deploy`|<br><br><br><br><br>`maven-resources-plugin:resources`<br>`maven-compiler-plugin:compile`<br><br><br><br><br>`maven-resources-plugin:testResources`<br>`maven-compiler-plugin:testCompile`<br><br>`maven-surefire-plugin:test`<br><br>`maven-jar-plugin:jar`<br><br><br><br><br>`maven-install-plugin:install`<br>`maven-deploy-plugin:deploy`|<br><br><br>处理项目主资源文件<br><br>复制主资源文件至主输出目录<br>编译主代码至主输出目录<br><br><br>处理项目测试资源文件<br><br>复制测试资源文件至测试输出目录<br>编译测试代码至测试输出目录<br><br>执行测试用例<br><br>创建项目`jar`包<br><br><br><br><br>将项目输出构建安装到本地仓库<br>将项目输出构建部署到远程仓库|
site|`pre-site`<br>`site`<br>`post-site`<br>`site-deploy`|<br>`maven-site-plugin:site`<br><br>`maven-site-plugin:deploy`|生成项目站点之前需要完成的工作<br>生成项目站点文档<br>生成项目站点之后需要完成的工作<br>将生成的项目站点发布到服务器上

> default生命周期还有很多其他阶段，默认它们没有绑定任何插件，因此也没有任何实际行为。

#### 自定义绑定

&emsp;&emsp;除了内置绑定以外，用户还能够自己选择将某个插件目标绑定到生命周期的某个阶段上，这种自定义绑定方式能让Maven项目在构建过程中执行更多更富特色的任务。

```xml
<build>
  <plugins>
    <plugin>
      <!-- 如果是Maven官方插件org.apache.maven.plugins，就可以省略groupId配置 -->
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-source-plugin</artifactId>
      <version>2.1.1</version>
      <executions>
        <!-- 每个execution用来配置一个任务 -->
        <execution>
          <id>attach-sources</id>
          <!-- 很多插件的目标在编写时已经定义了默认绑定阶段。所以可以去掉phase -->
          <phase>verify</phase>
          <!-- 配置制定要执行的插件目标 -->
          <goals>
            <goal>jar-no-fork</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

### 插件配置

命令行插件配置：`mvn install -Dmaven.test.skip=true`

```xml
<build>
  <plugins>
    <!-- POM中插件全局配置 -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>2.1</version>
      <!-- configuration位于plugin下 -->
      <configuration>
        <source>1.5</source>
        <target>1.5</target>
      </configuration>
    </plugin>

    <!-- POM中插件任务配置 -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-antrun-plugin</artifactId>
      <version>1.3</version>
      <executions>
        <execution>
          <id>ant-validate</id>
          <phase>validate</phase>
          <goals>
            <goal>run</goal>
          </goals>
          <!-- configuration位于execution下 -->
          <configuration>
            <tasks>
              <echo>I'm bound to validate phase.</echo>
            </tasks>
          </configuration>
        </execution>
        <execution>
          <id>ant-verify</id>
          <phase>verify</phase>
          <goals>
            <goal>run</goal>
          </goals>
          <configuration>
            <tasks>
              <echo>I'm bound to verify phase.</echo>
            </tasks>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
<build>
```

### 获取插件信息

`mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin:2.1`

可以省去版本信息，让Maven自动获取最新版本来进行表述

使用**插件目标前缀**替换坐标：`mvn help:describe -Dplugin=compiler`

插件目标：`mvn help:describe -Dplugin=compiler -Dgoal=compile`

详细信息：`mvn help:describe -Dplugin=compiler -Ddetail`

### 从命令行调用插件

可以通过`mvn`命令激活生命周期阶段，从而执行那些绑定在生命周期阶段上的插件目标。但Maven还支持直接从命令行调用插件目标。

> 有些任务不适合绑定在生命周期上，例如`maven-help-plugin:describe`，我们不需要在构建项目的时候去描述插件信息，又如`maven-dependency-plugin:tree`，我们也不需要在构建项目的时候去显示依赖树。

+ `mvn help:describe -Dplugin=compiler`
+ `mvn dependency:tree`

### 插件解析机制

```xml
<!-- 插件远程仓库 -->
<pluginRepositories>
  <pluginRepository>
    <id>central</id>
    <name>Maven Plugin Repository</name>
    <url>http://repo1.maven.org/maven2</url>
    <layout>default</layout>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
    <releases>
      <updatePolicy>never</updatePolicy>
    </releases>
  </pluginRepository>
</pluginRepositories>
```

#### 解析插件版本

仓库元数据：`groupId/artifactId/maven-metadata.xml`

&emsp;&emsp;Maven在超级POM中为所有核心插件设定了版本。

&emsp;&emsp;如果用户使用某个插件时没有设定版本，而这个插件又不属于核心插件的范畴，Maven就会去检查所有仓库中可用的版本，然后做出选择。

&emsp;&emsp;Maven遍历本地仓库和所有远程插件仓库，将该路径下的仓库元数据归并后，就能计算出latest和release的值。latest表示所有仓库中该构件的最新版本，而release表示最新的非快照版本。

#### 解析插件前缀

仓库元数据：`groupId/maven-metadata.xml`
