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
      <id>quickstart-snapshot</id>
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
添加 distributionManagement 元素
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
