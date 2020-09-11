Hadoop官网下载3.3.0源码包.

Windows有最长路径限制，解压Hadoop源码到某盘根目录，并重命名文件夹，名字尽量短，如C:\h3s

查看building.txt，找到Windows区域，按所需依赖准备环境。


我的环境：
WIN10 19041.450
JDK 8u261 （官网说支持JAVA11为运行时，但不支持用JAVA11编译）
Cmake最新版、zlib最新版、git最新版、protocolbuffer最新版、maven最新版


步骤：
安装git cmake Python JDK8并添加环境变量，注意安装git时选用UNIX命令要选最后一个
ENV：
ZLIB_HOME    JAVA_HOME      MAVEN_HOME
PATH添加JAVA_HOME\bin      MAVEN_HOME\bin
添加环境变量Platform  值为x64


zlib.net下载最新版压缩包并解压到C盘根目录
nmake命令在系统cmd下找不到命令，用VS命令行运行：
cd C:\zlib-1.2.11
nmake -f win32/Makefile.msc 

ProtocolBuffer最新版
https://github.com/protocolbuffers/protobuf/releases
https://github.com/protocolbuffers/protobuf/releases/download/v3.13.0/protobuf-java-3.13.0.zip
https://github.com/protocolbuffers/protobuf/releases/download/v3.13.0/protoc-3.13.0-win64.zip
将两个压缩包解压，然后将 protoc.exe 复制到 protobuf-$version\src 目录下，将src文件夹加入环境变量
PATH添加C:\protobuf-java-3.13.0\protobuf-3.13.0\src
命令行进入C:\protobuf-java-3.13.0\protobuf-3.13.0\java
mvn install



VS DEV COMMAND PROMPT执行：
#进入VS安装目录找到设置平台的批处理
cd C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\VC\Auxiliary\Build
 vcvarsall.bat x86_amd64

VS打开native.sln和winutils.sln升级到最新



maven编译源码命令（不进行测试，节省时间）：
mvn package -Pdist,native-win  -Dmaven.test.skip=true -Dtar -Dmaven.javadoc.skip=true







遇到的错误及解决办法（有些地方可能忘了记录错误只记了解决办法）：

Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.7:run (make) on project hadoop-hdfs-native-client: An Ant BuildException has occured: exec returned: 1
[ERROR] around Ant part ...<exec failοnerrοr="true" dir="C:\h3s\hadoop-hdfs-project\hadoop-hdfs-native-client\target/native" executable="msbuild">... @ 11:135 in C:\h3s\hadoop-hdfs-project\hadoop-hdfs-native-client\target\antrun\build-main.xml

     cmake和MsBuild莫名报错，暂时在hadoop-hdfs-project\hadoop-hdfs-native-client\pom.xml中，屏蔽它们的错误
camke和msbuild，failonerror值false改为true



在源码根目录pom.xml中repositories中加入阿里云镜像，snapshots开启，并将jboss仓库注释掉，否则会下载很慢
		<repository>
        <id>nexus-aliyun</id>
        <name>nexus-aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>


在C:\h3s\hadoop-hdfs-project\hadoop-hdfs-native-client\target\native\下创建C:\h3s\hadoop-hdfs-project\hadoop-hdfs-native-client\target\native\bin\RelWithDebInfo目录



在编译到
[INFO] --- frontend-maven-plugin:1.6:install-node-and-yarn (install node and yarn) @ hadoop-yarn-applications-catalog-webapp ---
[INFO] Installing node version v8.11.3
[INFO] Downloading https://nodejs.org/dist/v8.11.3/win-x64/node.exe to C:\repo\com\github\eirslett\node\8.11.3\node-8.11.3-win-x64.exe
[INFO] No proxies configured
[INFO] No proxy was configured, downloading directly
时，下载过于缓慢。进入yarn官网，https://classic.yarnpkg.com/zh-Hans/docs/install/ 发现yarn最新版本为，nodejs官网最新版本为14.10.1，遂将C:\h3s\hadoop-yarn-project\hadoop-yarn\hadoop-yarn-applications\hadoop-yarn-applications-catalog\hadoop-yarn-applications-catalog-webapp中pom.xml中的 <nodeVersion>v8.11.3</nodeVersion> <yarnVersion>v1.7.0</yarnVersion> 改为 <nodeVersion>v14.10.1</nodeVersion> <yarnVersion>v1.22.5</yarnVersion>尝试。
下载nodejs还是过于缓慢，复制命令行nodejs.exe安装包地址到浏览器手动下载，并复制到命令行提示的目录并重命名

后在Netch中发现有Node代理模式，开启后mvn编译不再卡住，编译成功。


