# Customizing IIS Logging

注册实现 ILogPlugin或 ILogPluginEx接口的COM组件，将自定义日志记录模块添加到IIS服务器。

## Creating Custom Logging Modules for IIS

1. 编写一个COM组件，适应为ILogPluginEx\ILogPlugin接口的相关定义
2. 利用Regsvr32.exe进行注册COM组件
3. 通过在配置数据库的/ LM / LOGGING路径中添加密钥，将日志记录模块添加到可用日志记录模块列表中。
4. 通过使用管理用户界面或使用脚本，将自定义日志记录模块添加到特定网站的服务中。

## Creating Logging User Interface Modules for IIS

- 无用户接口模块
- 使用内置的用户接口模块：可以借用内置IIS日志记录模块随附的用户接口模块之一。在元数据库中/ LM / LOGGING路径的Microsoft IIS日志文件格式键下指定的用户接口模块GUID最适合此用途，因为它提供了基本的配置功能。
- 新建用户接口模块：您可以创建自己的实现适当接口的COM对象。

建立的过程同上相近

## 后续学习内容

Windows COM组件开发相关

## 问题汇总

1. windows COM组件开发流程
2. 服务器采用的框架以及服务器自身提供的接口如何
3. 服务器IIS自带的Logging框架如何
4. 如何将COM组件包装成DLL