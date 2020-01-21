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

5. 이전 단계에서 생성된 URI 링크를 테스트하여 작동하는지 확인

### 작업 3: 서브넷 서비스 엔드포인트 만들기

**스토리지 계정의 기본 규칙 상태 표시**

1. 다음 CLI 명령을 입력하여 스토리지 계정에 대한 기본 규칙의 상태를 표시합니다.

```sh
az storage account show --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --query networkRuleSet.defaultAction
```

**필요한 경우 네트워크 액세스를 기본적으로 거부하도록 기본 규칙 설정**

1. 규칙이 현재 'Allow'로 설정된 경우 'Deny'로 설정

```sh
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny
```

**스토리지 계정 네트워크 규칙 목록 표시**

1. 네트워크 규칙 목록을 표시한 다음 검토합니다.

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**기존 VNet(WestVNet) 및 서브넷(WestSubnet1)에서 Azure Storage용 서비스 엔드포인트를 사용하도록 설정**

1. 서브넷 규칙을 업데이트하여 WestVNet - WestSubnet1에서 스토리지를 사용하도록 설정합니다.

```sh
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"
```

**가상 네트워크 및 서브넷용 네트워크 규칙 추가**

> **참고**: 네트워크 액세스를 거부하도록 기본 규칙을 설정해야 합니다. 이렇게 하지 않으면 네트워크 규칙이 적용되지 않습니다.

1. 'subnet_id' 변수 만들기

```sh
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"
```

2. 네트워크 서브넷 규칙을 추가합니다.

```sh
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id
```

**스토리지 계정 네트워크 규칙 업데이트 목록 표시**

1. 네트워크 규칙 목록을 표시하고 다시 검토합니다.

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**이전 태스크에서 액세스할 수 있었던 페이지 테스트(네트워크 규칙이 액세스를 거부해야 함)**

1. 테스트할 링크를 클릭합니다.
2. 결과는 "access denied"가 되어야 합니다.

```sh
echo $private_URI
```

**스토리지 계정(west) 정리**

1. 검토가 완료되면 스토리지 계정 리소스 제거

```sh
az storage account delete -n $my_storage_account -g my_resource_group
```

### 태스크 4 – File Storage 만들기

**Azure CLI를 사용하여 eastus 스토리지 계정 만들기**

1. 고유한 이름에 사용할 임의 문자열 생성(여전히 메모리에 있지 않은 경우)

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "추가되는 임의 문자열:  "$myRand
```

2. 변수 만들기

```sh
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand
```

3. eastus 스토리지 계정 만들기

```sh
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2
```

4. https가 아닌 트래픽 허용(그러면 Linux 드라이브 탑재 시 "오류 13"이 발생하지 않음)

```sh
az storage account update -n $my_storage_account --https-only false
```

5. 스토리지 계정 목록 표시
6. 새로 만든 계정 확인

```sh
az storage account list -o table
```

**파일 공유 만들기 및 파일 업로드**

1. 환경 변수 설정

```sh
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. 'eastfiles' 파일 공유 만들기

```sh
file_share_name=eastfiles

az storage share create --name $file_share_name --quota 2048
```

3. 업로드할 'myFileShareFile.html' 파일 만들기

```sh
echo "File Shares Share Files">myFileShareFile.html
```

4. 공유할 파일 업로드

```sh
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html
```

5. Azure File Storage에 있는 파일의 출력 목록
6. 이제 'myFileShareFile.html'이 File Storage에 있음

```sh
az storage file list --share-name $file_share_name -o table
```

**Linux 가상 머신에서 파일 공유 탑재**

1. Linux VM으로의 SSH 실행

```sh
east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"

ssh azuser@$east_vm_ip
```

> 참고: VM SSH 세션에서 스토리지 연결 준비를 위한 명령 실행

2. Linux VM에 eastfiles용 디렉터리 만들기

```sh
mkdir -p $my_storage_account/eastfiles
```

3. Linux VM에 cifs-utils 설치

```sh
sudo apt-get update

sudo apt-get install cifs-utils
```

4. 포털에서 코드를 가져와 Linux 컴퓨터에 스토리지 연결

    1. Portal에서 다음 단계를 수행합니다.
        * eaststorage*123af4* 또는 비슷한 이름의 스토리지 계정을 엽니다.
        * eastfiles 파일 공유로 이동합니다.
    1. "eastfiles" 블레이드에서 연결을 클릭합니다.
    1. **Linux** 탭으로 변경해 연결 문자열을 복사합니다.
    1. EastDebianVM ssh 세션으로 돌아옵니다.
    1. ssh 세션에서 연결 문자열을 붙여 넣습니다.

> ```sh
> # 포털의 연결 문자열은 다음과 유사합니다.
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

**파일이 Azure와 VM 간에 양방향으로 공유되는지 확인**

1. 공유가 탑재되었으므로 https가 아닌 트래픽 허용 중지(CLI)
    * **새 Azure Portal 웹 페이지** 인스턴스를 열고 Cloud Shell 시작
    * SSH 세션이 아닌 **Azure CLI**에 명령 입력

```sh
az storage account update -n $my_storage_account --https-only true
```

> 참고: 다음 명령을 실행하기 위해 **Linux SSH 세션**으로 돌아갑니다.

2. Linux SSH 세션에서 탑재 디렉터리로 이동합니다.

```sh
# eastDebianVM SSH에서
cd /mnt/$my_storage_account
```

3. Azure File Storage에서 VM으로 파일이 공유되는지 확인합니다.

```sh
# 공유된 파일을 확인합니다.
ls
```

4. VM에서 새 파일을 추가합니다.

```sh
# VM에서 새 파일을 만듭니다.
echo "eastDebianVM의 파일">newFile.txt
```

5. `exit` SSH session

```sh
# VM ssh 세션에서
exit
```

6. Azure Portal에서 파일(newFile.txt)을 표시하여 공유가 양방향으로 모두 작동하는지 확인합니다.

**File Storage 리소스 정리**

1. 스토리지 계정을 제거합니다.

```sh
az storage account delete -n $my_storage_account -g my_resource_group
```
