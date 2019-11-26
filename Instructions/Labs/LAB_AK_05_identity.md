# 实验室 05 练习解答

## 说明

1. 将以下 CLI 脚本复制到记事本等编辑器中
1. 标题为 `# ----在运行之前编辑这些值----` 的位置部分
1. 编辑值，使它们代表你的环境，并在本地保存文件
1. 登录到 azure 门户，打开 bash Cloud Shell
1. 检查依赖项是否到位（请参阅脚本顶部的注释）
1. 从本地文件复制 CLI 脚本并将脚本粘贴到 Bash Cloud Shell 中
1. 有些任务可能需要 1 分钟以上的时间，请等待脚本完成并检查输出
1. 如果发生错误，请与讲师联系

> 注意：学生应按照实验室 01 设置默认订阅

```sh
# AZ-010 LAB5 解决方案
# 设置以下值：
#   订阅 ID
# 在 Azure CLI Bash 中运行以下命令之前
# 使用：az account list -o table 查看订阅

# ----------启动----------

# ----在运行之前编辑这些值----
subscriptionID=[**用于实验室的订阅 ID**]

# ----主要脚本----
# ----设置默认订阅----
az account set --subscription $subscriptionID
# ---注意默认订阅---
az account list --output table

# ----创建 WestRG ----
az group create --location westus --name WestRG --output table

# ----创建 EastRG ----
az group create --location eastus --name EastRG --output table

# ----列出资源组----
az group list -o table
```
