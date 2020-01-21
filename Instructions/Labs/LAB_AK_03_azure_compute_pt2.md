# 实验 03 pt 2 练习解答

## 说明

1. 将以下 CLI 脚本复制到记事本等编辑器中
   1. 本地保存文件
1. 登录到 azure 门户，打开 bash Cloud Shell
1. 从本地文件复制 CLI 脚本并将脚本粘贴到 Bash Cloud Shell 中
1. 有些任务可能需要 1 分钟以上的时间，请等待脚本完成并检查输出
1. 如果发生错误，请与讲师联系

> 注意：学生应按照实验室 01 设置默认订阅

```sh
# AZ-010 LAB3 解决方案
# ---------------
# 依赖关系：
#   LAB1-解决方案已到位（已创建 WestRG 和 EastRG）
# ---------------
# 警告：几个步骤需要花费 1 分钟以上
# 如果遇到错误，请耐心等待并咨询讲师

# ----------启动----------

# ----主要脚本----

# ----练习 2：创建和测试 Ubuntu 规模集 ----
# ----创建 Ubuntu 规模集 ----

# ----创建规模集资源组----
az group create --name EastScaleRG --location eastus

# ----创建规模集----
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys

# ----定义自动缩放配置文件----
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1

# ----创建规则以自动横向扩展----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1

# ----创建规则以自动缩小----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1

# --------测试 Ubuntu 规模集（可选任务）--------

# ----使用以下输出为规模集 Ubuntu 服务器会话连接 SSH ----
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers

# *******************************手动步骤**************************************
# ----使用以上输出通过 SSH 连接到 Ubuntu 规模集实例 0----
# ----转到 LAB03 EXERCISE2 TASK2 并完成以下步骤
# ----     * 安装/运行压力
# ----     * 查看比例
# *********************************************************************************

# ----通过以下 CLI 命令运行以清理规模集演示----
# az group delete --name EastScaleRG --yes --no-wait
# 运行最后的可选任务以测试 Ubuntu 规模集
# **捕获上述最终连接信息输出，以使用 SSH 连接至 Ubuntu 规模集实例 0**
# 然后转到 LAB03 EXERCISE2 TASK2 并完成以下步骤
# * 安装/运行压力
# * 查看比例
# * 清理规模集
