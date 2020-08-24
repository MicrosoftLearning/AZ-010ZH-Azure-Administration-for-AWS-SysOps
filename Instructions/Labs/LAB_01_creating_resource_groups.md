---
lab:
    title: '创建资源组'
    module: '模块 1：Azure 管理'
---

# 实验室 01：创建资源组

Azure Cloud Shell CLI 简介

## 学生实验室手册

## 方案

本实验介绍在 Azure 门户中设置将用于课程的订阅，并介绍使用 Azure CLI 命令的 Azure Cloud Shell。  课程讲师将根据特定的课程环境讲解配置 Azure 订阅所需的任何步骤。你将通过配置两个资源组在 Azure Cloud Shell 中运行第一个 CLI 命令。

## 目标

完成本实验室后，你将能够：

* 使用 Azure 门户设置订阅（课程讲师根据需要讲解步骤）
* 使用 Azure CLI 命令创建资源组

## 实验室设置

* **预计用时**：20 分钟

## 说明

### 开始前

#### 设置任务

1. 按照课程讲师提供的步骤，使用 Azure 门户配置将用于本课程的 Azure 帐户。

## 练习 1：在使用 Cloud Shell 的 Azure CLI 中开始，创建 2 个资源组

本练习的主要任务如下：

1. 启动 Azure Cloud Shell
1. 设置将用于实验室的默认订阅
1. 使用 CLI 命令创建 WestRG 资源组
1. 使用 CLI 命令创建 EastRG 资源组

### 练习 1 - 任务 1：打开 Cloud Shell

**在 Azure Cloud Shell 中设置订阅**

1. 在门户顶部，单击“**Cloud Shell**”图标以打开“Cloud Shell”窗格。

1. 在“Cloud Shell”界面中，选择“**Bash**”。

1. 在 **Cloud Shell** 命令提示符，键入以下命令，然后按 **Enter** 列出与用于门户登录的帐户关联的所有订阅。

```sh
az account list --output table
```

1. 查看订阅列表，确认是否有订阅被标记为 "default `true`"
1. 如果没有为你所需的订阅设置默认值，请重置默认订阅
1. 在 **Cloud Shell** 命令提示符，在“**your desired subscriptionID**”中键入以下命令，然后按 **Enter** 设置默认订阅。

```sh
# 用你的订阅 ID 或订阅名称替换 --subcription 值
az account set --subscription [1111a1a1-22bb-3c33-d44d-e5e555ee5eee]
az account list --output table
```

4. 查看订阅列表，并确保将正确的订阅被标记为 "default `true`"

### 任务 2：使用 CLI 创建 WestRG 资源组

1. 在 **Cloud Shell** 命令提示符下，键入以下命令，创建位于美国西部区域的 WestRG 资源组。

```sh
az group create --location westus --name WestRG --output table
```

2. 在 **Cloud Shell** 命令提示符下，键入以下命令，列出美国西部区域可用的资源组。

```sh
az group list --output table
```

3. 验证列出了新创建的 WestRG

### 任务 3：使用 CLI 创建 EastRG 资源组

1. 在 **Cloud Shell** 命令提示符下，键入以下命令，创建位于美国东部区域的 EastRG 资源组。

```sh
az group create --location eastus --name EastRG --output table
```

2. 在 **Cloud Shell** 命令提示符下，键入以下命令，列出美国东部区域可用的资源组。

```sh
az group list --output table
```

3. 验证是否列出了新创建的 EastRG

> **结果**：在本实验中，你已经配置了默认的 Azure 订阅，并创建了两个资源组，我们将在随后的实验中进一步使用它们。
