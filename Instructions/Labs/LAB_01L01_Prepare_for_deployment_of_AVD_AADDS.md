---
lab:
  title: 实验室：准备部署 Azure 虚拟桌面 (Microsoft Entra DS)
  module: 'Module 1: Plan an AVD Architecture'
---

# 实验室 - 准备部署 Azure 虚拟桌面 (Microsoft Entra DS)
# 学生实验室手册

## 实验室依赖项

- Azure 订阅
- Microsoft 帐户或 Microsoft Entra 帐户，在与 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色并在 Azure 订阅中具有所有者或参与者角色

## 预计用时

150 分钟

>**备注**：预配 Microsoft Entra DS 大约需要等待 90 分钟。

## 实验室方案

需要准备在 Azure Active Directory 域服务 (Microsoft Entra DS) 环境中部署 Azure 虚拟桌面

## 目标
  
完成本实验室后，你将能够：

- 实现 Microsoft Entra DS 域
- 配置 Microsoft Entra DS 域环境

## 实验室文件

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## 说明

### 练习 0：增加 vCPU 配额数

此练习的主要任务如下：

1. 确定当前的 vCPU 使用情况
1. 请求增加 vCPU 配额

#### 任务 1：确定当前的 vCPU 使用情况

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。 

   >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。 

1. 如果未注册 Microsoft.Compute**** 资源提供程序，则在 Azure 门户中，在 Cloud Shell**** 的 PowerShell 会话中，运行以下命令进行注册：

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. 在 Azure 门户中 Cloud Shell 的 PowerShell 会话中，运行以下命令以验证 Microsoft.Compute 资源提供程序的注册状态 ：

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**注意**：验证状态是否显示为“已注册”。 如果未显示，请等待几分钟并重复此步骤。

1. 在 Azure 门户中，在 Cloud Shell**** 的 Powershell 会话中，运行以下命令以使用 Azure 区域名称创建 Powershell 变量（将 `<Azure_region>` 占位符替换为要用于此实验室的 Azure 区域的名称，例如 `eastus`）：

   ```powershell
   $location = '<Azure_region>'
   ```

   > **注意**：若要确定 Azure 区域的名称，请在 Cloud Shell 的 PowerShell 提示符下运行 `(Get-AzLocation).Location`。
   
1. 在 Azure 门户中，在 Cloud Shell**** 的 Powershell 会话中，运行以下命令，确定 vCPU 的当前使用情况以及 StandardDSv3Family**** Azure VM 的相应限制：

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. 查看上一步中所执行命令的输出，并确保目标 Azure 区域中 Azure VM 的标准 DSv3 系列**** 中至少有 20**** 个可用的 vCPU。 如果已满足此条件，请直接进行下一个练习。 否则，继续本练习的下一个任务。 

#### 任务 2：请求增加 vCPU 配额

1. 在 Azure 门户中，搜索并选择“订阅”，然后从“订阅”边栏选项卡中选择表示你打算用于本实验室的 Azure 订阅的条目 。
1. 在 Azure 门户的“订阅”边栏选项卡左侧垂直菜单的“设置”部分中，选择“使用情况 + 配额” 。 
1. 在“ **使用情况 + 配额**” 边栏选项卡上，从顶部搜索栏中选择以下下拉箭头：

   |**设置**|**值**|
   |---|---|
   |**搜索**|**标准 DSv3**|
   |所有位置****|全部清除，然后检查你的位置******|
   |**资源提供程序** | **Microsoft.Compute** |
   
1. 在返回的“标准 DSv3 系列 vCPU”**** 项中，选择铅笔图标，选择“编辑”****。
1. 在“配额详细信息”边栏选项卡中的“新建限制”列文本框中，键入“30”，然后选择“保存并继续”****************。
1. 允许配额请求完成。  片刻后，“配额详细信息”边栏选项卡将指定已批准请求并增加配额。 关闭“配额详细信息”边栏选项卡。

    >**注意**：根据 Azure 区域的选择和当前需求，可能需要提出支持请求。 有关创建支持请求过程的说明，请参阅[创建 Azure 支持请求](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)。

### 练习 1：实现 Azure Active Directory 域服务 (AD DS) 域

此练习的主要任务如下：

1. 创建和配置 Microsoft Entra 用户帐户以管理 Microsoft Entra DS 域
1. 使用 Azure 门户部署 Microsoft Entra DS 实例
1. 配置 Microsoft Entra DS 部署的网络和标识设置

#### 任务 1：创建和配置 Microsoft Entra 用户帐户以管理 Microsoft Entra DS 域

1. 从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户的凭据登录，该帐户在本实验室将使用的订阅中具有所有者角色，并在与此 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色。
1. 在显示 Azure 门户的 Web 浏览器中，导航到 Microsoft Entra 租户的“概述”**** 边栏选项卡，并在左侧垂直菜单的“管理”**** 部分中，单击“属性”****。
1. 在 Microsoft Entra 租户的“属性”**** 边栏选项卡的最底部，选择“管理安全默认值”**** 链接。
1. 如有需要，在“启用安全默认值”边栏选项卡上，选择“否”，选中“我的组织正在使用条件访问”复选框，然后选择“保存”   。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。 

   >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。 

1. 在“Cloud Shell”窗格中，运行以下命令以登录到 Microsoft Entra 租户：

   ```powershell
   Connect-AzureAD
   ```

1. 在“Cloud Shell”窗格中，运行以下命令，以检索与 Azure 订阅关联的 Microsoft Entra 租户的主 DNS 域名：

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. 在“Cloud Shell”窗格中，运行以下命令以创建将授予提升的权限的 Microsoft Entra 用户（将 `<password>` 占位符替换为随机的复杂密码）：

   > **注意**：确保记住所使用的密码。 稍后将在本实验室和后续实验室中用到它。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. 在“Cloud Shell”窗格中，运行以下命令，将全局管理员角色分配给第一个新建的 Microsoft Entra 用户：

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **注意**：Azure AD PowerShell 模块将全局管理员角色称为公司管理员。

1. 在“Cloud Shell”窗格中，运行以下命令以确定新建的 Microsoft Entra 用户的用户主体名称：

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **注意**：记录用户主体名称。 本练习中稍后会用到它。 

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户中，搜索并选择“订阅”****，然后从“订阅”**** 边栏选项卡中选择本实验室中使用的 Azure 订阅。 
1. 在显示 Azure 订阅属性的边栏选项卡上，依次选择“访问控制(IAM)”、“添加”和“添加角色分配”************。 
1. 在“添加角色分配”边栏选项卡上，选择“所有者”，然后单击“下一步”************
1. 单击“+ 选择成员”超链接****。
1. 在“选择成员”**** 边栏选项卡中，选择“aadadmin1”**** 项，然后单击“选择”**** 按钮，然后单击“下一步”****。
1. 在“查看 + 分配”边栏选项卡中，选择“查看 + 分配”按钮********。

   > **备注**：在本实验室的后续部分，你将使用 aadadmin1**** 帐户，从一台已加入 Microsoft Entra DS 的 Windows 10 Azure VM 管理 Azure 订阅以及相应 Microsoft Entra 租户。 


#### 任务 2：使用 Azure 门户部署 Microsoft Entra DS 实例

1. 在实验室计算机的 Azure 门户中，搜索并选择“Microsoft Entra 域服务”****，然后从“Microsoft Entra 域服务”**** 边栏选项卡上选择“+ 创建”****。 这将打开“创建 Microsoft Entra 域服务”**** 边栏选项卡。
1. 在“创建 Microsoft Entra 域服务”**** 边栏选项卡的“基本信息”**** 选项卡上，指定以下设置（其他设置保留其现有值）并选择“下一步”****:

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|选择“新建 az140-11a-RG”****|
   |DNS 域名|adatum.com|
   |区域|要托管 AVD 部署的区域的名称|
   |SKU|**标准**|

   > **备注**：虽然在技术上不需要，但一般情况下，应分配不同于任何现有 Azure 或本地 DNS 名称空间的 Microsoft Entra DS 域名。

1. 在“创建 Microsoft Entra 域服务”**** 边栏选项卡的“网络”**** 选项卡上，在“虚拟网络”**** 下拉列表旁边选择“新建”****。
1. 在“创建虚拟网络”**** 边栏选项卡上，分配以下设置，然后选择“确定”****：

   |设置|值|
   |---|---|
   |名称|az140-aadds-vnet11a****|
   |地址范围|**10.10.0.0/16**|
   |子网名称|aadds-Subnet****|
   |地址范围|10.10.0.0/24****|

1. 返回“创建虚拟网络”**** 边栏选项卡的“网络”**** 选项卡，选择“下一步”****（将其他设置保留其现有值）。
1. 在“创建 Microsoft Entra 域服务”**** 边栏选项卡的“管理”**** 选项卡上，接受默认设置并选择“下一步”****。
1. 在“创建 Microsoft Entra 域服务”**** 边栏选项卡的“同步”**** 选项卡上，确保已选择“全部”****，然后选择“下一步”****。
1. 在“创建 Microsoft Entra 域服务”**** 边栏选项卡的“安全设置”**** 选项卡上，接受默认设置并选择“下一步”****。
1. 在“创建 Microsoft Entra 域服务”**** 边栏选项卡的“标记”**** 选项卡上，接受默认设置并选择“下一步”
2. 在“创建 Microsoft Entra 域服务”**** 边栏选项卡的“查看 + 创建”**** 选项卡上，选择“创建”****。 
3. 查看有关在创建 Microsoft Entra DS 域后无法更改的设置的通知，然后选择“确定”****。

   >**备注**：预配 Microsoft Entra DS 域后无法更改的设置包括其 DNS 名称、Azure 订阅、资源组、托管其域控制器的虚拟网络和子网以及林类型。

   > 注意：在继续下一个练习之前，请等待部署完成。 这可能需要大约 90 分钟。 

#### 任务 3：配置 Microsoft Entra DS 部署的网络和标识设置

1. 在实验室计算机的 Azure 门户中，搜索并选择“Microsoft Entra 域服务”****，然后从“Microsoft Entra 域服务”**** 边栏选项卡中，选择“adatum.com”**** 条目以导航到新预配的 Microsoft Entra DS 实例。 
1. 在 Microsoft Entra DS 实例的“adatum.com”**** 边栏选项卡上，单击以**检测到托管域的配置问题**开头的警告。 
1. 在“adatum.com | 配置诊断”**** 边栏选项卡上，单击“运行”****。
1. 在“验证”**** 部分中，展开“DNS 记录”**** 窗格，然后单击“修复”****。
1. 在“DNS 记录”**** 边栏选项卡上，再次单击“修复”****。
1. 导航回 Microsoft Entra DS 实例的“adatum.com”**** 边栏选项卡，并在“所需配置步骤”**** 部分中查看有关 Microsoft Entra DS 密码哈希同步的信息。 

   > **备注**：任何需要能够访问 Microsoft Entra DS 域计算机及其资源的现有仅云端用户都必须更改其密码或重置密码。 这适用于之前在此实验室中创建的 aadadmin1**** 帐户。

1. 从实验室计算机的 Azure 门户中，打开“Cloud Shell”**** 窗格中的“PowerShell”**** 会话。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以确定 Microsoft Entra aadadmin1**** 用户帐户的 objectID 属性：

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以重置你在上一步中标识的 objectId 的 aadadmin1 用户帐户的密码（将 `<password>` 占位符替换为随机的复杂密码）****：

   > **注意**：确保记住所使用的密码。 稍后将在本实验室和后续实验室中用到它。

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **备注**：在实际方案中，通常会将“-ForceChangePasswordNextLogin”**** 的值设置为 $true。 在本案例中，我们选择了 $false**** 来简化实验室步骤。 

1. 重复前面的两个步骤，重置 wvdaadmin1**** 用户帐户的密码。


### 练习 2：配置 Microsoft Entra DS 域环境
  
此练习的主要任务如下：

1. 使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM
1. 部署 Azure Bastion
1. 查看 Microsoft Entra DS 域的默认配置
1. 创建将同步到 Microsoft Entra DS 的 AD DS 用户和组

#### 任务 1：使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM

1. 在实验室计算机上的 Azure 门户中，从“Cloud Shell”窗格中的 PowerShell 会话中运行以下命令，将名为 cl-Subnet**** 的子网添加到在上一个任务中创建的名为 **az140-aadds-vnet11a** 的虚拟网络中 ：

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 在实验室计算机中，找到 \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json 参数文件，并使用Visual Studio Code打开该文件****。

1.  在第 21 行，找到 domainPassword 参数的值。 更新参数文件中的现有密码，以使用你在本实验室前面为 aadadmin1 用户帐户设置的密码，然后保存文件********。

1. 在 Azure 门户的“Cloud Shell”窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中选择“上传”，然后将文件 \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json 和 \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json 上传到 Cloud Shell 主目录中****************。
1. 从“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，以部署运行 Windows 10、将用作 Azure 虚拟桌面客户端的 Azure VM 并将其加入 Microsoft Entra DS 域：

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

   > 备注：部署可能需要大约 10 分钟的时间。 等待部署完成后再继续下一个任务。 


#### 任务 2：部署 Azure Bastion 

> **注意**：Azure Bastion 允许在没有公共终结点（你在本练习的上一个任务中部署的终结点）的情况下连接到 Azure VM，同时提供针对暴力攻击（利用操作系统级别凭据）的保护。

> **注意**：请确保浏览器已启用弹出窗口功能。

1. 在显示 Azure 门户的浏览器窗口中，打开另一个选项卡，并在浏览器选项卡中导航到 [**Azure 门户**](https://portal.azure.com)。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以将名为 AzureBastionSubnet 的子网添加到你在本练习前面创建的虚拟网络 az140-adds-vnet11 中 ：

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
1. 在 Azure 门户中，搜索并选择“Bastion”，然后从“Bastion”边栏选项卡中选择“+ 创建”  。
1. 在“创建 Bastion”边栏选项卡的“基本”选项卡上，指定以下设置并选择“查看 + 创建”  ：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|az140-11a-RG****|
   |名称|az140-11a-bastion****|
   |区域|在本练习的先前任务中部署资源的同一 Azure 区域|
   |层|**基本**|
   |虚拟网络|az140-aadds-vnet11a****|
   |子网|AzureBastionSubnet (10.10.254.0/24)****|
   |公共 IP 地址|**新建**|
   |公共 IP 名称|az140-aadds-vnet11a-ip****|

1. 在“创建 Bastion”边栏选项卡的“查看 + 创建”选项卡上，选择“创建”  ：

   > **备注**：请等待部署完成再继续此练习的下一个任务。 部署可能需要大约 5 分钟时间。


#### 任务 3：查看 Microsoft Entra DS 域的默认配置

> **备注**：在登录到新加入 Microsoft Entra DS 的计算机之前，需要将登录要使用的用户帐户添加到 AAD DC 管理员**** Microsoft Entra 组。 此 Microsoft Entra 组会在与预配 Microsoft Entra DS 实例的 Azure 订阅关联的 Microsoft Entra 租户中自动创建。

> **备注**：预配 Microsoft Entra DS 实例时，可以选择使用现有的 Microsoft Entra 用户帐户填充此组。

1. 在实验室计算机的 Azure 门户中，从“Cloud Shell”窗格中运行以下命令，将 aadadmin1**** Microsoft Entra 用户帐户添加到 AAD DC 管理员**** Microsoft Entra 组：

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. 关闭 Cloud Shell 窗格。
1. 在实验室计算机的 Azure 门户中，搜索并选择“虚拟机”****，然后从“虚拟机”**** 边栏选项卡中，选择“az140-cl-vm11a”**** 条目。 这将打开“az140-cl-vm11a”**** 边栏选项卡。
1. 在“az140-cl-vm11a”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-cl-vm11a”边栏选项卡的“Bastion”选项卡中，提供以下凭据并选择“连接”************************：
1. 出现提示时，请以 aadadmin1 用户身份登录，使用之前在本实验室中确定的其主体名称以及之前在实验室中创建此用户帐户时为其设置的密码****。
1. 在与 az140-cl-vm11a**** Azure VM 的 Bastion 会话中，以管理员身份启动 Windows PowerShell ISE****，并从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令以安装 Active Directory 和 DNS 相关的远程服务器管理工具：

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **备注**：等待安装完成再继续下一步。 这可能需要大约 2 分钟。

1. 在与 az140-cl-vm11a**** Azure VM 的 Bastion 会话中，在“开始”**** 菜单中，导航到“Windows 管理员工具”**** 文件夹，将其展开，然后从工具列表中启动“Active Directory 用户和计算机”****。 
1. 在“Active Directory 用户和计算机”**** 控制台中，查看默认层次结构，包括“AADDC 计算机”**** 和“AADDC 用户”**** 组织单位。 请注意，前者包括 az140-cl-vm11a**** 计算机帐户，后者包括从与托管 Microsoft Entra DS 实例部署的 Azure 订阅关联的 Microsoft Entra 租户同步的用户帐户。 “AADDC 用户”**** 组织单位还包括从同一 Microsoft Entra 租户同步的“AAD DC 管理员”**** 组及其组成员身份。 不能直接在 Microsoft Entra DS 域中修改此成员身份，但必须在 Microsoft Entra DS 租户中对其进行管理。 任何更改都会自动与 Microsoft Entra DS 域中托管的组的副本同步。 

   **提示：** 如果“Active Directory 用户和计算机”**** 未列出任何与域相关的内容，则右键单击“Active Directory 用户和计算机”****，选择“更改域”****，并选择域“Adatum”****。

   > **备注**：目前，该组仅包含 aadadmin1**** 用户帐户。

1. 在“Active Directory 用户和计算机”**** 控制台的“AADDC 用户”**** OU 中，选择“aadadmin1”**** 用户帐户，显示其“属性”**** 对话框，切换到“帐户”**** 选项卡，请注意，用户主体名称后缀与 Microsoft Entra DNS 主域名匹配且不可修改。 
1. 在“Active Directory 用户和计算机”**** 控制台中，查看“域控制器”**** 组织单位的内容，请注意，它包括具有随机生成名称的两个域控制器的计算机帐户。 

#### 任务 4：创建将同步到 Microsoft Entra DS 的 AD DS 用户和组

1. 在与 az140-cl-vm11a**** Azure VM 的 Bastion 会话中，启动 Microsoft Edge，导航到[Azure 门户](https://portal.azure.com)，然后提供 aadadmin1**** 用户帐户的用户主体名称并使用之前在本实验室中设置的密码作为其密码进行登录。
1. 在 Azure 门户中打开“Cloud Shell”****。
1. 如果系统提示选择“Bash”**** 或“PowerShell”****，请选择“PowerShell”****。 

   >**备注**：由于这是首次使用 aadadmin1**** 用户帐户启动 Cloud Shell****，因此需要配置其 Cloud Shell 主目录。 当出现“未装载任何存储”**** 消息时，请选择在本实验室中使用的订阅，然后选择“创建存储”****。 

1. 在“Cloud Shell”窗格内的 PowerShell 会话中，运行下列命令登录以对 Microsoft Entra 租户进行身份验证：

   ```powershell
   Connect-AzureAD
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令来检索与 Azure 订阅关联的 Microsoft Entra 租户的主 DNS 域名：

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令以创建将在后续实验室中使用的 Microsoft Entra 用户帐户（将 `<password>` 占位符替换为随机的复杂密码）：

   > **注意**：确保记住所使用的密码。 稍后将在本实验室和后续实验室中用到它。

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

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以创建名为 az140-wvd-aadmins**** 的 Microsoft Entra 组，并将其添加到 aadadmin1**** 和 wvdaadmin1**** 用户帐户：

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. 在“Cloud Shell”窗格中，重复上一步，为将在后续实验室中使用的用户创建 Microsoft Entra 组，并将其添加到之前创建的 Microsoft Entra 用户帐户：

   >**备注**：备注：由于虚拟机上剪贴板的大小有限，并非所有列出的 cmdlet 都能正确复制。 打开虚拟机上的记事本，并使用类型文本、类型剪贴板文本构造（属于 Lightning Bolt 控件的一部分）将所有 cmdlet 复制到其中。 确保所有 cmdlet 都位于记事本中后，以块的形式将其剪切并粘贴到 Cloud Shell 中，并运行它们。

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
1. 在与 az140-cl-vm11a**** Azure VM 的 Bastion 中，在显示 Azure 门户的 Microsoft Edge 窗口中，搜索并选择“Azure Active Directory”**** 边栏选项卡，在“Microsoft Entra 租户”边栏选项卡左侧垂直菜单栏的“管理”**** 部分中，选择“用户”****，然后在“用户”\|“所有用户”**** 边栏选项卡上，验证是否已创建新用户帐户。
1. 导航回“Microsoft Entra 租户”边栏选项卡，在左侧垂直菜单栏上的“管理”**** 部分中，选择“组”****，然后在“组”\|“所有组”**** 边栏选项卡上，验证新组帐户是否已创建。
1. 在与名为 az140-cl-vm11a**** 的 Azure VM 之间的 Bastion 会话中，切换到“Active Directory 用户和计算机”**** 控制台，在“Active Directory 用户和计算机”**** 控制台中，导航到“AADDC 用户”**** OU，并验证它是否包含相同的用户和组帐户。

   >**备注**：可能需要刷新控制台视图。
