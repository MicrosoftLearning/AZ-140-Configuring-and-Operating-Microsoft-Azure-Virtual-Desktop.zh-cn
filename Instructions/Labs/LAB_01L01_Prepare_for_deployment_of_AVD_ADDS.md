---
lab:
  title: 实验室：准备部署 Azure 虚拟桌面 (AD DS)
  module: 'Module 1: Plan a AVD Architecture'
---

# <a name="lab---prepare-for-deployment-of-azure-virtual-desktop-ad-ds"></a>实验室 - 准备部署 Azure 虚拟桌面 (AD DS)
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- 本实验室将使用的 Azure 订阅。
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有将在本实验室中使用的 Azure 订阅的所有者或参与者角色，以及与 Azure 订阅关联的 Azure AD 租户的全局管理员角色。

## <a name="estimated-time"></a>预计用时

60 分钟

## <a name="lab-scenario"></a>实验室方案

你需要准备部署 Active Directory 域服务 (AD DS) 环境

## <a name="objectives"></a>目标
  
完成本实验室后，你将能够：

- 使用 Azure VM 部署 Active Directory 域服务 (AD DS) 单域林
- 将 AD DS 林与 Azure Active Directory (Azure AD) 租户集成

## <a name="lab-files"></a>实验室文件 

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## <a name="instructions"></a>说明

### <a name="exercise-0-increase-the-number-of-vcpu-quotas"></a>练习 0：增加 vCPU 配额数

此练习的主要任务如下：

1. 确定当前的 vCPU 使用情况
1. 请求增加 vCPU 配额

#### <a name="task-1-identify-current-vcpu-usage"></a>任务 1：确定当前的 vCPU 使用情况

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。 

   >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。 

1. 如果未注册 Microsoft.Compute 和 Microsoft.Network 资源提供程序，则在 Azure 门户中 Cloud Shell 的 PowerShell 会话中，运行以下命令进行注册  ：

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. 在 Azure 门户中 Cloud Shell 的 PowerShell 会话中，运行以下命令以验证 Microsoft.Compute 资源提供程序的注册状态 ：

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**注意**：验证状态是否显示为“已注册”。 如果未显示，请等待几分钟并重复此步骤。

1. 在 Azure 门户中，在 Cloud Shell 的 PowerShell 会话中，运行以下命令以确定 vCPU 的当前使用情况以及 StandardDSv3Family 和 StandardBSFamily Azure VM 的相应限制（将 `<Azure_region>` 占位符替换为你打算用于本实验室的 Azure 区域的名称，例如 `eastus`）  ：

   ```powershell
   $location = '<Azure_region>'
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardBSFamily'}
   ```

   > **注意**：若要确定 Azure 区域的名称，请在 Cloud Shell 的 PowerShell 提示符下运行 `(Get-AzLocation).Location`。
   
1. 查看上一步中执行的命令的输出，并确保目标 Azure 区域中 Azure VM 的标准 DSv3 系列 vCPU 和标准 BS 系列 vCPU 中至少有 30 个可用的 vCPU  。 如果已满足此条件，请直接进行下一个练习。 否则，继续本练习的下一个任务。 

#### <a name="task-2-request-vcpu-quota-increase"></a>任务 2：请求增加 vCPU 配额

1. 在 Azure 门户中，搜索并选择“订阅”，然后从“订阅”边栏选项卡中选择表示你打算用于本实验室的 Azure 订阅的条目 。
1. 在 Azure 门户的“订阅”边栏选项卡左侧垂直菜单的“设置”部分中，选择“使用情况 + 配额” 。 

   **注意：** 你可能不需要提出支持工单来增加配额。

   **注意：** 请求增加配额需要使用多重身份验证 (MFA) 登录。 如果需要使用 MFA 配置帐户，请参阅[规划 Azure Active Directory 多重身份验证部署](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted)。 
   
1. 在“Azure Pass - 赞助 | 使用情况 + 配额”边栏选项卡上，选择“区域”，在下拉列表中选中要用于此实验室的 Azure 区域名称旁边的复选框，选择“应用”，确保“计算”条目出现在“区域”条目左侧的下拉列表中，然后在搜索框中键入“标准 BS”     。 
1. 在结果列表中，选中“标准 BS 系列 vCPU”项旁边的复选框，选择工具栏中的“请求配额增加”条目，然后在下拉列表中选择“输入新限制”  。
1. 在“请求配额增加”窗格中的“新建限制”列文本框中，键入“30”，然后选择“提交”   。
1. 如果出现提示，请在“请求配额增加”窗格中，选择“使用多重身份验证进行身份验证”，并按照提示进行身份验证 。
1. 允许配额请求完成。  片刻后，“配额详细信息”边栏选项卡将指定已批准请求并增加配额。 关闭“配额详细信息”边栏选项卡。
1. 重复步骤 3-7，将标准 DSv3 VM 大小的配额限制设置为 30 。

   >**注意**：根据 Azure 区域的选择和当前需求，可能需要提出支持请求。 有关创建支持请求过程的说明，请参阅[创建 Azure 支持请求])https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request) 。

### <a name="exercise-1-deploy-an-active-directory-domain-services-ad-ds-domain"></a>练习 1：部署 Active Directory 域服务 (AD DS) 域

此练习的主要任务如下：

1. 准备 Azure VM 部署
1. 使用 Azure 资源管理器快速启动模板来部署运行 AD DS 域控制器的 Azure VM
1. 使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM
1. 部署 Azure Bastion

#### <a name="task-1-prepare-for-an-azure-vm-deployment"></a>任务 1：准备 Azure VM 部署

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在显示 Azure 门户的 Web 浏览器中，导航回 Azure AD 租户的“概述”边栏选项卡，并在左侧垂直菜单的“管理”部分中，单击“属性”  。
1. 在 Azure AD 租户的“属性”边栏选项卡的最底部，选择“管理安全默认值”链接 。
1. 如有需要，在“启用安全默认值”边栏选项卡上，选择“否”，选中“我的组织正在使用条件访问”复选框，然后选择“保存”   。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。 

   >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。 


#### <a name="task-2-deploy-an-azure-vm-running-an-ad-ds-domain-controller-by-using-an-azure-resource-manager-quickstart-template"></a>任务 2：使用 Azure 资源管理器快速启动模板来部署运行 AD DS 域控制器的 Azure VM

1. 在实验室计算机上，在显示 Azure 门户的 Web 浏览器中，从“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以创建资源组（将 `<Azure_region>` 占位符替换为要用于此实验室的 Azure 区域的名称，例如 `eastus`）：

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. 在 Azure 门户中，关闭 Cloud Shell 窗格。
1. 在实验室计算机的同一 Web 浏览器窗口中，打开另一个 Web 浏览器选项卡并导航到名为[创建新的 Windows VM 并创建新的 AD 林、域和 DC](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain) 的自定义版快速入门模板。 
1. 在“创建新的 Windows VM 并创建新的 AD 林、域和 DC”页上，选择“部署到 Azure” 。 这会自动将浏览器重定向到 Azure 门户中的“使用新的 AD 林创建 Azure VM”边栏选项卡。
1. 在“使用新的 AD 林创建 Azure VM”边栏选项卡中，选择“编辑参数”。
1. 在“编辑参数”边栏选项卡中，选择“加载文件”，在“打开”对话框中，选择“\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json”，然后依次选择“打开”和“保存”     。 
1. 在“使用新的 AD 林创建 Azure VM”边栏选项卡中，指定以下设置（将其他设置保留为默认值）：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|az140-11-RG|
   |域名|adatum.com|

1. 在“使用新的 AD 林创建 Azure VM”边栏选项卡中，选择“查看 + 创建”，然后单击“创建”  。

   > 注意：请等待部署完成再继续下一个练习。 这可能需要大约 15 分钟。 

#### <a name="task-3-deploy-an-azure-vm-running-windows-10-by-using-an-azure-resource-manager-quickstart-template"></a>任务 3：使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM

1. 在实验室计算机上，在显示 Azure 门户的 Web 浏览器中，在“Cloud Shell”窗格中打开 PowerShell 会话并运行以下命令，将名为 cl-Subnet 的子网添加到在上一个任务中创建的名为 az140-adds-vnet11 的虚拟网络中 ：

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 在 Azure 门户的“Cloud Shell”窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中选择“上传”，然后将文件 \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json 和 \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json 上传到 Cloud Shell 主目录中   。
1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令将运行 Windows 10 的 Azure VM（该 VM 将用作客户端）部署到新建的子网：

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > 注意：请不要等待部署完成，而是继续执行下一个任务。 部署可能需要大约 10 分钟时间。

#### <a name="task-4-deploy-azure-bastion"></a>任务 4：部署 Azure Bastion 

> **注意**：Azure Bastion 允许在没有公共终结点（你在本练习的上一个任务中部署的终结点）的情况下连接到 Azure VM，同时提供针对暴力攻击（利用操作系统级别凭据）的保护。

> **注意**：请确保浏览器已启用弹出窗口功能。

1. 在显示 Azure 门户的浏览器窗口中，打开另一个选项卡，并在浏览器选项卡中导航到 Azure 门户。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以将名为 AzureBastionSubnet 的子网添加到你在本练习前面创建的虚拟网络 az140-adds-vnet11 中 ：

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户中，搜索并选择“Bastion”，然后从“Bastion”边栏选项卡中选择“+ 创建”  。
1. 在“创建 Bastion”边栏选项卡的“基本”选项卡上，指定以下设置并选择“查看 + 创建”  ：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|az140-11-RG|
   |名称|az140-11-bastion|
   |区域|在本练习的先前任务中部署资源的同一 Azure 区域|
   |层|**基本**|
   |虚拟网络|az140-adds-vnet11|
   |子网|AzureBastionSubnet (10.0.254.0/24)|
   |公共 IP 地址|**新建**|
   |公共 IP 名称|az140-adds-vnet11-ip|

1. 在“创建 Bastion”边栏选项卡的“查看 + 创建”选项卡上，选择“创建”  ：

   > 注意：请等待部署完成再继续下一个练习。 部署可能需要大约 5 分钟时间。

### <a name="exercise-2-integrate-an-ad-ds-forest-with-an-azure-ad-tenant"></a>练习 2：将 AD DS 林与 Azure AD 租户集成
  
此练习的主要任务如下：

1. 创建将同步到 Azure AD 的 AD DS 用户和组
1. 配置 AD DS UPN 后缀
1. 创建将用于配置与 Azure AD 同步的 Azure AD 用户
1. 安装 Azure AD Connect
1. 配置混合 Azure AD 联接

#### <a name="task-1-create-ad-ds-users-and-groups-that-will-be-synchronized-to-azure-ad"></a>任务 1：创建将同步到 Azure AD 的 AD DS 用户和组

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中选择“az140-dc-vm11”  。
1. 在“az140-dc-vm11”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-dc-vm11 \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-dc-vm11 的远程桌面会话中，以管理员身份启动 Windows PowerShell ISE 。
1. 在“管理员:Windows PowerShell ISE”脚本窗格中，运行以下命令禁用针对管理员的 Internet Explorer 增强安全性：

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中，运行以下命令以创建一个 AD DS 组织单位，该组织单位将包含同步到本实验室中使用的 Azure AD 租户的范围内的对象：

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中，运行以下命令以创建一个 AD DS 组织单位，该组织单位将包含已加入域的 Windows 10 客户端计算机的计算机对象：

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. 在“管理员:Windows PowerShell ISE”脚本窗格中，运行以下命令创建 AD DS 用户帐户，这些帐户将同步到本实验室中使用的 Azure AD 租户（将 `<password>` 占位符替换为随机的复杂密码）：

   > **注意**：确保记住所使用的密码。 稍后将在本实验室和后续实验室中用到它。

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **注意**：该脚本创建了九个名为 aduser1 - aduser9 的非特权用户帐户和一个名为 wvdadmin1 的 ADATUM\\Domain Admins 组成员的特权帐户   。

1. 在“管理员:Windows PowerShell ISE”脚本窗格中，运行以下命令以创建将同步到本实验室中使用的 Azure AD 租户的 AD DS 组对象：

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以向你在前一个步骤中创建的组添加成员：

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### <a name="task-2-configure-ad-ds-upn-suffix"></a>任务 2：配置 AD DS UPN 后缀

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格中运行以下命令，以安装最新版本的 PowerShellGet 模块（如果提示确认，请选择“是”）：

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以安装最新版本的 Az PowerShell 模块（如果提示确认，请选择“全部确认”）：

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以登录 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请提供具有本实验室所用订阅的所有者角色的用户帐户的凭据。
1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以检索与 Azure 订阅关联的 Azure AD 租户的 ID 属性：

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以安装和导入最新版本的 Azure AD PowerShell 模块：

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以对 Azure AD 租户进行身份验证：

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. 如果出现提示，请使用之前在此任务中使用的相同凭据登录。 
1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以检索与 Azure 订阅关联的 Azure AD 租户的主 DNS 域名：

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以将与 Azure 订阅关联的 Azure AD 租户的主 DNS 域名添加到 AD DS 林的 UPN 后缀列表中：

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. 在“管理员:Windows PowerShell ISE”脚本窗格中运行以下命令，以将与 Azure 订阅关联的 Azure AD 租户的主 DNS 域名分配为 AD DS 域中所有用户的 UPN 后缀：

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以将 adatum.com UPN 后缀分配给学生域用户 ：

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### <a name="task-3-create-an-azure-ad-user-that-will-be-used-to-configure-directory-synchronization"></a>任务 3：创建将用于配置目录同步的 Azure AD 用户

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格，运行以下命令以创建新的 Azure AD 用户（将 `<password>` 占位符替换为随机的复杂密码）：

   > **注意**：确保记住所使用的密码。 稍后将在本实验室和后续实验室中用到它。

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. 在“管理员:Windows PowerShell ISE”脚本窗格中运行以下命令，以将全局管理员角色分配给新创建的 Azure AD 用户： 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **注意**：Azure AD PowerShell 模块将全局管理员角色称为公司管理员。

1. 在“管理员:Windows PowerShell ISE”脚本窗格中运行以下命令，以标识新创建的 Azure AD 用户的用户主体名称：

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **注意**：记录用户主体名称。 本练习中稍后会用到它。 


#### <a name="task-4-install-azure-ad-connect"></a>任务 4：安装 Azure AD Connect

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格中运行以下命令，以启用 TLS 1.2：

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. 在与 az140-dc-vm11 的远程桌面会话中，启动 Internet Explorer，导航到 [Microsoft Edge 企业版下载页](https://www.microsoft.com/en-us/edge/business/download)。
1. 从 [Microsoft Edge 企业版下载页](https://www.microsoft.com/en-us/edge/business/download)下载 Microsoft Edge 的最新稳定版本，安装、启动该版本，并使用默认设置对其进行配置。
1. 在与 az140-dc-vm11 的远程桌面会话中，使用 Microsoft Edge 导航到 [Azure 门户](https://portal.azure.com)。 如果出现提示，请使用在本实验室使用的订阅中具有所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“Azure Active Directory”边栏选项卡，在“Azure AD 租户”选项卡上，在中心菜单的“管理”部分，选择“Azure AD Connect”   。
1. 在“Azure AD Connect”边栏选项卡上，先选择“连接同步”链接，然后选择“下载 Azure AD Connect”链接  。 这将自动打开新的浏览器标签页，其中显示了“Microsoft Azure Active Directory Connect”下载页。
1. 在“Microsoft Azure Active Directory Connect”下载页，选择“下载” 。
1. 如果系统提示是运行还是保存 AzureADConnect.msi 安装程序，请选择“运行” 。 否则，请在下载文件后打开该文件以启动 Microsoft Azure Active Directory Connect 向导。
1. 在“Microsoft Azure Active Directory Connect”向导的“欢迎使用 Azure AD Connect”页面中，选中复选框“我同意许可条款和隐私声明”，然后选择“继续”   。
1. 在“Microsoft Azure Active Directory Connect”向导的“快速设置”页面中，选择“自定义”选项。
1. 在“安装所需组件”页面中，取消选择所有可选配置选项并选择“安装”。
1. 在“用户登录”页面中，确保仅启用“密码哈希同步”，然后选择“下一步”。
1. 在“连接到 Azure AD”页面中，使用你在上一个练习中创建的“aadsyncuser”用户帐户凭据进行身份验证，然后选择“下一步”  。 

   > **注意**：提供在本练习前面记录的 aadsyncuser 帐户的 userPrincipalName 属性，并指定你之前在本实验中设置的密码作为其密码。

1. 在“连接目录”页面上，选择“adatum.com”林条目右边的“添加目录”按钮  。
1. 在“AD 林帐户”窗口中，确保选择“新建 AD 帐户”选项，指定以下凭据，然后选择“确定”：

   |设置|值|
   |---|---|
   |用户名|ADATUM\Student|
   |密码|**Pa55w.rd1234**|

1. 返回“连接目录”页面，确保“adatum.com”条目显示为已配置的目录，然后选择“下一步”  
1. 在“Azure AD 登录配置”页面中，看到警告说明“如果 UPN 后缀与已验证域名不匹配，用户将无法使用本地凭据登录 Azure AD”，启用复选框“继续，不将所有 UPN 后缀与已验证域匹配”，然后选择“下一步”。

   > **注意**：这是意料之中的，因为 Azure AD 租户没有与 adatum.com AD DS 的其中一个 UPN 后缀匹配的经过验证的自定义 DNS 域。

1. 在“域和 OU 筛选”页面上，在选择选项“同步选定的域和 OU”，展开 adatum.com 节点，清除所有复选框，仅选中“ToSync” OU 附近的复选框，然后选择“下一步”   。
1. 在“唯一标识你的用户”页面中，接受默认设置，然后选择“下一步” 。
1. 在“筛选用户和设备”页面中接受默认设置，然后选择“下一步”。
1. 在“可选功能”页面中接受默认设置，然后选择“下一步”。
1. 在“准备配置”页面在确保选中“配置完成后启动同步过程”复选框，然后选择“安装”。

   > **注意**：安装需要约 2 分钟。

1. 查看“配置完成”页中的信息，然后选择“退出”关闭“Microsoft Azure Active Directory Connect”窗口。
1. 在与 az140-vm11 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中，导航到 Adatum 实验室 Azure AD 租户的“用户”-“所有用户”边栏选项卡 。
1. 在“用户 \| 所有用户”边栏选项卡中，请注意，用户对象列表包括在本实验室前面创建的 AD DS 用户帐户列表，其中“是”条目显示在“本地同步已启用”列中  。

   > **注意**：可能需要等待几分钟，然后刷新浏览器页面，AD DS 用户帐户才会出现。
