# Customizing IIS Logging

注册实现 ILogPlugin或 ILogPluginEx接口的COM组件，将自定义日志记录模块添加到IIS服务器。

## Creating Custom Logging Modules for IIS

1. 编写一个COM组件，适应为**ILogPluginEx/ILogPlugin**接口的相关定义
2. 利用Regsvr32.exe进行注册COM组件
3. 通过在配置数据库的/ LM / LOGGING路径中添加密钥，将日志记录模块添加到可用日志记录模块列表中。
4. 通过使用管理用户界面或使用脚本，将自定义日志记录模块添加到特定网站的服务中。

## Creating Logging User Interface Modules for IIS

- 无用户接口模块
- 使用内置的用户接口模块：可以借用内置IIS日志记录模块随附的用户接口模块之一。在元数据库中/ LM / LOGGING路径的Microsoft IIS日志文件格式键下指定的用户接口模块GUID最适合此用途，因为它提供了基本的配置功能。
- 新建用户接口模块：您可以创建自己的实现适当接口的COM对象。

建立的过程同上相近

## 开发进度以及问题汇总

采用C# .net进行的Demo开发，简单实现了ILogPulgin接口的内容，但发现无法实现IIS模块的嵌入，即自定义无效

![image-20210129145209851](C:\Users\CRUN\AppData\Roaming\Typora\typora-user-images\image-20210129145209851.png)

后讨论对流程进行拆分

1. COM组件的开发（单独流程
2. 实现COM组件的定义
3. IIS COM组件的加载



COM组件的开发流程

目前采用C#进行组件开发，利用C++进行测试调用





## 其余一些思考

能否直接通过HttpModule对请求进行截获，进行日志处理

![img](https://pic002.cnblogs.com/images/2010/66691/2010111515341046.gif)



## 后续学习内容

Windows COM组件开发相关

## 问题汇总

1. windows COM组件开发流程
2. 服务器采用的框架以及服务器自身提供的接口如何
3. 服务器IIS自带的Logging框架如何
4. 如何将COM组件包装成DLL