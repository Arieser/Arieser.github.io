---
title: 发布构件到Maven中央仓库
date: 2018-06-19 11:40:00
tags:
    - maven
---



总结的工具包为使用方便决定发布到maven中央仓库，因为第一次发布，遇到很多问题，简要记录一下。

![1529380073135](/images/1529380073135.png)

```html
Sonatype官网：http://www.sonatype.org/
```

<!--more-->

Step 1: 注册Sonatype用户

> 注册地址：<https://issues.sonatype.org/secure/Signup!default.jspa> 
>
> oss地址：[https://oss.sonatype.org](https://oss.sonatype.org/) ，用于查询构件

Step2: 创建issue

1. create issue:

   ```
   Project：Community Support - Open Source Project Repository Hosting (OSSRH)
   Issue Type: new Project
   --- next ---
   Summary:   <简要项目介绍>
   Group id: me.silloy <需要时自己的域名，同时在工程中使用>
   ```

   点击create，创建issue

2. 下图查看已创建的issue

   ![1529386058988](/images/1529386058988.png)

Step 3: 等待issue审批通过

> 一般需要1-2天，审批通过后会收到邮件通知，在自己提交的issue下面可以看到Sonatype工作人员的回复。同时issue状态修改为resolved。

![1529386399022](/images/1529386399022.png)



Step 4: 发布准备

1. 生成gpg密钥，并发布到PGP密钥服务器，见引用

   **简要介绍win10环境下gpg密钥生成方法**

   - 下载Gpg4win， 安装

   - 依次执行一下命令

     > 版本检查: `gpg --version`, 我用的是2.2.8
     >
     > 生成key： `gpg --gen-key`
     >
     > 检查本地key： gpg --list-keys
     >
     > 发布公钥：gpg --keyserver hkp://pool.sks-keyservers.net --send-keys A3434534534
     >
     > 校验是否发布成功：gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 732796B4

     ```shell
     # List all available gpg servers:
     $ gpg-connect-agent --dirmngr 'keyserver --hosttable'
     ```

2. 修改maven设置

   - 修改maven全局配置文件setting.xml， 增加一下内容

     ```xml
     <servers>
         <server>
             <id>oss</id>
             <username>Harvey.Su</username>
             <password><![CDATA[password]]></password>
         </server>
     </servers>
     ```

   - 修改pom文件，加入需要的信息

     ```xml
     <groupId>me.silloy</groupId>
         <artifactId>zjtools</artifactId>
         <version>0.0.1</version>
         <packaging>jar</packaging>
         <description>a java tool package</description>
     
         <licenses>
             <license>
                 <name>The Apache Software License, Version 2.0</name>
                 <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
             </license>
         </licenses>
     
         <developers>
             <developer>
                 <name>sushaohua</name>
                 <email>sshzh90@gmail.com</email>
             </developer>
         </developers>
     
     
         <scm>
             <connection>scm:git:git@github.com:silloy/zjtools.git</connection>
            <developerConnection>scm:git:git@github.com:silloy/zjtools.git</developerConnection>
             <url>git@github.com:silloy/zjtools.git</url>
         </scm>
     ```

   - 增加一个名为oss的profile (⭐⭐⭐⭐⭐)

     ```xml
     <build>
         <plugins>
             <plugin>
                 <groupId>org.apache.maven.plugins</groupId>
                 <artifactId>maven-compiler-plugin</artifactId>
                 <version>${maven-compiler-plugin.version}</version>
                 <configuration>
                     <compilerVersion>${maven.compiler.source}</compilerVersion>
                     <source>${maven.compiler.source}</source>
                     <target>${maven.compiler.source}</target>
                     <encoding>${project.build.sourceEncoding}</encoding>
                 </configuration>
             </plugin>
             <!-- Source -->
             <plugin>
                 <groupId>org.apache.maven.plugins</groupId>
                 <artifactId>maven-source-plugin</artifactId>
                 <version>${maven-source-plugin.version}</version>
                 <executions>
                     <execution>
                         <id>attach-sources</id>
                         <phase>compile</phase>
                         <goals>
                             <goal>jar-no-fork</goal>
                         </goals>
                     </execution>
                 </executions>
             </plugin>
             <!-- Javadoc -->
             <plugin>
                 <groupId>org.apache.maven.plugins</groupId>
                 <artifactId>maven-javadoc-plugin</artifactId>
                 <version>${maven-javadoc-plugin.version}</version>
                 <executions>
                     <execution>
                         <id>attach-javadocs</id>
                         <phase>package</phase>
                         <goals>
                             <goal>jar</goal>
                         </goals>
                     </execution>
                 </executions>
                 <configuration>
                     <encoding>UTF-8</encoding>
                     <charset>UTF-8</charset>
                     <additionalOptions>
                         <additionalOption>-Xdoclint:none</additionalOption>
                     </additionalOptions>
                 </configuration>
             </plugin>
         </plugins>
         <resources>
             <resource>
                 <directory>src/main/java</directory>
                 <includes>
                     <include>**/*.properties</include>
                 </includes>
                 <filtering>false</filtering>
             </resource>
         </resources>
     </build>
     <!-- profiles -->
     <profiles>
         <profile>
             <id>sonar</id>
             <activation>
                 <!--<activeByDefault>true</activeByDefault>-->
                 <jdk>[1.8,)</jdk>
             </activation>
             <!--<properties>-->
             <!--<additionalparam>-Xdoclint:none</additionalparam>-->
             <!--</properties>-->
             <build>
                 <!--<finalName>${project.artifactId}-${project.version}-SNAPSHOT</finalName>-->
                 <finalName>${project.artifactId}-${project.version}</finalName>
                 <plugins>
                     <!-- GPG -->
                     <plugin>
                         <groupId>org.apache.maven.plugins</groupId>
                         <artifactId>maven-gpg-plugin</artifactId>
                         <version>${maven-gpg-plugin.version}</version>
                         <executions>
                             <execution>
                                 <id>sign-artifacts</id>
                                 <phase>verify</phase>
                                 <goals>
                                     <goal>sign</goal>
                                 </goals>
                             </execution>
                         </executions>
                     </plugin>
                 </plugins>
             </build>
         </profile>
     </profiles>
     
     <distributionManagement>
         <snapshotRepository>
             <id>oss</id>
             <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
         </snapshotRepository>
         <repository>
             <id>oss</id>
             <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
         </repository>
     </distributionManagement>
     ```

     javadoc 不规范的情况下，可以把maven-javadoc-plugin注释掉。

     **特别注意**：snapshotRepository 与 repository 中的 id 一定要与 setting.xml 中 server 的 id 保持一致。这里我们都设置为oss。 

Step 5:  上传构件到OSS

执行命令：

```shell
mvn clean deploy -P sonar -Darguments="gpg.passphrase=密钥密码"
```

看到如下提示信息，说明deploy成功

```shell
Uploading: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1.jar
Uploaded: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1.jar (34 kB at 966 B/s)
Uploading: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1.pom
Uploaded: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1.pom (9.7 kB at 985 B/s)
Downloading: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/maven-metadata.xml
Uploading: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/maven-metadata.xml
Uploaded: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/maven-metadata.xml (296 B at 5 B/s)
Uploading: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1-sources.jar
Uploaded: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1-sources.jar (20 kB at 1.9 kB/s)
Uploading: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1-sources.jar
Uploaded: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1-sources.jar (20 kB at 10 kB/s)
```

Step 6: 在OSS中发布构件

> 登录https://oss.sonatype.org，可以在`Staging Repositories`中查看到已上传的构件，可进行模糊查询，快速定位到自己的构件，状态为open，勾选，然后点击close按钮，接下来系统会自动验证该构件是否满足指定要求，当验证完毕后，状态会变为 Closed，最后，点击 Release 按钮来发布该构件 。
>
> 备注：切记是要发布release版本的构件，不能是snapshot，不然不会出现在Staging Repositories里面。 

Step 7: 通知 Sonatype“构件已成功发布”

> 需要在曾经创建的 Issue 下面回复一条“构件已成功发布”的评论，这是为了通知 Sonatype 的工作人员为需要发布的构件做审批( I released the component has been successfully, please approval, thank you!)，发布后会关闭该 Issue 。

Step 8: 等待审批，1~2天，审批通过后会收到邮件通知。

![1529637162176](/images/1529637162176.png)

Step 9: 在https://oss.sonatype.org/#stagingRepositories找到自己的构建，点击release

Step 9: 从中央仓库搜索自己发布的构件

> 地址：<http://search.maven.org/> 

Step 10: <http://search.maven.org/> 上搜索自己的构件 ，大功告成，可以在项目中引用啦。以后发布就简单了，不需要每次都审核。

问题：

1. gpg错误处理`Enter passphrase: gpg: gpg-agent is not available in this session`

   可能是版本问题，或者没有安装gpg-agent，详见<https://askubuntu.com/questions/860370/gpg-agent-cant-be-reached> ，具体步骤

   1. linux 系统解决方案

   - 安装gpg2  `sudo apt install gpgv2`

   - 然后在maven的settings.xml中加入两个属性，主要要在激活的profile里面 

     ```xml
     <profile>
         <properties>
             <gpg.executable>gpg2</gpg.executable>
             <gpg.useagent>true</gpg.useagent>
         </properties>
     </profile>
     ```

   - `mvn clean deploy -P oss`， 参看 [Avoid gpg signing prompt when using Maven release plugin](https://stackoverflow.com/questions/14114528/avoid-gpg-signing-prompt-when-using-maven-release-plugin)

   2. windows 解决方案 (Windows下Gpg4win、Git、ssh-pageant配置)[https://www.mjollnir.cc/archives/216.html]

      .git/config修改

      ```shell
      [gpg]
      	program = gpg
      ```

2. Failed to execute goal org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy (default-deploy) on project zjtools: Failed to deploy artifacts: Could not transfer artifact me.silloy:zjtools:jar:0.0.1 from/to oss (https://oss.sonatype.org/service/local/staging/deploy/maven2/): Access denied to: https://oss.sonatype.org/service/local/staging/deploy/maven2/me/silloy/zjtools/0.0.1/zjtools-0.0.1.jar, ReasonPhrase: Forbidden. -> [Help 1]

   ![1529459800811](/images/1529459800811.png)

   答案：[引用][problem2]，提交给sonatype就可以，工作人员会开权限

3. 可以在 <http://search.maven.org/>  搜索到，但是不能在 http://mvnrepository.com/ 搜索到，是因为更新频率不一样，等一天左右就好了，参见工作人员回复

   ![1529720659785](/images/1529720659785.png)

4. `gpg --list-keys` 出现 unknown 解决

   ```shell
   gpg --edit-key user@useremail.com
   
   gpg> trust
   
   Please decide how far you trust this user to correctly verify other users' keys
   (by looking at passports, checking fingerprints from different sources, etc.)
   
     1 = I don't know or won't say
     2 = I do NOT trust
     3 = I trust marginally
     4 = I trust fully
     5 = I trust ultimately
     m = back to the main menu
   
   Your decision? 5
   gpg> save
   ```



References

[发布到中央仓库](https://skyao.gitbooks.io/learning-maven/content/publish/central/)

[上传自己的构件(Jar)到Maven中央仓库](https://my.oschina.net/u/2335754/blog/476676)

[将 Smart 构件发布到 Maven 中央仓库](https://my.oschina.net/huangyong/blog/226738)

[Maven-008-Nexus 私服部署发布报错 Failed to deploy artifacts: Failed to transfer file: ... Return code is: 4XX, ReasonPhrase: ... 解决方案](https://www.cnblogs.com/fengpingfan/p/5197608.html)

[Why am I getting a “401 Unauthorized” error in Maven?](https://stackoverflow.com/questions/24830610/why-am-i-getting-a-401-unauthorized-error-in-maven)

[将项目发布到 Maven 中央仓库踩过的坑](http://brianway.github.io/2017/05/17/release-to-maven-central-repo/)

[[gpg —list-keys command outputs uid [ unknown \] after importing private key onto a clean install](https://unix.stackexchange.com/questions/407062/gpg-list-keys-command-outputs-uid-unknown-after-importing-private-key-onto/407070)

[problem2]: https://www.oschina.net/question/1444646_2277979