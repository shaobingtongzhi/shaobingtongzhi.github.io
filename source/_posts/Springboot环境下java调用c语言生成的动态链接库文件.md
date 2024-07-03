---
title: Springboot环境下java调用c语言生成的动态链接库文件
date: 2024-04-25
categories:
  - 学习笔记
  - 09-java相关笔记
tags:
  - JNI
  - DLL
  - so
  - 动态链接库
---

# 背景

在项目中要求部分算法程序必须使用 C 或 C++ 进行开发，而项目使用的是 Java 程序，框架是Springboot，好在java中提供了JNI机制，允许 Java 代码调用本地（Native）代码，比如 C 或 C++ 编写的代码

# 基本流程

## 生成JNI头文件

用于链接动态链接库

1. 编写一个Java类 AddJni.java

```java
package net.xxx.jni;

public class AddJni {
    public static native int Add(int x,int y);
    public static native int Sub(int x,int y);
    public static native int Mul(int x,int y);
    public static native int Div(int x,int y);
}
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-04-25_15-16-51.5zo75wjzxeo0.2hylh27hjqw0.jpg)



2. 执行javah命令，生成 .h文件，为了链接 c 程序所用

```sh
#注意执行当前命令所在的目录是 项目/src/main/java 目录
javah net.xxx.jni.AddJni
#执行完成后会在 项目/src/main/java 目录下生成一个 .h 的文件，把这个文件拷贝到 项目/src/main/resources/native 目录下，很关键
# .h 头文件包含了本地方法的声明
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-04-25_15-20-21.3ko24hy2jh40.3l7y5nvkkp20.jpg)

## 编写本地 C 语言方法的代码

1. 在 项目/src/main/resources/native 目录下新建一个 add.c 文件

```c
#include "net_xxx_jni_AddJni.h"
JNIEXPORT jint JNICALL Java_net_zhiboredian_jni_AddJni_Add
  (JNIEnv * env, jclass jc, jint x, jint y){
    return x+y;
  }
JNIEXPORT jint JNICALL Java_net_zhiboredian_jni_AddJni_Sub
  (JNIEnv * env, jclass jc, jint x, jint y){
    return x-y;
  }
JNIEXPORT jint JNICALL Java_net_zhiboredian_jni_AddJni_Mul
  (JNIEnv * env, jclass jc, jint x, jint y){
    return x*y;
  }
JNIEXPORT jint JNICALL Java_net_zhiboredian_jni_AddJni_Div
  (JNIEnv * env, jclass jc, jint x, jint y){
    return x/y;
  }
```

> 注意这个 C 文件里方法的名称必须和头文件 net_xxx_jni_AddJni.h 中的方法名称完全一致

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-04-25_15-23-14.3oubcd610lu0.qb8nw1zp1pc.jpg)

2. 生成动态链接库文件 .so 或者 .dll 结尾的文件，.so 实在 Linux 系统下使用，.dll 是在 Windows 系统下使用

```sh
# 注意：在控制台里进入add.c所在的目录执行下面的命令

gcc -fPIC -I "D:/Program Files/Java/jdk-1.8/include" -I "D:/Program Files/Java/jdk-1.8/include/win32" -shared -o libadd.dll add.c

# 命令中的D:/Program Files/Java/jdk-1.8/include 和 D:/Program Files/Java/jdk-1.8/include/win32 路径下包含了编译用到的 jni.h 和 jni_md.h

# libadd.dll 表示 生成 dll 文件，可修改为 libadd.so 文件名称随意
# add.c 表示源文件
# windows 下安装gcc ,参考链接：https://blog.csdn.net/K346K346/article/details/130870292
```

## 加载本地库

1. 在 util 包下建一个 NativeLoader 类用来加载库文件，这个类是网上找的

```java
public class NativeLoader {
    /**
     * 加载项目下的native文件，DLL或SO
     *
     * @param dirPath 需要扫描的文件路径，项目下的相对路径
     * @throws IOException
     * @throws ClassNotFoundException
     */
    public synchronized static void loader(String dirPath) throws IOException, ClassNotFoundException {
        Enumeration<URL> dir = Thread.currentThread().getContextClassLoader().getResources(dirPath);
        // 获取操作系统类型
        String systemType = System.getProperty("os.name");
        //String systemArch = System.getProperty("os.arch");
        // 获取动态链接库后缀名
        String ext = (systemType.toLowerCase().indexOf("win") != -1) ? ".dll" : ".so";
        while (dir.hasMoreElements()) {
            URL url = dir.nextElement();
            String protocol = url.getProtocol();
            if ("jar".equals(protocol)) {
                JarURLConnection jarURLConnection = (JarURLConnection) url.openConnection();
                JarFile jarFile = jarURLConnection.getJarFile();
                // 遍历Jar包
                Enumeration<JarEntry> entries = jarFile.entries();
                while (entries.hasMoreElements()) {
                    JarEntry jarEntry = entries.nextElement();
                    String entityName = jarEntry.getName();
                    if (jarEntry.isDirectory() || !entityName.startsWith(dirPath)) {
                        continue;
                    }
                    if (entityName.endsWith(ext)) {
                        loadJarNative(jarEntry);
                    }
                }
            } else if ("file".equals(protocol)) {
                File file = new File(url.getPath());
                loadFileNative(file, ext);
            }

        }
    }

    private static void loadFileNative(File file, String ext) {
        if (null == file) {
            return;
        }
        if (file.isDirectory()) {
            File[] files = file.listFiles();
            if (null != files) {
                for (File f : files) {
                    loadFileNative(f, ext);
                }
            }
        }
        if (file.canRead() && file.getName().endsWith(ext)) {
            try {
                System.load(file.getPath());
                System.out.println("加载native文件 :" + file + "成功!!");
            } catch (UnsatisfiedLinkError e) {
                System.out.println("加载native文件 :" + file + "失败!!请确认操作系统是X86还是X64!!!");
            }
        }
    }

    /**
     * @throws IOException
     * @throws ClassNotFoundException
     * @Title: scanJ
     * @Description 扫描Jar包下所有class
     */
    /**
     * 创建动态链接库缓存文件，然后加载资源文件
     *
     * @param jarEntry
     * @throws IOException
     * @throws ClassNotFoundException
     */
    private static void loadJarNative(JarEntry jarEntry) throws IOException, ClassNotFoundException {

        File path = new File(".");
        //将所有动态链接库dll/so文件都放在一个临时文件夹下，然后进行加载
        //这是应为项目为可执行jar文件的时候不能很方便的扫描里面文件
        //此目录放置在与项目同目录下的natives文件夹下
        String rootOutputPath = path.getAbsoluteFile().getParent() + File.separator;
        String entityName = jarEntry.getName();
        String fileName = entityName.substring(entityName.lastIndexOf("/") + 1);
        System.out.println(entityName);
        System.out.println(fileName);
        File tempFile = new File(rootOutputPath + File.separator + entityName);
        // 如果缓存文件路径不存在，则创建路径
        if (!tempFile.getParentFile().exists()) {
            tempFile.getParentFile().mkdirs();
        }
        // 如果缓存文件存在，则删除
        if (tempFile.exists()) {
            tempFile.delete();
        }
        InputStream in = null;
        BufferedInputStream reader = null;
        FileOutputStream writer = null;
        try {
            //读取文件形成输入流
            in = NativeLoader.class.getResourceAsStream(entityName);
            if (in == null) {
                in = NativeLoader.class.getResourceAsStream("/" + entityName);
                if (null == in) {
                    return;
                }
            }
            NativeLoader.class.getResource(fileName);
            reader = new BufferedInputStream(in);
            writer = new FileOutputStream(tempFile);
            byte[] buffer = new byte[1024];

            while (reader.read(buffer) > 0) {
                writer.write(buffer);
                buffer = new byte[1024];
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if (in != null) {
                in.close();
            }
            if (writer != null) {
                writer.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            System.load(tempFile.getPath());
            System.out.println("加载native文件 :" + tempFile + "成功!!");
        } catch (UnsatisfiedLinkError e) {
            System.out.println("加载native文件 :" + tempFile + "失败!!请确认操作系统是X86还是X64!!!");
        }

    }
}
```

2. 在需要用的地方调用加载方法加载一下

```JAVA
NativeLoader.loader("native"); //会自动加载 native 目录下的 动态链接库文件
```

![](https://github.com/hfshaobing/picx-images-hosting/raw/master/20240606/Snipaste_2024-04-25_15-24-40.4jup764060e0.6ykp4vzs95g0.jpg)

## 调用本地方法

```java
 int add = AddJni.Add(1, 2);
 int sub = AddJni.Sub(1, 2);
 int mul = AddJni.Mul(1, 2);
 int div = AddJni.Div(1, 2);
```



# 参考链接

https://blog.csdn.net/qq_28483283/article/details/90259838

https://blog.csdn.net/m0_37257009/article/details/98727047

https://blog.csdn.net/m0_37257009/article/details/98739889

[安装gcc](https://blog.csdn.net/K346K346/article/details/130870292)

