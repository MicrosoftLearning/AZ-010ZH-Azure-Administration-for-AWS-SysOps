# 实验室 05 练习解答

## 说明

1. 将以下 CLI 脚本复制到记事本等编辑器中
   1. 标题为 `# ----在运行之前编辑这些值----` 的位置部分
   1. 编辑值，使其代表你的环境，否则将出现**错误**
   1. 本地保存文件
1. 登录到 azure 门户，打开 bash Cloud Shell
1. 检查依赖项是否到位（请参阅脚本顶部的注释）
1. 从本地文件复制 CLI 脚本并将脚本粘贴到 Bash Cloud Shell 中
1. 有些任务可能需要 1 分钟以上的时间，请等待脚本完成并检查输出
1. 如果发生错误，请与讲师联系

> 学生应按照实验 01 设置默认订阅 (subscriptionID)
>
> 为用户域创建变量。
> 如果帐户电子邮件地址为 eric@contoso.com，则命令为 `my_domain=ericcontoso.onmicrosoft.com`.
>
> * *如有需要，请咨询讲师。*

```sh
# AZ-010 LAB5 解决方案
# ---------------
# 依赖关系: 实验 1 - 解决方案已到位（已创建 WestRG）
# ---------------
# 设置以下值：
#   订阅 ID
#   my_domain
# 在 Azure CLI Bash 中运行以下命令之前

# ----------启动----------

# +++++++++++++++++++++++++++++++++++++++++++++++++++++
# ----在运行之前编辑这些值----
subscriptionID=[**用于实验室的订阅 ID**]
# 为用户域创建变量
my_domain=[**UsernameEmaildomain.onmicrosoft.com**]
# 创建唯一的 AD 用户密码
password_ad_user=[**sTR0ngP@ssWorD543%**]
# +++++++++++++++++++++++++++++++++++++++++++++++++++++

# ----主要脚本----
# ----设置默认订阅----
az account set --subscription $subscriptionID
# ---注意默认订阅---
＃----创建用户帐户名 (user-principal-name)----
my_user_account=AZ010@$my_domain

#----创建用户帐户----
az ad user create \
    --display-name AZ010Tester \
    --password $password_ad_user \
    --user-principal-name $my_user_account

#----列出 AD 用户----
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
# ====================================================
＃----注意上面输出中列出的新 AD 用户----
# ====================================================

＃----向新用户添加角色（“所有者”`）----
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

＃----列出你所创建用户的角色分配（注意更改之处）----
az role assignment list --assignee $my_user_account -g WestRG

#----创建限制软件安装的策略----
＃----创建策略定义。
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.'\
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All

#----订阅级别的范围----
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'

＃----显示你新创建的策略----
az policy assignment show --name 'SQL12AZ010'

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#----练习 2：使用查询资源管理器监视日志和警报
# ******************************************************************
# ---------------------------手动步骤---------------------------
＃----导航到 Log Analytics 查询演示----
#---------------https://portal.loganalytics.io/demo ----------------
＃-----按照实验 5 练习 2 的说明使用查询资源管理器

```

> **练习 2 - 任务 1 - 查看监视日志和警报**
> 访问演示环境**
>
> 1. 在新的浏览器选项卡中导航到 [Log Analytics 查询演示](https://portal.loganalytics.io/demo)。
> 2. 使用查询资源管理器
>     1. 选择查询浏览器（右上方）。
>     2. 展开收藏夹，然后选择“所有有错误的 Syslog 记录”**。
>     3. 注意，查询已添加到编辑窗格中。注意查询的结构。
>     4. 运行查询。浏览返回的记录。
>     5. 请按照讲师的其他步骤操作。
>     6. 当你有时间尝试其他“收藏夹”和“保存的查询”时。
