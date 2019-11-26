---
lab:
    title: 'Azure VM 和规模集'
    module: '模块 3：Azure 计算'
---
    
# 实验室 03：Azure 计算

## 学生实验室手册

## 方案

在本实验中，你将在 Cloud Shell 中使用带有 Bash 接口的 CLI 命令来管理可用性集中的 Azure Windows 和 Debian VM。此外，该实验室还将创建一个带有可选测试的 Ubuntu 规模集，以演示放大和缩小。

## 目标

完成此实验室之后，你将可以使用 Azure CLI 执行以下操作：

* 创建规模集
* 创建和配置 Windows 和 Linux VM
* 配置 Ubuntu 规模集

## 实验室设置

* **预计用时**：60 分钟

## 说明

### 开始前

#### 设置任务

1. **对先前实验室的依赖性：**
    1. 模块 1：Azure 管理 - **实验室：创建资源组**。已配置 EastRG 和 WestRG 资源组。
    1. 模块 2：Azure 网络 - **实验室：虚拟网络和对等互连**。已配置具有子网和对等互连的 VNet。

### 练习 1：创建在可用性集内配置的 VM

本练习的主要任务如下：

1. 创建可用性集
1. 创建 Windows 虚拟机
1. 创建 Linux Debian 虚拟机

#### 任务 1：创建 Windows 虚拟机

* 创建可用性集
* 创建 Windows Server 2016 DataCenter VM
* 配置为 WebServer

**准备变量**

1. 在 **Cloud Shell** 命令提示符中，键入以下命令，然后按 **Enter** 准备在以下脚本中使用的变量。

> **注意**：创建一个唯一密码并将其写下来

```sh
resourceGroupName='WestRG'
location='westus'
adminUserName='azuser'
adminPassword='UniqueP@$$w0rd-Here' # make unique
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'
```

**创建可用性集**

1. 键入以下命令以创建 **WestAS** 可用性集。

```sh
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location
```

**创建 WestWinVM VM**

1. 键入以下命令以创建 **WestWinVM** Windows Server VM。

```sh
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --size $vmSize \
  --location $location \
  --availability-set $availabilitySet
  ```

**打开端口**

1. 键入以下命令以打开 WestWinVM 中的端口。

```sh
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000
```

**检查你的工作**

1. 启动 Azure 门户并导航到 WestRG
2. 注意资源，包括 WestWinVM
3. 启动 Azure 顾问并注意建议

#### 任务 2：将 WestWinVM 配置为 Web 服务器并允许 Ping

**允许 ICMPv4-In（运行 PowerShell 的 Bash CLI）**

1. 在 **Cloud Shell** 命令提示符下，键入以下命令，然后按 **Enter** 通过将 PowerShell 命令发送到 WestWinVM Windows Server 来允许 ICMPv4-In。

> **注意：**如果 Cloud Shell 已超时，则可能需要刷新任务 1 中的变量。

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}'
```

**安装 IIS（运行 PowerShell 的 Bash CLI）**

1. 键入以下命令以安装 IIS，以将 PowerShell 命令发送到 WestWinVM Windows Server。

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'
```

**检查 IIS 服务器是否在 VM 公共 IP 地址上运行**

1. 输入以下命令以捕获 WestWSinVM 公共 IP 地址。

```sh
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

#----在浏览器的 IP 地址上方粘贴以查看 IIS 是否正在运行----
```

2. 将生成的 IP 地址粘贴到 Web 浏览器中，以验证是否存在 IIS 默认页面。

#### 任务 3：创建配置有 DNS 的 Debian 虚拟机

在 Cloud Shell 中创建两个 Debian 虚拟服务器并通过 SSH 连接到服务器

虚拟机 1： **WestDebianVM**

> - WestRG 资源组
>   - WestVNet
>      - WestSubnet1

和

虚拟机 2： **EastDebianVM**

> - East 资源组
>   - EastVNet
>     - EastSubnet2 子网

**创建 WestDebianVM**

1. 输入以下命令以创建 WestDebianVM

```sh
az vm create \
--image credativ:Debian:8:latest \
--size 'Standard_D1' \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--location westus \
--name WestDebianVM \
--generate-ssh-keys
```

2. 注意将生成 shh 密钥（如果不存在）并保存在`~/.ssh`目录中。在 Cloud Shell 中键入以下命令以查看`~/.ssh`目录的目录内容。

```sh
ls .ssh
```

**创建 EastAS 可用性集**

1. 输入以下命令以创建 EastAS

```sh
az vm availability-set create --name EastAS --resource-group EastRG
```

**创建 EastDebianVM**

1. 键入以下命令以创建 EastDebianVM

```sh
az vm create \
--image credativ:Debian:8:latest \
--size 'Standard_D1' \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys
```

**配置 Debian 虚拟机的 DNS**

1. 键入以下命令以使用 Bash 创建随机字符串，以用于分配唯一的 DNS 名称。

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

**配置 DNS**

1. 键入以下命令，为 Linux Debian VM 分配唯一的 DNS 名称。

```sh
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand
```

2. 注意 Linux Debian VM 的 DNS 名称。

**确保两个 Debian VM 都在运行**

1. 键入以下命令以查看 Linux Debian VM 的状态（它们应该正在运行）。

```sh
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table
```

#### 任务 4：使用 SSH 连接到 WestDebianVM 并对 WestWinVM 进行 ping 测试

**连接到 West 资源组中的 Debian 虚拟机**

1. 键入以下命令以获取 VM 的公共 IP 地址和专用 IP 地址

```sh
az vm list-ip-addresses --resource-group WestRG

az vm list-ip-addresses --resource-group EastRG
```

2. 注意值 EastDebianVM、WestDebianVM 和 WestWinVM IP 地址并**记录 IP**（或在门户中每个 VM 的概述页面中查找）。

**通过 SSH 进入 WestDebianVM 虚拟机**

1. 使用 WestDebianVM IP 地址编辑以下命令以启动 SSH 会话

```sh
ssh azuser@<PUBLIC IP address of West Debian VM>
```

**从 SSH 会话对 Windows VM 专用地址进行 Ping 处理**

> *由于两个都位于同一个 **专用** VNet 上，此操作应当有效*

1. 使用 WestWinVM IP 地址编辑以下命令，然后在 Cloud Shell SSH 会话中键入以对 WestWinVM 进行 ping 处理。

```sh
ping <PRIVATE IP address of the Windows server>
```

**Ping EastDebianVM**

1. 使用 EastDebianVM IP 地址编辑以下命令，然后在 Cloud Shell SSH 会话中键入以对 WestWinVM 进行 ping 处理。

> *由于先前配置的 VNet 对等互连，此操作应该有效*

```sh
ping <PRIVATE IP address of eastdebianvm>
```

> **注意**：输入`exit`退出 SSH

### 练习 2：创建和测试 Ubuntu 规模集

本练习的主要任务如下：

1. 创建 Ubuntu 缩放集和自动缩放配置文件
1. 创建自动扩展和自动缩小规则
1. 测试 Ubuntu 规模集（可选）

#### 任务 1：创建 Ubuntu 规模集

**创建规模集资源组**

1. 键入以下命令来为你的规模集创建一个资源组

```sh
az group create --name EastScaleRG --location eastus
```

**创建规模集**

1. 键入以下命令以创建规模集

```sh
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys
```

查看门户仪表板并注意缩放规则之前的 2 个服务器

> - 资源组
>   - EastScaleRG
>     - EastUbuntuServers > 实例

**定义自动缩放配置文件**

1. 键入以下命令以定义自动缩放配置文件

```sh
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1
  ```

> *应该看到消息* “遵循`az monitor autoscale rule create`以添加缩放规则。”

**创建规则以自动横向扩展**

1. 键入以下命令以创建**自动横向扩展**规则

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1
```

**创建规则以自动缩小**

1. 键入以下命令以创建**自动缩小**规则

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1
```

> *注意：“缩小”可能会在短短 2 分钟内发生，因此以下结果在最初的几分钟内将会有所不同*

#### 任务 2：测试 Ubuntu 规模集（可选任务）

> 此任务将在服务器上生成 CPU 符合，作为演示规模集行为的测试。

**列出在规模集中运行的服务器**

* 在第一分钟，脚本应列出 2 个服务器实例（例如 0 和 1）
* 一分钟后的某个时间应仅列出 1 个服务器实例（例如 0）

1. 键入以下命令以列出服务器
1. 30 秒后**重复**此命令，直到仅列出 1 个服务器实例

```sh
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers
```

**使用 SSH 连接到实例 0**

1. 使用门户获取规模集实例 0 的 IP 地址
1. 键入以下已编辑的命令以通过 SSH 进行连接

```sh
# example: ssh azuser@13.92.224.66 -p 50000  
ssh azuser@<instance 0 IP> -p 50000
```

**运行 Stress 4 分钟（240 秒）**

1. 在 SSH 会话中键入以下命令以安装 Stress 应用程序（在 240 秒后超时）

```sh
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 240 &
```

**确认 SSH 会话中的 Stress**

1. 在 SSH 中键入以下命令以监控 Stress

```sh
top
```

**退出和 SSH**

```sh
# ctrl-c
exit
```

**注意自动扩展和自动缩小**

1. 键入以下命令以在 Autoscale 上运行监控

```sh
watch az vmss list-instances \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --output table
```

> **注意**：可能需要几分钟的时间 Stress 才能开始记录超过 50% 的负荷
>
> **注意**：按下 Ctrl+C 可关闭“watch”

**清理规模集演示**

```sh
az group delete --name EastScaleRG --yes --no-wait
```
