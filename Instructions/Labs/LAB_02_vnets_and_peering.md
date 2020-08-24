---
lab:
    title: 'Azure 虚拟网络和对等互连'
    module: '模块 2：Azure 网络'
---
    
# 实验室 02：Azure 虚拟网络和对等互连

## 学生实验室手册

## 方案

在本实验室中，你将在 Cloud Shell 中使用带有 Bash 接口的 CLI 命令来管理虚拟网络 (VNet)。你将创建跨资源组和区域的 VNet，并配置 VNET 对等互连。

## 目标

完成此实验室之后，你将可以使用 Azure CLI 执行以下操作：

* 在所需资源组中创建和配置具有子网的 VNet。
* 在 VNet 之间配置 VNET 对等互连。

## 实验室设置

* **预计用时**：30 分钟

## 说明

### 开始前

#### 设置任务

1. 已在 **模块 1** 中配置 EastRG 和 WestRG 资源组：**Azure 管理，实验室：创建资源组**。

## 练习 1：创建带有子网的虚拟网络

本练习的主要任务如下：

1. 创建具有子网的 West VNet。
1. 验证是否创建了 West 网络和子网
1. 创建具有子网的 East VNet。
1. 验证是否创建了 East 网络和子网。

### 练习 1 - 任务 1 创建具有子网的 West VNet

1. 在 **Cloud Shell** 命令提示符下，键入以下命令，以使用 WestSubNet1 子网创建 WestVNet 虚拟网络。

```sh
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24
```

2. 验证是否创建了 West 网络和子网。

```sh
az network vnet list --output table
```

3. 验证是否创建了 West 网络和子网。

```sh
az network vnet subnet list --resource-group WestRG --vnet-name WestVNet --output table
```

### 任务 2 创建具有子网的 East VNet

1. 在 **Cloud Shell** 命令提示符下，键入以下命令，以使用 EastSubNet1 子网创建 EastVNet 虚拟网络。

```sh
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24
```

2. 你可以为现有的 VNet 添加额外的子网。
3. 在 **Cloud Shell** 命令提示符下，键入以下命令，以创建 EastVNet EastSubNet2 子网。

```sh
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24
```

4. 验证是否创建了 East NetWork (EastVNet)

```sh
az network vnet list --output table
```

5. 验证是否创建了 East 网络子网

```sh
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table
```

### 任务 3：创建从 West 到 East 的对等互连网络

1. 使用 `remote-vnet-id` CLI 命令在 West 和 East VNet 之间创建对等互连
1. 在 **Cloud Shell** 在命令提示符下，键入以下命令以在变量中捕获 EastVNet ID。

```sh
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId
```

3. 键入以下命令以将 WestVNet 与 EastVNet 对等互连

```sh
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access
```

> *注意：未使用 Cloud Shell `--remote-vnet-id $EastVNetId` 并会发出警告，但这在较旧的CLI Shell 中可能是必需的*

4. 键入以下命令以验证对等互连的状态

```sh
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table
  ```

### 任务 4：创建从 East 到 West 的对等互连网络

1. 在变量中捕获 WestVNet ID

```sh
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId
```

2. 键入以下命令以将 EastVNet 与 WestVNet 对等互连

```sh
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access
```

3. 键入以下命令以验证对等互连的状态

```sh
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
  ```

> **结果**：在本实验中，你配置了 East 和 West 虚拟网络和子网，并在网络之间建立了对等互连（从 East 到 West 和从 West 到 East）。这些资源将在随后的实验室中进一步使用。
