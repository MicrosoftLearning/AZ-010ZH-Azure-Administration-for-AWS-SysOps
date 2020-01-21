---
lab:
    title: '创建用户、组和策略。监视日志和警报。'
    module: '模块 5：Azure 身份识别'
---

# 实验室 01：Azure 身份识别

创建用户、组和策略以及监视日志和警报

## 学生实验室手册

## 方案

在本实验室中，你将在 Cloud Shell 中使用带有 Bash 接口的 CLI 命令来管理 Azure 标识。  你将使用 Azure 基于角色的访问控制、Azure Policy，并使用 Query Explorer 查看监控。

## 目标

完成本实验室后，你将能够：

* 使用 Azure 基于角色的访问控制创建和配置用户和组
* 创建限制软件安装的策略
* 使用 Query Explorer 在 Azure 门户中查看监控日志和警报

## 实验室设置

* **预计用时**：45 分钟

## 说明

### 开始前

#### 设置任务

1. 配置将用于本课程的 Azure 帐户。
2. **模块 1: Azure 管理，实验室： 创建资源组** 已配置 WestRG 资源组。

### 练习 1：创建用户、组和策略

本练习的主要任务如下：

1. 添加用户和组
1. 创建限制软件安装的策略

#### 练习 1 - 任务 1：打开 Cloud Shell

**사용자 추가**

1. 사용자 도메인용 변수를 만듭니다.

> 이메일 주소 eric@contoso.com을 사용하면 명령이 `my_domain=ericcontoso.onmicrosoft.com`이 됩니다.
>
> * *필요한 경우 강사에게 문의하세요.*

2. 사용자 도메인용 변수를 편집합니다.

```sh
# 사용자 도메인용 변수 만들기
# user@contoso.com =>  usercontoso.onmicrosoft.com

my_domain=<email+service>.onmicrosoft.com
```

**사용자 계정 만들기**

1. 사용자 계정 이름을 만듭니다.

```sh
my_user_account=AZ010@$my_domain
```

2. 강력한 고유 암호를 만듭니다(암호 편집).

```sh
# 암호를 고유한 암호로 편집합니다. 이때 선행 "!"를 제거하지 않으면 오류가 발생합니다.
az ad user create \
    --display-name AZ010Tester \
    --password !sTR0ngP@ssWorD543%* \
    --user-principal-name $my_user_account
```

3. 표시 이름, 암호 및 사용자 주체 이름을 기록해 둡니다.

**사용자 및 그룹 관리**

1. AD 사용자 목록을 표시합니다.

```sh
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
```

2. 이전 단계에서 만든 사용자가 나열되어야 합니다.
3. 모든 역할 할당 목록을 표시합니다.

```sh
az role assignment list --all -o table
```

4. 이 목록의 초기 상태를 기록해 둡니다.
5. 리소스 그룹용 역할 할당 목록을 표시합니다.

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

6. 새 사용자에게 역할("소유자")을 추가합니다.

```sh
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#az role assignment create --role "Owner" --assignee <할당 대상 개체 ID> --resource-group <리소스 그룹>
```

**사용자 및 그룹의 변경 내용 검토**

1. 리소스 그룹용 역할 할당 목록을 다시 표시합니다(변경 사항 확인).

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

2. 만든 사용자에 대한 역할 할당 목록을 표시합니다(변경 사항 확인).

```sh
az role assignment list --assignee $my_user_account -g WestRG #--output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

## 작업 2 – 소프트웨어 설치를 제한하는 정책 만들기

**소프트웨어 제한 정책 배포**

> *[Github](https://github.com/Azure/azure-policy/tree/master/samples/built-in-policy/require-sqlserver-version12)에서 아래 스크립트에 사용되는 rules.json 및 parameters.json을 검토합니다.*

1. 정책 정의를 만듭니다.

```sh
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.' \
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All
```

2. 구독 수준에서 범위를 지정합니다.

```sh
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'
```

3. 정책 할당 목록을 표시합니다.

```sh
az policy assignment list
```

4. 새로 만든 정책을 표시합니다.

```sh
az policy assignment show --name 'SQL12AZ010'
```

**규정 준수 여부 확인**

1. Azure Policy 서비스 페이지로 돌아갑니다.
2. 규정 준수를 선택합니다. 규정 준수 상태 필터를 "모든 준수 상태"로 설정합니다.
3. 정책 및 정의 상태를 검토합니다.
## 연습 2: 쿼리 탐색기를 통해 로그 및 경고 모니터링

1. 로그 및 경고 모니터링

### 연습 2 - 작업 1 – 모니터링 로그 및 경고 검토

**데모 환경 액세스**

1. 새 브라우저 탭에서 [로그 분석 쿼리 데모](https://portal.loganalytics.io/demo)로 이동합니다.
2. 쿼리 탐색기 사용
    1. 오른쪽 위의 쿼리 탐색기를 선택합니다.
    2. 즐겨찾기를 확장한 다음 *오류가 있는 모든 Syslog 레코드*를 선택합니다.
    3. 쿼리가 편집 창에 추가됩니다. 쿼리의 구조를 확인합니다.
    4. 쿼리를 실행합니다. 반환된 레코드를 살펴봅니다.
    5. 강사의 추가 단계를 따릅니다.
    6. 시간이 되면 다른 즐겨찾기와 저장된 쿼리도 사용해 봅니다.
