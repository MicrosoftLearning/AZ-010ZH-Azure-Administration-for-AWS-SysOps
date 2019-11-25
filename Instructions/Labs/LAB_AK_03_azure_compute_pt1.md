# 实验 03 pt 1 练习解答

## 说明

1. 将以下 CLI 脚本复制到记事本等编辑器中
1. 标题为 `# ----EDIT THESE VALUES Before Running----` 的位置部分
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
# WARNING: Several Steps take more than 1 minute
# Be patient and consult instructor if errors are encountered

# ----------START----------

# ----在运行之前将这些值编辑为唯一值----
adminUserName='azuser'
adminPassword='UniqueP@$$w0rd-Here'

# ----设置变量----
resourceGroupName='WestRG'
location='westus'
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'

# ----Main Scripts----

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

# ----Allow ICMPv4-In (Bash CLI running PowerShell)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}'

# ----Install IIS (Bash CLI running PowerShell)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'

# ----Check the IIS Server is running on VM public IP address----
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ***********************************************************
# Paste above IP address in browser to see if IIS is running
# ***********************************************************

# ----Create Debian virtual machines configured with DNS----
# ----Create WestDebianVM----
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

# ----Create EastAS Availability Set----
az vm availability-set create --name EastAS --resource-group EastRG

# ----Create EastDebianVM----
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

# ----Configure DNS of Debian Machines----
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand

# ----Configure DNS----
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand

# ----Ensure both Debian VMs are running----
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table

# ----Connect to WestDebianVM with SSH and ping test WestWinVM----
# ----Connect to the Debian virtual machine in WestRG----

# ----get Public and Private IP Addresses of the VMs----
WestDebianIP=$(az vm list-ip-addresses -n WestDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

EastDebianIP=$(az vm list-ip-addresses -n EastDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

WestWinIP=$(az vm list-ip-addresses -n WestWinVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

#----SSH into the WestDebianVM virtual machine----

# *******************************************
# To Start SSH Session use the command below
echo "For SSH to WestDebianVM use: " 'ssh' $adminUserName'@'$WestDebianIP
echo "EastDebianVM IP: " $EastDebianIP
echo "WestWinVM IP: " $WestWinIP
# *******************************************

# *******************************************
# --------Start SSH Session Commands--------
# --See Lab 03 Exercise 1 Task 4: Ping VMs--
# --Note above SSH connection and IP values--
# *******************************************
```

**完成练习1 任务 4：SSH 和 Ping**

**使用 LAB_AK_03_azure_compute_pt2.md 继续实验室 03 练习解答解决方案第 2 部分**
