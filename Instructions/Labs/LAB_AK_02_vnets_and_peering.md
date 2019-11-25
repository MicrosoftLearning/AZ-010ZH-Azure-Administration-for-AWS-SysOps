# 实验室 02 练习解答

## 说明

1. 登录到 azure 门户，打开 bash Cloud Shell
1. 复制以下 CLI 脚本
1. 将脚本粘贴到 Bash Cloud Shell 中
1. 有些任务可能需要 1 分钟以上的时间，请等待脚本完成并检查输出
1. 如果发生错误，请与讲师联系

> 注意：学生应按照实验室 01 设置默认订阅

```sh
# AZ-010 LAB2 解决方案
# ---------------
# 依赖关系：LAB1-解决方案已到位（已创建 WestRG 和 EastRG）
# ---------------
# 警告：几个步骤需要花费 1 分钟以上
# 如果遇到错误，请耐心等待并咨询讲师

# ----------启动----------

# ----创建 WestVNet 和 WestSubNet1----
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24

# ----验证已创建的 WestVNet 和 WestSubNet1----
az network vnet subnet list --resource-group WestRG \
--vnet-name WestVNet --output table

# ----创建 EastVNet 和 EastSubNet1 ----
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24

# ----在 EastVNet 上创建 EastSubNet2----
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24

# ----验证已创建的 EastVNet----
az network vnet list --output table

# ----确认已创建的 EastVNet 子网----
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table

# ------启动从西到东对等互连网络------
# ----捕获变量中的 EastVNet ID----
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId

# ----从东到西对等互连----
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access

# ----验证对等互连状态----
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table

# ------启动从西到东对等互连网络------
# ----捕获变量中的 WestVNet ID----
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId

# echo ----从东到西对等互连----
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access

# ----验证对等互连状态----
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
```
