title: maven将jar包deploy到Jfrog Artifactory
categories: Backend
date: 2017-06-11 23:08:30
tags: [Maven, Jenkins, Jfrog]
---

可以采取两种方式，第一种是通过maven的deploy命令，第二种是通过jenkins的`Artifactory`插件。

## 1. maven的deploy命令

在`settings.xml`中配置servers和profiles：

    <?xml version="1.0" encoding="UTF-8"?>
    <settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd" xmlns="http://maven.apache.org/SETTINGS/1.1.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <servers>
        <server>
          <username>uname</username>
          <password>passwd</password>
          <id>central</id>
        </server>
        <server>
          <username>uname</username>
          <password>passwd</password>
          <id>snapshots</id>
        </server>
      </servers>
      <profiles>
        <profile>
          <repositories>
            <repository>
              <snapshots>
                <enabled>false</enabled>
              </snapshots>
              <id>central</id>
              <name>libs-release</name>
              <url>http://localhost:8081/artifactory/libs-release</url>
            </repository>
            <repository>
              <snapshots />
              <id>snapshots</id>
              <name>libs-snapshot</name>
              <url>http://localhost:8081/artifactory/libs-snapshot</url>
            </repository>
            <repository>
              <snapshots>
                <enabled>false</enabled>
              </snapshots>
              <id>central-local</id>
              <name>libs-release-local</name>
              <url>http://localhost:8081/artifactory/libs-release-local</url>
            </repository>
            <repository>
              <snapshots />
              <id>snapshots-local</id>
              <name>libs-snapshot-local</name>
              <url>http://localhost:8081/artifactory/libs-snapshot-local</url>
            </repository>
          </repositories>
          <pluginRepositories>
            <pluginRepository>
              <snapshots>
                <enabled>false</enabled>
              </snapshots>
              <id>central</id>
              <name>plugins-release</name>
              <url>http://localhost:8081/artifactory/plugins-release</url>
            </pluginRepository>
            <pluginRepository>
              <snapshots />
              <id>snapshots</id>
              <name>plugins-snapshot</name>
              <url>http://localhost:8081/artifactory/plugins-snapshot</url>
            </pluginRepository>
          </pluginRepositories>
          <id>artifactory</id>
        </profile>
      </profiles>
      <activeProfiles>
        <activeProfile>artifactory</activeProfile>
      </activeProfiles>
    </settings>

<!--more-->

然后，在`pom.xml`中增加`distributionManagement`节点配置：

    <distributionManagement>
        <snapshotRepository>
            <id>snapshots</id>
            <name>snapshots</name>
            <url>http://ci.betalpha.com:8081/artifactory/libs-snapshot-local</url>
        </snapshotRepository>
        <repository>
            <id>central</id>    
            <name>releases</name>
            <url>http://ci.betalpha.com:8081/artifactory/libs-release-local</url>
        </repository>
    </distributionManagement>
    
执行命令：

    $ mvn deploy

这种方式比较简单，可以在本地执行，也可以在jenkins中作为shell命令执行。

## 2. jenkins的`Artifactory`插件

首先在jenkin中安装`Artifactory`插件：

    Manage Jenkins -> Manage Plugins -> Artifactory Plugin

然后，需要配置servers：

    Manage Jenkins -> Configure System -> Artifactory

![jenkins-artifactory.png](/images/post/jenkins-artifactory.png)

最后，在项目中配置artifactory和maven：

![artifactory_config_1.png](/images/post/artifactory_config_1.png)
![artifactory_config_2.png](/images/post/artifactory_config_2.png)
![artifactory_config_3.png](/images/post/artifactory_config_3.png)

这种方式，项目本身不需要关心如何deploy，而且在jenkins上每一个build都会直接关联到JFrog Artifactory上。

### 参考

- [Artifactory lugin](https://wiki.jenkins-ci.org/display/JENKINS/Artifactory+Plugin)