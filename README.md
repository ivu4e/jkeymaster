JKeyMaster
==========
[![Build Status](https://travis-ci.org/tulskiy/jkeymaster.svg?branch=master)](https://travis-ci.org/tulskiy/jkeymaster)

这是一个Java库，它提供单个接口来注册多个平台的全局热键:

  * Windows
  * 基于X11的系统（理论上仅在某些Linux发行版和PCBSD上进行了测试）
  * Mac OSX
详细用法参考：[java程序中注册设置全局系统级热键快捷键，支持Windows、Linux、Mac OSX](https://www.ivu4e.com/blog/lang/2020-06-20/483.html)

构建
--------

需要 Java 1.6+ 和 Maven 用于构建

    mvn clean install
    
对于 jkeymaster-dbus, 你需要下载并构建几个依赖

    git clone git@github.com:tulskiy/libmatthew-java.git
    cd libmatthew-java
    mvn clean install

    git clone git@github.com:tulskiy/dbus-java.git
    cd dbus-java
    mvn clean install -DskipTests
    
然后您可以使用配置文件`linux`从项目根目录进行构建
    
    mvn -P linux clean install 

Maven
-----

Release 版本:
    
    <dependencies>
        <dependency>
            <groupId>com.github.tulskiy</groupId>
            <artifactId>jkeymaster</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>
    
Snapshot 版本:
    
    <dependencies>
        <dependency>
            <groupId>com.github.tulskiy</groupId>
            <artifactId>jkeymaster</artifactId>
            <version>1.3-SNAPSHOT</version>
        </dependency>
    </dependencies>
    
    <repositories>
        <repository>
            <id>snapshots-repo</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </repository>
    </repositories>

日志
-------

JKeyMaster uses [slf4j](https://github.com/qos-ch/slf4j) as the logging framework. To get output from the library you
should include a slf4j binding to your classpath.

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-jdk14</artifactId>
        <version>1.7.13</version>
    </dependency>

测试
-------

To test that the library works on your system, run the jar file

    java -cp jkeymaster-1.2.jar com.tulskiy.keymaster.AWTTest

It will open a simple window. At the top is a field that listens to key presses.
Choose your hotkey and press `Grab` button. Check log output to see if it failed.
If the log is clean, you should be able to press the hotkey anywhere and it should
popup a message.

There's also a `Grab media keys` button that should register media keys
(only Windows and Linux) - Play/Pause, Next Track, Previous Track, Stop.

If you get UnsatisfiedLinkError on linux, it is probable that you have an older jna version installed, see [#19](https://github.com/tulskiy/jkeymaster/issues/19)
you will need to either remove libjna-java from your system, or add `-Djna.nosys=true` to jvm args

API 示例
-----------

Main class is `Provider`. To get provider for current platform:

    Provider provider = Provider.getCurrentProvider(useSwingEventQueue);
    
where `useSwingEventQueue` is a boolean parameter specifying whether to fire events
on swing event queue or just simple thread. Usefull when your hotkey listener will 
directly access or modify some swing components.

Provider supports two methods for registering hotkeys - one accepts AWT KeyStroke:

    provider.register(KeyStroke.getKeyStroke("control shift PLUS"), listener);

the other accepts a `MediaKey` parameter:

    provider.register(MediaKey.MEDIA_NEXT_TRACK, listener);

For simplicity reasons, it is currently impossible to unregister some particular
hotkey, but all hotkeys can be reset.

When you're done using the library, you need to reset hotkeys and stop the provider:

    provider.reset();
    provider.stop();
    
DBus
----

On some systems DBus does not allow registering hotkeys for media keys. To fix this, you can use
`GnomeMediaProvider` class. Just create new instance and register media hotkeys with it. It implements same api as
other providers

    final GnomeMediaProvider provider = new GnomeMediaProvider("test");
    provider.register(MediaKey.MEDIA_STOP, new HotKeyListener() {...});
    
however, this requires extra native libraries. You will need to have your users install

    # apt-get install libunixsocket-java
     
then, if you get UnsatisfiedLinkError, you will need to find where libunix-java.so is located and add this path as 
`-Djava.library.path=$YOUR_PATH`. It's usually /usr/lib/jni or /usr/local/lib/jni
    
Clojure
-------

If you are interested in nicer api from clojure, check out a wrapper library 
developed by Stian Håklev: https://github.com/houshuang/keymaster-clj

Feedback
--------

I would really appreciate it if you could test the library and post some bug reports.
I've tested on Win 7, Win XP, Ubuntu 10.10 and 11.04. I've tested Mac OSX code
only in 32-bit virtual machine, and a bit on a real 64-bit machine. So feedback is greatly
appreciated, especially from people who have real Mac with real Apple keyboard,
BSD users, people with some advanced keyboards, and just anybody who wants to help.
