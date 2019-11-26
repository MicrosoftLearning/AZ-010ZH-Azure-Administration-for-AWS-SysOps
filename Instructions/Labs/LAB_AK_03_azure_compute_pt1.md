# 实验 03 pt 1 练习解答

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
# AZ-010 LAB3 解决方案
# ---------------
# 依赖关系:
#   LAB1-解决方案已到位（已创建 WestRG 和 EastRG）
#   LAB2-解决方案已到位（已创建 VNets、SubNets 和 Peering）
# ---------------
# 警告：几个步骤需要花费 1 分钟以上
# 如果遇到错误，请耐心等待并咨询讲师

# ----------启动----------

# ----在运行之前将这些值编辑为唯一值----
adminUserName='azuser'
adminPassword='UniqueP@$$w0rd-Here'

# ----设置变量----
resourceGroupName='WestRG'
location='westus'
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'

#----主要脚本----

# ----创建可用性集----
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location

# ----创建 WestWinVM VM----
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --location $location \
  --size $vmSize \
  --availability-set $availabilitySet

# ----打开端口----
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000

#----允许 ICMPv4-In（运行 PowerShell 的 Bash CLI）----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}'

#----安装 IIS（运行 PowerShell 的 Bash CLI）----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'

#----检查 IIS 服务器是否在 VM 公共 IP 地址上运行----
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ***********************************************************
# 在浏览器的 IP 地址上方粘贴以查看 IIS 是否正在运行
# ***********************************************************

#----创建配置有 DNS 的 Debian 虚拟机----
#----创建 WestDebianVM----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--size 'Standard_D1' \
--location westus \
--name WestDebianVM \
--generate-ssh-keys

#----创建 EastAS 可用性集----
az vm availability-set create --name EastAS --resource-group EastRG

#----创建 EastDebianVM----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--size 'Standard_D1' \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys

#----配置 Debian 虚拟机的 DNS----
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand

#----配置 DNS----
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand

#----确保两个 Debian VM 都在运行----
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table

#----使用 SSH 连接到 WestDebianVM 并对 WestWinVM 进行 ping 测试----
#----连接到 WestRG 中的 Debian 虚拟机----

#----获取 VM 的公共和私有 IP 地址----
WestDebianIP=$(az vm list-ip-addresses -n WestDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

EastDebianIP=$(az vm list-ip-addresses -n EastDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

WestWinIP=$(az vm list-ip-addresses -n WestWinVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

#----SSH into the WestDebianVM virtual machine----

# *******************************************
#要启动 SSH 会话，请使用下面的命令
echo "For SSH to WestDebianVM use: " 'ssh' $adminUserName'@'$WestDebianIP
echo "EastDebianVM IP: " $EastDebianIP
echo "WestWinVM IP: " $WestWinIP
# *******************************************

# *******************************************
#--------启动 SSH 会话命令--------
#--请参阅实验室 03 练习 1 任务4：Ping VMs--
#--注意上面的 SSH 连接和 IP 值--
# *******************************************
```

**完成练习1 任务 4：SSH 和 Ping**

**使用 LAB_AK_03_azure_compute_pt2.md 继续实验室 03 练习解答解决方案第 2 部分**
