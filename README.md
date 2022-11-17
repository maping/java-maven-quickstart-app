# A Java Maven Quickstart App

## 1. Create a repo manually

### 1.1 Create a repo on GitHub
Click "Repositories",then click "New" button,input "java-maven-quickstart-app", leave all other input as default, click "Create repository".

### 1.2 Create a Java helloword app by Maven
```console
$ cd code
$ mvn archetype:generate -DgroupId=xyz.javaneverdie.quickstart -DartifactId=quickstartapp -DarchetypeArtifactId=maven-archetype-quickstart -Dversion=1.0-SNAPSHOT -DinteractiveMode=false
```

### 1.3 Init repo 
```console
$ cd quickstartapp
$ echo "# java-maven-quickstart-app" >> README.md
$ git init
$ git add -A
$ git commit -m "add java maven archetype quickstart app"
$ git branch -M main
$ git remote add origin https://github.com/maping/java-maven-quickstart-app.git
$ git push -u origin main
```

## 2. Modify pom.xml

### 2.1 增加源代码编码和 java compile 版本属性设定
```code
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>
```
>重要：java compiler 版本要与实际使用的 JDK 保持一致。

### 2.2 增加 maven-shade-plugin 
由于程序是有 main 方法的，默认打成的 jar 包，是不能直接运行的。需要把 main 方法的信息需要写入到 jar 文件中的 META-INF/MANIFEST.MF 文件，所以需要使用 maven-shade-plugin。
```code
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.4.1</version>
        <configuration>
          <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
              <mainClass>xyz.javaneverdie.quickstart.App</mainClass>
            </transformer>
          </transformers>
        </configuration>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
</build>
```

### 2.3 修改所有的依赖包到最新的 Release 版本
访问 https://repository.sonatype.org 查找每个依赖的最新的 Release 版本 (2022-11-17 更新)

## 3. Compile and run application
```console
$ cd quickstartapp
$ mvn clean package
$ java -jar target/quickstartapp-1.0-SNAPSHOT.jar
Hello World!
```

恭喜你，你已经成功构建了一个 Java 应用！

## 4. 部署项目到 Nexus 仓库

### 4.1 安装并配置好 Nexus 私有仓库

### 4.2 在 Nexus 中创建 Hosted 类型仓库
- 创建 maven-quickstart-release 仓库
- 创建 maven-quickstart-snapshot 仓库

## 4.3 修改 pom.xml
添加 distributionManagement 元素
```code
  <distributionManagement>
    <repository>
      <id>quickstart-release</id>
      <name>A Java Maven Quickstart Releases Repository</name>
      <url>http://localhost:8081/repository/maven-quickstart-release/</url>
    </repository>
    <snapshotRepository>
      <id>quickstart-snapshot</id>
      <name>A Java Maven Quickstart Snapshots Repository</name>
      <url>http://localhost:8081/repository/maven-quickstart-snapshot/</url>
    </snapshotRepository>
  </distributionManagement>
```

## 4.4 修改 ~/.m2/settings.xml
添加 server 元素
```code
    <server>
      <id>quickstart-releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>

    <server>
      <id>quickstart-snapshot</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
```
>重要：server 中的 id 值必须要和 distributionManagement 中的 repository 中的 id 值一致。

## 4.5 部署项目
```console
$ cd quickstartapp
$ mvn clean package
$ mvn deploy
```
Maven 会根据 pom.xml 文件中的版本号中是否带有 -SNAPSHOT（必须全部大写）来判断这个是快照版本还是正式版本。
- 如果是快照版本，在 mvn deploy 时会自动发布到快照版本库中；使用快照版本的模块，在不更改版本号的情况下，直接编译打包时，Maven 会自动从仓库服务器上下载最新的快照版本。
- 如果是正式版本，在 mvn deploy 时会自动发布到快照版本库中；

提示部署成功后，访问 http://localhost:8081/ ，搜索 quickstart，会发现在 maven-quickstart-snapshot Repo 中有刚刚部署成功的 quickstartapp-1.0-SNAPSHOT。

## 5. 发布正式版

### 5.1 修改 pom.xml，增加 scm 元素
```code
  <scm>
    <connection>scm:git:https://github.com/maping/java-maven-quickstart-app.git</connection>
    <developerConnection>scm:git:https://github.com/maping/java-maven-quickstart-app.git</developerConnection>
    <url>https://github.com/maping/java-maven-quickstart-app.git</url>
    <tag>HEAD</tag>
  </scm>
```
### 5.2 修改 pom.xml，增加 maven-release-plugin 元素
```code
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-release-plugin</artifactId>
        <version>2.5.3</version>
        <configuration>
          <tagBase>https://github.com/maping/java-maven-quickstart-app.git</tagBase>
          <branchBase>https://github.com/maping/java-maven-quickstart-app.git</branchBase>
        </configuration>
      </plugin>
```
### 5.3 正式发布
>重要：发布正式版的条件：
- 所有自动化测试通过
- 项目没有配置任何快照版本的依赖
- 项目没有配置任何快照版本的插件
- 项目源代码已经全部提交到代码仓库

>温馨提示：执行 `git push` 后，请再执行一次 `git pull` 确保本地和 github 代码仓库完全一致

执行 `mvn release:prepare` 进行发布前准备
```console
$ mvn release:prepare
...
What is the release version for "java-maven-quickstart-app"? (xyz.javaneverdie.quickstart.quickstartapp) 1.0: :
What is SCM release tag or label for "java-maven-quickstart-app"? (xyz.javaneverdie.quickstart.quickstartapp) quickstartapp-1.0: : 1.0
What is the new development version for "java-maven-quickstart-app"? (xyz.javaneverdie.quickstart.quickstartapp) 1.1-SNAPSHOT: :
...
```
执行 `mvn release:perform` 签出代码，执行`mvn release:perform`构建新版本，并部署到仓库中
```console
$ mvn release:perform
...
...Uploaded to quickstart-release: http://localhost:8081/repository/maven-quickstart-release/xyz/javaneverdie/quickstart/quickstartapp/1.0/quickstartapp-1.0-javadoc.jar (403 kB at 3.9 MB/s)
...
```
发布成功后，访问 http://localhost:8081/ ，搜索 quickstart，会发现在 maven-quickstart-release Repo 中有刚刚部署成功的 quickstartapp-1.0。

不仅项目的主构件会被生成并且发布到仓库中，基于该主构件的 -source.jar 和 -javadoc.jar 也一并生成了，这真是太方便了！

查看本地和 github 仓库中的 pom.xml 文件，发现 version 都改为 1.1-SNAPSHOT 了。

查看 github 仓库的 Tags，发现有一个新的 Tag：quickstartapp-1.0，点击该 Tag 右边的 ...，可以依此 Tag 创建 Release。
