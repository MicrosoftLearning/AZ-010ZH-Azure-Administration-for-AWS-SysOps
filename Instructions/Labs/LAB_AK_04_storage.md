# 实验室 04 练习解答

## 说明

1. 将以下 CLI 脚本复制到记事本等编辑器中
   1. 标题为 `# ----在运行之前编辑这些值----` 的位置部分
   1. **编辑值**，否则你会收到**错误**
   1. 本地保存文件
1. 登录到 azure 门户，打开 bash Cloud Shell
1. 检查依赖项是否到位（请参阅脚本顶部的注释）
1. 从本地文件复制 CLI 脚本并将脚本粘贴到 Bash Cloud Shell 中
1. 有些任务可能需要 1 分钟以上的时间，请等待脚本完成并检查输出
1. 如果发生错误，请与讲师联系

> 关于`EDIT THESE VALUES`部分的注意事项
>
> 创建要在 `myUploadPath` 和 `myDownloadPath` 中使用的 `az_user_name` 变量
>
> 需要编辑后面的命令来设置 `az_user_name`
>
> 从 Cloud Shell 提示中的 `<name>@azure` 捕获**名称**
> 例如 - 如果 Cloud Shell 中的提示是 `eric@Azure:~$`，则设置 **`az_user_name='eric'`**

```sh
## AZ-010 LAB4 解决方案
# ---------------
# 依赖关系：
#   LAB1-解决方案已到位（已创建 WestRG 和 EastRG）
#   LAB2-解决方案已到位（已创建 VNets、SubNets 和 Peering）
#   LAB3-解决方案已到位（已创建 EastDebianVM）
# ---------------
# 警告：几个步骤需要花费 1 分钟以上
# 如果遇到错误，请耐心等待并咨询讲师

# ----------启动----------

# ----在运行之前编辑这些值----
subscriptionID=<**用于实验室的订阅 ID**>

az_user_name=<name>

# ----主要脚本----
# ----设置默认订阅----
az account set --subscription $subscriptionID

# ----创建随即字符串以供唯一名称使用----

myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand

# ----设置变量----
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # default
my_storage_kind=StorageV2
my_access_tier=hot
my_storage_encryption=blob

# **************************************************************
echo "Your my_storage_account name will be: " $my_storage_account
# **************************************************************

# ----创建存储帐户----
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption

# ----在 Bash CLI 中设置环境变量----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"

# ----创建存储帐户密钥环境变量----
# Display Keys
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table
# ----export AZURE_STORAGE_KEY=<storage_account_key1>----
# ----将 key1 存储为环境变量----
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"

# ----制作文件 `helloAdmin.html` 以上传到 Blob 存储
echo "<h1>Hello Azure Administrators</h1>">helloAdmin.html

# ----使用公共访问权限创建 Blob 容器----
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob

# ----将文件上传到公共 Blob 容器----
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name

# ----下载公共文件----
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html

# ---------upload-batch---------
mkdir uploadfiles

# ----创建要上传到 `upload` 目录中的文件----
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt

# ***********************************************************
# ASSUMES REPLACED <name> in edited variables section above
# az_user_name=<name>
# ***********************************************************

# path to $home in cloud shell
myUploadPath=/home/$az_user_name/uploadfiles

# Upload html files in path
az storage blob upload-batch -d $container_name \
-s $myUploadPath -o table

#----创建上传路径和文件夹----
mkdir downloadfiles

#----创建下载路径----
myDownloadPath=/home/$az_user_name/downloadfiles

#----从容器中批量下载文件----
az storage blob download-batch -d $myDownloadPath -s $container_name

#----获取 Blob 的 URL 并查看公共文件----
＃----单击生成的链接以查看文件
az storage blob url -c $container_name -n helloAdmin -o tsv

# ----任务 2：安全访问存储----
# ----在 west 存储帐户中创建专用 Blob 容器----

# ----刷新变量----
container_name=westblobcontainerprivate

# ----存储密钥----
export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

# ----存储连接密钥----
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"

# ----创建一个新的专用 Blob 容器
az storage container create --name $container_name --public-access blob
# ----生成将文件上传到专用 Blob 容器----
echo "<h1>Hello Azure Administrators - This is private</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name

# ----在服务级别创建共享访问签名 (SAS)----
end_date=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`

CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"


# ----生成完整的 Blob URI 和测试链接----
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
# ***********************************************************************
# ----上面生成的测试链接----
# ***********************************************************************


# ---------任务 3：创建一个子网服务终结点---------
# ----将默认规则设置为默认情况下拒绝网络访问（如果需要）----
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny

# ----更新子网规则以在 WestVNet - WestSubnet1 上启用存储。----
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"

# ----创建 `subnet_id` 变量
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"

# ----添加网络子网规则。
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id

# ----列出存储帐户网络规则更新----
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
# **********************************************************
# ----测试上一个任务中可访问的页面
# ----网络规则应拒绝访问****
echo $private_URI
# 测试上面的链接
# *********************************************************

# ----任务 4 – 创建文件存储----
# ----使用 Azure CLI 创建存储帐户----

# ----创建变量----
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand

#----创建存储帐户----
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2

#----allow non-https traffic (avoids "error 13" when monting linux drive)----
az storage account update -n $my_storage_account --https-only false

#---------创建文件共享并上传文件---------
#----设置环境变量----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
#----创建 `eastfiles` 文件共享----
file_share_name=eastfiles
#----创建存储文件共享----
az storage share create --name $file_share_name --quota 2048
＃----创建 `myFileShareFile.html` 文件进行上传
echo "File Shares Share Files">myFileShareFile.html
#----上传文件以共享----
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# ******************************************************************
# ---------------------------手动步骤---------------------------
#----从 Linux 虚拟机装载文件共享----
#----通过 SSH 连接到 Linux VM----
＃----将这些命令输入到 Clould Shell Bash 中以启动 SSH 会话----
#> east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"
#> ssh azuser@$east_vm_ip
#
＃----将这些命令键入 SSH 会话（或参见实验 4 的任务 4）----
＃----以建立目录并附加到 Azure 存储----
#ssh> mkdir -p $ my_storage_account / eastfiles
#ssh> sudo apt-get update
#ssh> sudo apt-get install cifs-utils
# ==================================================================
#----从门户获取代码以将存储连接到 Linux 计算机#---
#
#    1. 在门户中：
#        * 打开你的 eaststorage*123af4*（类似名称）存储帐户。
#        * 导航到 eastfiles 文件共享。
#    1. 在“eastfiles”边栏选项卡中，单击“连接”。
#    1. 切换到 **Linux** 选项卡并复制连接字符串。
#    1. 返回你的 EastDebianVM ssh 会话。
#    1. 将连接字符串粘贴到 ssh 会话中
#
# ==================================================================
# ====打开“新 Azure 门户网页”选项卡，启动 Cloud Shell====
#----现在已安装共享 (CLI)，停止允许非 https 流量
＃----在“Azure CLI”（不是 SSH 会话）中键入命令****
#
#> az storage account update -n $my_storage_account --https-only true
#
# ==================================================================
# ================返回 **Linux SSH 会话=================
＃----在 Linux SSH 会话中，继续使用以下命令----
#
#ssh> cd /mnt/$my_storage_account
#ssh> echo "I am from eastDebianVM">newFile.txt
＃----使用“ls”命令在 Linux 计算机上的共享中查看文件
＃----要退出 SSH 会话，请键入“exit” ----
＃----确认 Azure 门户中存在文件 (newFile.txt)

# =================================================================
# -----------准备就绪时清理存储实验-----------
＃----------返回 CLI（退出 ssh 会话）---------
＃---使用以下命令删除文件存储帐户----
#> az storage account delete -n eaststore$myRand -g EastRG
#> az storage account delete -n weststore$myRand -g WestRG
#

> 查看上面的命令脚本以查看
> * 验证步骤
> * SSH 的手动步骤
> * 手动清理步骤

