---
lab:
    title: '实验室：准备部署 Azure 虚拟桌面 (Azure AD DS)'
    module: '模块 1：规划 AVD 体系结构'
---

# 实验室 - 准备部署 Azure 虚拟桌面 (Azure AD DS)
# 学生实验室手册

## 实验室依赖项

- Azure 订阅
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有与 Azure 订阅关联的 Azure AD 租户的全局管理员角色，以及 Azure 订阅的所有者角色或参与者角色

## 预计用时

150 分钟

>**备注**： 预配 Azure AD DS 需要大约 90 分钟的等待时间。

## 实验室场景

你需要准备在 Azure Active Directory 域服务 (Azure AD DS) 环境中部署 Azure 虚拟桌面

## 目标
  
完成本实验室后，你将能够：

- 实现 Azure AD DS 域
- 配置 Azure AD DS 域环境

## 实验室文件

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## 说明

### 练习 0：增加 vCPU 配额数

本练习的主要任务如下：

1. 确定当前的 vCPU 使用情况
1. 请求增加 vCPU 配额

#### 任务 1：确定当前的 vCPU 使用情况

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开 **“Cloud Shell”** 窗格。
1. 提示选择 **“Bash”** 或 **“PowerShell”** 时，选择 **“PowerShell”**。 

   >**备注**： 如果这是第一次启动 **Cloud Shell**，并显示消息 **“未装载任何存储”**，请选择你将在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1. 如果未注册 **Microsoft.Compute** 资源提供程序，则在 Azure 门户中 **Cloud Shell** 的 PowerShell 会话中，运行以下命令进行注册：

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. 在 Azure 门户中 **Cloud Shell** 的 PowerShell 会话中，运行以下命令以验证 **Microsoft.Compute** 资源提供程序的注册状态：

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**备注**： 验证状态是否显示为 **“已注册”**。如果未显示，请等待几分钟并重复此步骤。

1. 在 Azure 门户中，在 **Cloud Shell** 的 PowerShell 会话中，运行以下命令以确定 vCPU 的当前使用情况以及 **StandardDSv3Family** 和 **StandardBSFamily** Azure VM 的相应限制（将 `<Azure_region>` 占位符替换为你打算用于本实验室的 Azure 区域的名称，例如 `eastus`）：

   ```powershell
   $location = '<Azure_region>'
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardBSFamily'}
   ```

   > **备注**： 要标识 Azure 区域名称，请在 **Cloud Shell** 中的 PowerShell 提示符处，运行 `(Get-AzLocation).Location`。
   
1. 查看上一步中执行的命令的输出，并确保目标 Azure 区域中 Azure VM 的**标准 DSv3 系列**和 **StandardBSFamily** 中至少有 **20** 个可用的 vCPU。如果已满足此条件，请直接进行下一个练习。否则，继续本练习的下一个任务。 

#### 任务 2：请求增加 vCPU 配额

1. 在 Azure 门户中，搜索并选择 **“订阅”**，然后从 **“订阅”** 边栏选项卡中选择表示你打算用于本实验室的 Azure 订阅的条目。
1. 在 Azure 门户的“订阅”边栏选项卡左侧垂直菜单的 **“设置”** 部分中，选择 **“使用情况 + 配额”**。 
1. 在订阅的 **“使用情况 + 配额”** 边栏选项卡上，选择 **“请求增加”**。
1. 在 **“新建支持请求”** 边栏选项卡的 **“基本信息”** 选项卡上，指定以下设置并选择 **“下一步:解决方案 >”**：

   |设置|值|
   |---|---|
   |问题类型|**服务和订阅限制（配额）**|
   |订阅|将在本实验室中使用的 Azure 订阅的名称|
   |配额类型|**计算 VM（核心数-vCPU 数）订阅限制提高**|
   |支持计划|与目标订阅关联的支持计划的名称|

1. 在 **“新建支持请求”** 边栏选项卡的 **“详细信息”** 选项卡上，选择 **“提供详细信息”** 链接。
1. 在 **“新建支持请求”** 边栏选项卡的 **“配额详细信息”** 选项卡上，指定以下设置并选择 **“保存并继续”**：

   |设置|值|
   |---|---|
   |部署模型|**资源管理器**|
   |位置|你打算在本实验室中使用的 Azure 区域的名称|
   |类型|**标准**|
   |标准|**BS 系列**|
   |新 vCPU 限制|新限制|
   |标准|**DSv3 系列**|
   |新 vCPU 限制|新限制|

   >**备注**：在本例中，使用 **BS 系列** Azure VM 是为了将运行实验室环境的成本降至最低。这并不代表 **BS 系列** Azure VM 在 Azure 虚拟桌面场景中的预期用途。

1. 返回 **“新建支持请求”** 边栏选项卡的 **“详细信息”** 选项卡，指定以下设置并选择 **“下一步: 查看 + 创建 >”**：

   |设置|值|
   |---|---|
   |严重性|**C - 影响极小**|
   |首选联系方式|选择首选选项并提供你的详细联系方式|
    
1. 在 **“新建支持请求”** 边栏选项卡的 **“查看 + 创建”** 选项卡中，选择 **“创建”**。

   > **备注**： 此 vCPU 范围内的配额增加请求通常会在几个小时内完成。但是，无需等待即可完成本实验室。


### 练习 1：实现 Azure Active Directory 域服务 (AD DS) 域

本练习的主要任务如下：

1. 创建和配置 Azure AD 用户帐户以管理 Azure AD DS 域
1. 使用 Azure 门户部署 Azure AD DS 实例
1. 配置 Azure AD DS 部署的网络和标识设置

#### 任务 1：创建和配置 Azure AD 用户帐户以管理 Azure AD DS 域

1. 在实验室计算机上，启动 Web 浏览器，导航至 [Azure 门户](https://portal.azure.com)，然后通过提供用户帐户（该帐户具有你将在本实验室使用的订阅中的所有者角色以及与 Azure 订阅关联的 Azure AD 租户中的全局管理员角色）的凭据进行登录。
1. 在显示 Azure 门户的 Web 浏览器中，导航到 Azure AD 租户的“**概述**”边栏选项卡，并在左侧垂直菜单的“**管理**”部分中，单击“**属性**”。
1. 在 Azure AD 租户的“**属性**”边栏选项卡的最底部，选择“**管理安全默认值**”链接。
1. 如有需要，在“**启用安全默认值**”边栏选项卡上，选择“**否**”，选中“**我的组织正在使用条件访问**”复选框，然后选择“**保存**”。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开 **“Cloud Shell”** 窗格。
1. 提示选择 **“Bash”** 或 **“PowerShell”** 时，选择 **“PowerShell”**。 

   >**备注**： 如果这是第一次启动 **Cloud Shell**，并显示消息 **“未装载任何存储”**，请选择你将在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1. 从“Cloud Shell”窗格，运行以下命令以登录到 Azure AD 租户：

   ```powershell
   Connect-AzureAD
   ```

1. 从“Cloud Shell”窗格，运行以下命令以检索与 Azure 订阅关联的 Azure AD 租户的主 DNS 域名：

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. 在“Cloud Shell”窗格中，运行以下命令以创建将获得提升权限的 Azure AD 用户（将 `<password>` 占位符替换为随机的复杂密码）：

   > **备注**：确保你记住了所用的密码。稍后你将在本实验室或后续实验室中用到它。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. 在“Cloud Shell”窗格中，运行以下命令以将全局管理员角色分配给新创建的第一个 Azure AD 用户：

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **备注**：Azure AD PowerShell 模块将全局管理员角色称为公司管理员。

1. 从“Cloud Shell”窗格，运行以下命令以标识新创建的 Azure AD 用户的用户主体名称：

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **备注**：记录用户主体名称。你在本练习后面需要用到它。 

1. 关闭“Cloud Shell”窗格。
1. 在 Azure 门户中，搜索并选择**订阅**，然后从 **“订阅”** 边栏选项卡选择你在本实验室中使用的 Azure 订阅。 
1. 在显示 Azure 订阅属性的边栏选项卡上，依次选择 **“访问控制(IAM)”**、 **“+ 添加”**，然后在下拉列表中选择 **“添加角色分配”**。 
1. 在 **“添加角色分配”** 边栏选项卡上，指定以下设置并选择 **“保存”**：

   |设置|值|
   |---|---|
   |角色|**所有者**|
   |将访问权限分配到|**用户、组或服务主体**|
   |选择|**aadadmin1**|

   > **备注**： 你稍后将在实验室中使用 **aadadmin1** 帐户来管理 Azure 订阅和加入 Windows 10 Azure VM 的 Azure AD DS 中的相应 Azure AD 租户。 


#### 任务 2：使用 Azure 门户部署 Azure AD DS 实例

1. 在实验室计算机显示的 Azure 门户中，搜索并选择 **“Azure AD 域服务”**，然后从 **“Azure AD 域服务”** 边栏选项卡上选择 **“+ 添加”**。这将打开 **“创建 Azure AD 域服务”** 边栏选项卡。
1. 在 **“创建 Azure AD 域服务”** 边栏选项卡的 **“基本信息”** 选项卡上，指定以下设置并选择 **“下一步”** （其他设置保留为现有值）：

   |设置|值|
   |---|---|
   |订阅|在本实验室中使用的 Azure 订阅的名称|
   |资源组|新资源组名称 **az140-11a-RG**|
   |DNS 域名|**adatum.com**|
   |区域|要托管 AVD 部署的区域的名称|
   |SKU|**标准**|
   |林类型|**用户**|

   > **备注**： 虽然这在技术上不是必需的，但通常应分配一个不同于任何现有 Azure 或本地 DNS 名称空间的 Azure AD DS 域名。

1. 在 **“创建 Azure AD 域服务”** 边栏选项卡的 **“网络”** 选项卡上，选择 **“虚拟网络”** 下拉列表旁边的 **“新建”**。
1. 在“**创建虚拟网络**”边栏选项卡上，指定以下设置并选择“**确定**”：

   |设置|值|
   |---|---|
   |名称|**az140-aadds-vnet11a**|
   |地址范围|**10.10.0.0/16**|
   |子网名称|**aadds-Subnet**|
   |子网名称|**10.10.0.0/24**|

1. 返回 **“创建虚拟网络”** 边栏选项卡的 **“网络”** 选项卡，选择 **“下一步”** （其他设置保留为现有值）。
1. 在 **“创建 Azure AD 域服务”** 边栏选项卡的 **“管理”** 选项卡上，接受默认设置并选择 **“下一步”**。
1. 在 **“创建 Azure AD 域服务”** 边栏选项卡的 **“同步”** 选项卡上，确保 **“所有”** 处于选中状态，然后选择 **“下一步”**。
1. 在 **“创建 Azure AD 域服务”** 边栏选项卡的 **“安全设置”** 选项卡上，接受默认设置并选择 **“下一步”**。
1. 在 **“创建 Azure AD 域服务”** 边栏选项卡的 **“查看 + 创建”** 选项卡上，选择 **“创建”**。 
1. 查看有关在创建 Azure AD DS 域后将无法更改的设置的通知，然后选择 **“确定”**。

   >**备注**：在预配 Azure AD DS 域后将无法更改的设置包括其 DNS 名称、其 Azure 订阅、其资源组、托管其域控制器的虚拟网络和子网以及林类型。

   > **备注**：等待部署完成后，再继续下一个练习。该过程大约需要 90 分钟。 

#### 任务 3：配置 Azure AD DS 部署的网络和标识设置

1. 在实验室计算机显示的 Azure 门户中，搜索并选择 **“Azure AD 域服务”**，然后从 **“Azure AD 域服务”** 边栏选项卡上选择 **“adatum.com”** 条目，以导航到新预配的 Azure AD DS 实例。 
1. 在 Azure AD DS 实例的“**adatum.com**”边栏选项卡上，单击以“**检测到托管域配置问题**”开头的警告。 
1. 在 **“adatum.com | 配置诊断(预览)”**  边栏选项卡上，单击 **“运行”**。
1. 在 **“验证”** 部分，展开 **“DNS 记录”** 窗格并单击 **“修复”**。
1. 在 **“DNS 记录”** 边栏选项卡上，再次单击 **“修复”**。
1. 导航回 Azure AD DS 实例的 **“adatum.com”** 边栏选项卡，并在 **“必需的配置步骤”** 部分中，查看有关 Azure AD DS 密码哈希同步的信息。 

   > **备注**： 任何需要能够访问 Azure AD DS 域计算机及其资源的现有仅限云的用户都必须更改密码或重置密码。这适用于你之前在本实验中创建的 **aadadmin1** 帐户。

1. 在实验室计算机上，在 Azure 门户中的 **“Cloud Shell”** 窗格打开 **PowerShell** 会话。
1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以识别 Azure AD **aadadmin1** 用户帐户的 objectID 属性：

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以重置你在上一步中标识的 objectId 的 **aadadmin1** 用户帐户的密码（将 `<password>` 占位符替换为随机的复杂密码）：

   > **备注**：确保你记住了所用的密码。稍后你将在本实验室或后续实验室中用到它。

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **备注**： 在实际场景中，通常将 **-ForceChangePasswordNextLogin** 的值设置为 $true。在本例中，我们选择 **$false** 以简化实验室步骤。 

1. 重复上述两个步骤，重置 **wvdaadmin1** 用户帐户的密码。


### 练习 2：配置 Azure AD DS 域环境
  
本练习的主要任务如下：

1. 使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM
1. 部署 Azure Bastion
1. 查看 Azure AD DS 域的默认配置
1. 创建将同步到 Azure AD DS 的 AD DS 用户和组

#### 任务 1：使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM

1. 在实验室计算机上，在 Azure 门户的“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，将名为 **cl-Subnet** 的子网添加到你在上一个任务中创建的名为 **az140-aadds-vnet11a** 的虚拟网络：

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 在 Azure 门户的“Cloud Shell”窗格的工具栏中，选择 **“上传/下载文件”** 图标，在下拉菜单中选择 **“上传”**，然后将文件 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** 和 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** 上传到 Cloud Shell 主目录中。
1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以部署运行 Windows 10 的 Azure VM（该 VM 将用作 Azure 虚拟桌面客户端），并将其加入 Azure AD DS 域：

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **备注**： 部署可能需要大约 10 分钟。等待部署完成后，再继续执行下一个任务。 


#### 任务 2：部署 Azure Bastion 

> **备注**：通过 Azure Bastion，可以连接到你在本练习的上一个任务中部署的没有公共终结点的 Azure VM，同时提供针对操作系统级别凭据的暴力攻击防护。

> **备注**：确保你的浏览器启用了弹出功能。

1. 在显示 Azure 门户的浏览器窗口中，打开另一个选项卡，然后在该浏览器选项卡中导航到 Azure 门户。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开 **Cloud Shell** 窗格。
1. 在 Cloud Shell 窗格中的 PowerShell 会话中运行以下命令，将名为 **AzureBastionSubnet** 的子网添加到你在本练习前面部分创建的名为 **az140-adds-vnet11** 的虚拟网络中：

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户中搜索并选择“**Bastion**”，然后在“**Bastion**”边栏选项卡中，选择“**+ 创建**”。
1. 在“**创建 Bastion**”边栏选项卡的“**基本信息**”选项卡中，指定以下设置并选择“**查看 + 创建**”：

   |设置|值|
   |---|---|
   |订阅|在本实验室中使用的 Azure 订阅的名称|
   |资源组|**az140-11a-RG**|
   |名称|**az140-11a-bastion**|
   |区域|在本练习的上一个任务中向其中部署了资源的同一 Azure 区域|
   |层级|**基本**|
   |虚拟网络|**az140-aadds-vnet11a**|
   |子网|**AzureBastionSubnet (10.10.254.0/24)**|
   |公共 IP 地址|**新建**|
   |公共 IP 名称|**az140-aadds-vnet11a-ip**|

1. 在“**创建 Bastion**”边栏选项卡的“**查看 + 创建**”选项卡中，选择“**创建**”：

   > **备注**：请等待部署完成，再继续执行本练习的下一个任务。该部署可能需要大约 5 分钟。


#### 任务 3：查看 Azure AD DS 域的默认配置

> **备注**：在登录到新加入 Azure AD DS 的计算机之前，需要将要登录的用户帐户添加到 **“AAD DC 管理员”** Azure AD 组。此 Azure AD 组是在与预配 Azure AD DS 实例的 Azure 订阅关联的 Azure AD 租户中自动创建的。

> **备注**：预配 Azure AD DS 实例时，可以选择使用现有 Azure AD 用户帐户填充该组。

1. 在实验室计算机上，在 Azure 门户中的“Cloud Shell”窗格，运行以下命令以将 **aadadmin1** Azure AD 用户帐户添加到 **“AAD DC 管理员”** Azure AD 组：

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. 关闭“Cloud Shell”窗格。
1. 在实验室计算机上，在 Azure 门户中搜索并选择 **“虚拟机”**，然后在 **“虚拟机”** 边栏选项卡中，选择 **“az140-cl-vm11a”** 条目。此时会打开 **“az140-cl-vm11a”** 边栏选项卡。
1. 在“**az140-cl-vm11a**”边栏选项卡上，选择“**连接**”，在下拉菜单中选择“**Bastion**”，在“**az140-cl-vm11a \| 连接**”边栏选项卡的“Bastion”选项卡上，选择“**使用 Bastion**”。
1. 系统出现提示时，使用你之前在本实验室中指定的主体名称和之前在本实验室中创建该用户帐户时为其设置的密码，以 **aadadmin1** 用户的身份登录。
1. 在与 **az140-cl-vm11a** Azure VM 的远程桌面中，以管理员身份启动 **Windows PowerShell ISE**，并从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令来安装 Active Directory 以及与 DNS 相关的远程服务器管理工具：

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **备注**：请等待安装完成，再继续下一步。该过程大约需要 2 分钟。

1. 在与 **az140-cl-vm11a** Azure VM 的远程桌面中的 **“开始”** 菜单中，导航到 **“Windows 管理工具”** 文件夹，将其展开，然后从工具列表中启动 **“Active Directory 用户和计算机”**。 
1. 在 **“Active Directory 用户和计算机”** 控制台中，查看默认层次结构，包括 **“AADDC 计算机”** 和 **“AADDC 用户”** 组织单位。请注意，前者包括 **az140-cl-vm11a** 计算机帐户，后者包括从与托管 Azure AD DS 实例部署的 Azure 订阅关联的 Azure AD 租户同步的用户帐户。 **AADDC 用户**组织单位还包括从同一 Azure AD 租户同步的 **AAD DC 管理员**组及其组成员身份。不能直接在 Azure AD DS 域中修改此成员身份，而必须在 Azure AD DS 租户中对其进行管理。任何更改都会自动与 Azure AD DS 域中托管的组的副本同步。 

   > **备注**： 目前，该组仅包括 **aadadmin1** 用户帐户。

1. 在 **“Active Directory 用户和计算机”** 控制台的 **“AADDC 用户”** OU 中，选择 **“aadadmin1”** 用户帐户，显示其 **“属性”** 对话框，切换到 **“帐户”** 选项卡，并注意用户主体名称后缀与主 Azure AD DNS 域名匹配且不可修改。 
1. 在 **“Active Directory 用户和计算机”** 控制台中，查看 **“域控制器”** 组织单位的内容，并注意其中包括两个具有随机生成名称的域控制器的计算机帐户。 

#### 任务 4：创建将同步到 Azure AD DS 的 AD DS 用户和组

1. 在与 **az140-cl-vm11a** Azure VM 的远程桌面中，启动 Microsoft Edge，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供 **aadadmin1** 用户帐户的用户主体名称并将你之前在本实验中设置的密码作为其密码进行登录。
1. 在 Azure 门户中打开 **Cloud Shell**。
1. 当提示你选择 **“Bash”** 或 **“PowerShell”** 时，选择 **“PowerShell”**。 

   >**备注**： 由于这是第一次使用 **aadadmin1**  用户帐户启动 **Cloud Shell**，因此需要配置其 Cloud Shell 主目录。当出现消息 **“未安装存储”** 时，请选择你在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令登录，以对 Azure AD 租户进行身份验证：

   ```powershell
   Connect-AzureAD
   ```

1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以检索与 Azure 订阅关联的 Azure AD 租户的主 DNS 域名：

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以创建你将在后续实验室中使用的 Azure AD 用户帐户（将 `<password>` 占位符替换为随机的复杂密码）：

   > **备注**：确保你记住了所用的密码。稍后你将在本实验室或后续实验室中用到它。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以创建名为 **az140-wvd-aadmins** 的 Azure AD 组并向其添加 **aadadmin1** 和 **wvdaadmin1** 用户帐户：

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. 从“Cloud Shell”窗格，重复上一步，为将在后续实验室中使用的用户创建 Azure AD 组，并向其添加之前创建的 Azure AD 用户帐户：

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. 关闭 Cloud Shell 窗格。
1. 在与 **az140-cl-vm11a** Azure VM 的远程桌面中，在显示 Azure 门户的 Microsoft Edge 窗口中，搜索并选择 **“Azure Active Directory”** 边栏选项卡，在 Azure AD 租户边栏选项卡左侧垂直菜单栏的 **“管理”** 部分中，选择 **“用户”**，然后在 **“用户 \| 所有用户”** 边栏选项卡上，验证是否已创建新用户帐户。
1. 导航回“Azure AD 租户”边栏选项卡，在左侧垂直菜单栏上的 **“管理”** 部分中，选择 **“组”**，然后在 **“组 \| 所有组”** 边栏选项卡上，验证是否已创建新的组帐户。
1. 在与 **az140-cl-vm11a** Azure VM 的远程桌面中，切换到 **“Active Directory 用户和计算机”** 控制台，在 **“Active Directory 用户和计算机”** 控制台中，导航到 **“AADDC 用户”** OU，并验证它是否包含相同的用户和组帐户。

   >**备注**： 你可能需要刷新控制台的视图。
