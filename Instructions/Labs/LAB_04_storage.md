---
lab:
    title: 'Blob、安全访问存储、服务终结点和文件存储'
    module: '模块 4：Azure 存储'
---

# 实验室 04：Azure 存储

存储帐户、Blob、安全访问存储、服务终结点和文件存储

## 学生实验室手册

## 方案

在本实验室中，你将在 Cloud Shell 中使用带有 Bash 接口的 CLI 命令来管理 Azure 存储。你将配置存储帐户、Azure Blob 存储、安全访问存储 (SAS)、子网服务终结点和文件存储。

## 目标

完成此实验室之后，你将可以使用 Azure CLI 创建和配置：

* 存储帐户
* Blob 存储
* 安全访问存储 (SAS)
* 子网服务终结点
* 文件存储

## 实验室设置

* **预计用时**：60 分钟

## 说明

### 开始前

#### 设置任务

1. **对先前实验室的依赖性：**
    1. 模块 1：Azure 管理 - **实验室：创建资源组**。已配置 EastRG 和 WestRG 资源组。
    1. 模块 2：Azure 网络 - **实验室：虚拟网络和对等互连**。已配置具有子网和对等互连的 VNet。
    1. 模块 3：Azure 计算 - **实验室：Azure VM**。已创建 EastDebianVM。

## 练习 1：Blob、安全访问存储、服务终结点和文件存储

本练习的主要任务如下：

1. 创建和配置 Blob 存储
1. 使用安全访问存储 (SAS) 配置 Blob
1. 配置子网服务终结点
1. 创建和配置文件存储

### 练习 1 - 任务 1：使用 Blob 存储创建存储帐户

**创建存储帐户变量**

1. 创建随机字符串以用于唯一名称

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

2. 设置变量


```sh
# 设置变量
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # default
my_storage_kind=StorageV2
my_access_tier=hot
my_storage_encryption=blob

echo "Your my_storage_account name will be: " $my_storage_account
```

>注意上述命令中的存储帐户的名称

**创建存储帐户**

```sh
# create account
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption
```

**在 Bash CLI 中设置环境变量**

1. 创建环境变量
   * 存储帐户
   * 存储帐户连接字符串

```sh
# set environment variables
export AZURE_STORAGE_ACCOUNT=$my_storage_account

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. 创建存储帐户密钥环境变量

```sh
# 显示密钥
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table

# export AZURE_STORAGE_KEY=<storage_account_key1>
# store key1 as environment variable
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
```

> 提示：在文本文件中捕获所有变量和环境变量命令，这样，如果 Cloud Shell 在 20 分钟后超时并且变量从临时内存中丢失，你也可以轻松地重新输入。

**准备要添加到 Blob 存储的文件**

1. 制作文件 `helloAdmin.html` 以上传到 Blob 存储

```sh
#制作一个文件作为 Blob 添加到容器中
echo "<h1>Hello Azure Administrators</h1>">helloAdmin.html
```

2. check that the file `helloAdmin.html` is created

```sh
ls
```

**在 west 存储帐户中创建公共 Blob 容器**

1. 使用公共访问权限创建 Blob 容器

```sh
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob
# --connection-string $AZURE_STORAGE_CONNECTION_STRING
```

**将文件上传到公共 Blob 容器**

```sh
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name
    #--connection-string $AZURE_STORAGE_CONNECTION_STRING
```

**下载公共文件**

1. 下载你先前上传的文件
2. 确保以新名称 `helloAdmin.html` 下载 `helloAdminDownload.html`

```sh
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html
    #--account-name $AZURE_STORAGE_ACCOUNT \
```

3. 运行 `ls` 以确认文件 `helloAdminDownload.html` 已下载

```sh
ls
```

**upload-batch**

1. 创建一个 `upload` 目录

```sh
#在 uploadfiles 目录中创建文件
mkdir uploadfiles
```

2. 创建要上传到 `upload` 目录中的文件

```sh
#创建文件...
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt
```

3. 创建要在 `myUploadPath` 和 `myDownloadPath` 中使用的 `az_user_name` 变量

> 注意：需要编辑以下命令来设置 `az_user_name`
>
> 捕获 Cloud Shell 提示中的名称 `<name>@azure`
> 例如 - 如果 Cloud Shell 中的提示是 `eric@Azure:~$`，则设置 `az_user_name='eric'`

```sh
# REPLACE <name>
az_user_name=<name>
```

4. 创建上传路径和文件夹

```sh
# path to $home in cloud shell
myUploadPath=/home/$az_user_name/uploadfiles
```

5. 批量上传文件到容器

```sh
#将 HTML 文件上传到以下路径
az storage blob upload-batch -d $container_name \
-s $myUploadPath -o table
```

6. 列出 Blob 中的文件并检查上传的文件

```sh
az storage blob list --container-name $container_name -o table
```

**download-batch**

1. 创建一个 `download` 目录

> 注意：请确保已按照上述说明设置了 `az_user_name` 变量

2. 创建上传路径和文件夹

```sh
mkdir downloadfiles

myDownloadPath=/home/$az_user_name/downloadfiles
```

3. 从容器批量下载文件

```sh
az storage blob download-batch -d $myDownloadPath -s $container_name
```

4. 列出下载目录中的文件

```sh
# list downloadfiles
ls downloadfiles
```

**获取 Blob 的 URL 并查看公共文件**

1. 运行命令
2. 单击生成的链接以查看文件

```sh
az storage blob url -c $container_name -n helloAdmin -o tsv
```

**删除公共 Blob 容器**

1. 完成检查后，清理 Blob 容器

```sh
az storage container delete -n $container_name
```

### 任务 2：安全访问存储

**在 west 存储帐户中创建专用 Blob 容器**

1. 根据需要刷新变量
1. **编辑** `AZURE_STORAGE_ACCOUNT`以反映生成的帐户名称

> 注意：`container_name` 的新值是 `westblobcontainerprivate`

```sh
my_resource_group=WestRG
location=westus
container_name=westblobcontainerprivate

export AZURE_STORAGE_ACCOUNT=<*Enter*Storage*Account*> # $my_storage_account

export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"
```

**使用专用访问权限创建 Blob 容器**

1. 创建一个新的专用 Blob 容器

```sh
az storage container create --name $container_name --public-access blob
```

**将文件上传到专用 Blob 容器**

```sh
echo "<h1>Hello Azure Administrators - This is private</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name
```

**在服务级别创建共享访问签名 (SAS)**

1. 设置 `end_date`

> *注意：访问权限配置为仅持续 30 分钟*

```sh
#在容器上创建一个 30 分钟的只读 SAS 令牌，并将其存储为 CONTAINER_SAS_KEY
end_date=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`
```

2. 生成容器密钥变量

```sh
CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"
```

3. 生成 Blob 密钥变量

```sh
BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"
```

4. 生成完整的 Blob URI 和测试链接

```sh
# generate full blob URI and test link
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
```

5. 测试上一步中生成的 URI 链接，以确保其正常工作

### 任务 3：创建一个子网服务终结点

**显示存储帐户默认规则的状态**

1. 键入以下 CLI 命令，显示存储帐户默认规则的状态。

```sh
az storage account show --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --query networkRuleSet.defaultAction
```

**将默认规则设置为默认情况下拒绝网络访问（如果需要）**

1. 如果当前设置为 `Allow`，请将规则设置为 `Deny`

```sh
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny
```

**列出存储帐户网络规则**

1. 显示，然后查看网络规则列表。

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**在现有 vnet (WestVNet) 和子网 (WestSubnet1) 上启用 Azure 存储的服务终结点**

1. 更新子网规则以在 WestVNet - WestSubnet1 上启用存储。

```sh
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"
```

**为虚拟网络和子网添加网络规则**

> **注意**：必须将默认规则设置为“拒绝”，否则网络规则无效。

1. 创建 `subnet_id` 变量

```sh
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"
```

2. 添加网络子网规则。

```sh
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id
```

**列出存储帐户网络规则更新**

1. 显示并再次查看网络规则列表。

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**测试上一个任务中可访问的页面 – 网络规则应拒绝访问**

1. 单击链接进行测试。
2. 结果应为“拒绝访问”。

```sh
echo $private_URI
```

**清理存储帐户 (west)**

1. 完成检查后，删除存储帐户资源

```sh
az storage account delete -n $my_storage_account -g my_resource_group
```

### 任务 4 – 创建文件存储

**使用 Azure CLI 创建 eastus 存储帐户**

1. 创建随机字符串供唯一名称使用（如果内存中还没有）

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

2. 创建变量

```sh
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand
```

3. 创建 eastus 存储帐户

```sh
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2
```

4. allow non-https traffic (avoids "error 13"  mounting Linux drive)

```sh
az storage account update -n $my_storage_account --https-only false
```

5. 列出存储帐户
6. 注意新创建的帐户

```sh
az storage account list -o table
```

**创建文件共享并上传文件**

1. 设置环境变量

```sh
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. 创建 `eastfiles` 文件共享

```sh
file_share_name=eastfiles

az storage share create --name $file_share_name --quota 2048
```

3. 创建要上传的 `myFileShareFile.html` 文件

```sh
echo "File Shares Share Files">myFileShareFile.html
```

4. 上传文件以共享

```sh
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html
```

5. Azure 文件存储中文件的输出列表
6. 注意 `myFileShareFile.html` 现在位于文件存储中

```sh
az storage file list --share-name $file_share_name -o table
```

**从 Linux 虚拟机装载文件共享**

1. 通过 SSH 连接到 Linux VM

```sh
east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"

ssh azuser@$east_vm_ip
```

> 注意：在 VM SSH 会话中运行命令以准备进行存储连接

2. 在 Linux VM 上为 eastfiles 创建目录

```sh
mkdir -p $my_storage_account/eastfiles
```

3. 在 Linux VM 上安装 cifs-utils

```sh
sudo apt-get update

sudo apt-get install cifs-utils
```

4. 从门户获取代码以将存储连接到 Linux 计算机

    1. 在门户中：
        * 打开你的 eaststorage*123af4*（类似名称）存储帐户。
        * 导航到 eastfiles 文件共享。
    1. 在“eastfiles”边栏选项卡中，单击“连接”。
    1. 切换到 **Linux** 选项卡并复制连接字符串。
    1. 返回你的 EastDebianVM ssh 会话。
    1. 将连接字符串粘贴到 ssh 会话中

> ```sh
> # 来自门户的连接字符串将类似于以下内容
> sudo mkdir /mnt/eaststorage123af4
> if [ ! -d "/etc/smbcredentials" ]; then
> sudo mkdir /etc/smbcredentials
> fi
> if [ ! -f "/etc/smbcredentials/eaststorage123af4.cred" ]; then
>     sudo bash -c 'echo "username=eaststorage123af4" >> /etc/smbcredentials/eaststorage123af4.cred'
>     sudo bash -c 'echo "password=EEBKMxGPWyTHe5CKEU58d1PCEdU2gOMHWb4nRZM07RlT5dpk6yiY0nFHCeN4HQOqNr8HLuKhQqa05m4qrAXJrg==" >> /etc/smbcredentials/eaststorage123af4.cred'
> fi
> sudo chmod 600 /etc/smbcredentials/eaststorage123af4.cred
> 
> sudo bash -c 'echo "//eaststorage123af4.file.core.windows.net/eastfiles /mnt/eaststorage123af4 cifs nofail,vers=3.0,credentials=/etc/smbcredentials/eaststorage123af4.cred,dir_mode=0777,file_mode=0777,serverino" >> /etc/fstab'
> 
> sudo mount -t cifs //eaststorage123af4.file.core.windows.net/eastfiles /mnt/eaststorage123af4 -o vers=3.0,credentials=/etc/smbcredentials/eaststorage123af4.cred,dir_mode=0777,file_mode=0777,serverino
> ```

**验证文件是否已从 Azure 共享到 VM 或已从后者连接到前者**

1. 现已安装共享 (CLI)，停止允许非 https 流量
    * 打开**新 Azure 门户网页**实例并启动 Cloud Shell
    * 在“Azure CLI”（不是 SSH 会话）中输入命令****

```sh
az storage account update -n $my_storage_account --https-only true
```

> 注意：返回到**Linux SSH 会话**，以便执行以下命令

2. 在 Linux SSH 会话中，导航到装载目录

```sh
# 来自 eastDebianVM SSH
cd /mnt/$my_storage_account
```

3. 查看从 Azure 文件存储共享到 VM 的文件

```sh
# 查看共享文件
ls
```

4. 从 VM 添加新文件

```sh
# 从 VM 创建新文件
echo "I am from eastDebianVM">newFile.txt
```

5. `exit` SSH 会话

```sh
# 从 VM ssh 会话
exit
```

6. 确认 Azure 门户中存在文件 (newFile.txt)，从而确认共享在两个方向均有效

**清理文件存储资源**

1. 删除存储帐户

```sh
az storage account delete -n $my_storage_account -g my_resource_group
```
