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

* **预计用时**：45 分钟

## 说明

### 开始前

#### 设置任务

1. **对先前实验室的依赖性：**
    1. 模块 1：Azure 管理 - **实验室：创建资源组**。已配置 EastRG 和 WestRG 资源组。
    1. 模块 2：Azure 网络 - **实验室：虚拟网络和对等互连**。已配置具有子网和对等互连的 VNet。
    1. 模块 3：Azure 计算 - **实验室：Azure VM**。已创建 EastDebianVM。

### 练习 1：Blob、安全访问存储、服务终结点和文件存储

本练习的主要任务如下：

1. 创建和配置 Blob 存储
1. 使用安全访问存储 (SAS) 配置 Blob
1. 配置子网服务终结点
1. 创建和配置文件存储

#### 任务 1：使用 Blob 存储创建存储帐户

**创建存储帐户变量**

1. 创建随机字符串以用于唯一名称

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

2. 设置变量


```sh
# set variables
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # default
my_storage_kind=StorageV2myRand
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

1. 创建存储帐户和存储帐户连接字符串环境变量

```sh
# set environment variables
export AZURE_STORAGE_ACCOUNT=$my_storage_account

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. 创建存储帐户密钥环境变量

```sh
# Display Keys
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
# Make a file to add to the container as a blob
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

#### 下载公共文件

1. 以新名称 `helloAdminDownload.html` 下载 `helloAdmin.html`。

```sh
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html
    #--account-name $AZURE_STORAGE_ACCOUNT \
```

#### 运行 `ls` 以确认文件 `helloAdminDownload.html` 已下载

```bash
ls
```

**upload-batch**

1. 创建一个 `upload` 目录

```sh
# Create files in uploadfiles directory
mkdir uploadfiles
```

2. 创建要上传到 `upload` 目录中的文件

```sh
# Create Files...
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
# Upload html files in path
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

1. 运行命令，然后单击生成的链接。

```sh
az storage blob url -c $container_name -n helloAdmin -o tsv
```

**删除公共 Blob 容器**

1. 完成检查后，清理 Blob 容器

```bash
az storage container delete -n $container_name
```

#### 任务 2：安全访问存储

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

```bash
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
# Create a 30 minute read-only SAS token on the container and store as CONTAINER_SAS_KEY
end_date=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`
```

2. 生成容器和 Blob 密钥变量

```sh
CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"
```

3. 生成完整的 Blob URI 和测试链接

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




