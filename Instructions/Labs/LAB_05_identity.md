---
lab:
    title: '创建用户、组和策略。监视日志和警报。'
    module: '模块 5：Azure 身份识别'
---

# 实验室 01：Azure 身份识别

创建用户、组和策略以及监视日志和警报

## 学生实验室手册

## 方案

在本实验室中，你将在 Cloud Shell 中使用带有 Bash 接口的 CLI 命令来管理 Azure 标识。  你将使用 Azure 基于角色的访问控制、Azure Policy，并使用 Query Explorer 查看监控。

## 目标

完成本实验室后，你将能够：

* 使用 Azure 基于角色的访问控制创建和配置用户和组
* 创建限制软件安装的策略
* 使用 Query Explorer 在 Azure 门户中查看监控日志和警报

## 实验室设置

* **预计用时**：45 分钟

## 说明

### 开始前

#### 设置任务

1. 配置将用于本课程的 Azure 帐户。
2. **模块 1: Azure 管理，实验室： 创建资源组** 已配置 WestRG 资源组。

### 练习 1：创建用户、组和策略

本练习的主要任务如下：

1. 添加用户和组
1. 创建限制软件安装的策略

#### 练习 1 - 任务 1：打开 Cloud Shell

**添加用户**

1. 为用户域创建变量。

> 对于 eric@contoso.com 这一电子邮件地址，该命令将为 `my_domain=ericcontoso.onmicrosoft.com`。
>
> * *如有需要，请咨询讲师。*

2. 为用户域编辑变量。

```sh
# 为用户域创建变量
# user@contoso.com =>  usercontoso.onmicrosoft.com

my_domain=<email+service>.onmicrosoft.com
```

**创建用户帐户**

1. 创建用户帐户名

```sh
my_user_account=AZ010@$my_domain
```

2. 创建唯一的强密码（请编辑密码！）

```sh
# 将密码修改为唯一密码（删除前导“!”或错误）
az ad user create \
    --display-name AZ010Tester \
    --password !sTR0ngP@ssWorD543%* \
    --user-principal-name $my_user_account
```

3. 记下显示名称、密码和 --user-principal-name

**管理用户和组**

1. 列出 AD 用户。

```sh
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
```

2. 应列出你在先前步骤中创建的用户。
3. 列出所有角色分配。

```sh
az role assignment list --all -o table
```

4. 注意此列表的初始状态。
5. 列出资源组的角色分配。

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

6. 向新用户添加一个角色（“所有者”）。

```sh
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#az role assignment create --role "Owner" --assignee <assignee object id> --resource-group <resource_group>
```

**查看用户和组中的更改**

1. 重复列出资源组角色分配（注意更改之处）。

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

2. 列出你所创建用户的角色分配（注意更改之处）。

```sh
az role assignment list --assignee $my_user_account -g WestRG #--output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

## 任务 2 – 创建限制软件安装的策略

**部署软件限制策略**

> *在 [Github](https://github.com/Azure/azure-policy/tree/master/samples/built-in-policy/require-sqlserver-version12) 上查看 rules.json 和 parameters.json（在下面脚本中使用）*

1. 创建策略定义。

```sh
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.' \
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All
```

2. 订阅级别的范围。

```sh
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'
```

3. 列出角色分配。

```sh
az 策略分配列表
```

4. 显示你新创建的策略。

```sh
az policy assignment show --name 'SQL12AZ010'
```

**检查“合规性”**

1. 返回到 Azure 策略服务页面。
2. 选择“合规性”。并将合规状态过滤器设置为“所有合规状态”
3. 查看策略状态和定义。
## 练习 2：使用查询资源管理器监视日志和警报

1. 监视日志和警报

### 练习 2 - 任务 1 - 查看监视日志和警报

**访问演示环境**

1. 在新的浏览器选项卡中导航到 [Log Analytics 查询演示](https://portal.loganalytics.io/demo)。
2. 使用查询资源管理器
    1. 选择查询浏览器（右上方）。
    2. 展开收藏夹，然后选择“所有有错误的 Syslog 记录”**。
    3. 注意，查询已添加到编辑窗格中。注意查询的结构。
    4. 运行查询。浏览返回的记录。
    5. 请按照讲师的其他步骤操作。
    6. 当你有时间尝试其他“收藏夹”和“保存的查询”时。
