---
lab:
    title: '实验室：使用 PowerShell 部署和管理主机池和主机'
    module: '模块 2：实现 WVD 基础结构'
---

# 实验室 - 使用 PowerShell 部署和管理主机池和主机
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有将在本实验室中使用的 Azure 订阅的所有者或参与者角色，以及与 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
- 已完成实验室 **“准备部署 Azure 虚拟桌面 (AD DS)”**

## 预计用时

60 分钟

## 实验室场景

你需要在 Active Directory 域服务 (AD DS) 环境中使用 PowerShell 自动部署 Azure 虚拟桌面主机池和主机。

## 目标
  
完成本实验室后，你将能够：

- 使用 PowerShell 部署 Azure 虚拟桌面主机池和主机
- 使用 PowerShell 将主机添加到 Azure 虚拟桌面主机池

## 实验室文件

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## 说明

### 练习 1：使用 PowerShell 实现 Azure 虚拟桌面主机池和会话主机
  
本练习的主要任务如下：

1. 使用 PowerShell 准备部署 Azure 虚拟桌面主机池
1. 使用 PowerShell 创建 Azure 虚拟桌面主机池
1. 使用 PowerShell 对运行 Windows 10 企业版的 Azure VM 执行基于模板的部署
1. 使用 PowerShell 将运行 Windows 10 企业版的 Azure VM 作为会话主机添加到 Azure 虚拟桌面主机
1. 验证 Azure 虚拟桌面会话主机的部署

#### 任务 1：使用 PowerShell 准备部署 Azure 虚拟桌面主机池

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，搜索并选择 **“虚拟机”**，然后在 **“虚拟机”** 边栏选项卡中，选择 **“az140-dc-vm11”**。
1. 在“**az140-dc-vm11**”边栏选项卡上，选择“**连接**”，在下拉菜单中选择“**Bastion**”，在“**az140-dc-vm11 \| 连接**”边栏选项卡的“**Bastion**”选项卡上，选择“**使用 Bastion**”。
1. 出现提示时，请提供以下凭据并选择“**连接**”：

   |设置|值|
   |---|---|
   |用户名|**Student**|
   |密码|**Pa55w.rd1234**|

1. 在与 **az140-dc-vm11** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令以标识名为 **WVDInfra** 的组织单位的可分辨名称，该组织单位将托管 Azure 虚拟桌面池主机的计算机会话对象：

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令确定 **“ADATUM\\Student”** 帐户（该帐户用于将 Azure 虚拟桌面主机加入 AD DS 域 (**student@adatum.com**） 的 UPN 后缀：

   ```powershell
   (Get-ADUser -Filter {sAMAccountName -eq 'student'} -Properties userPrincipalName).userPrincipalName
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令安装 DesktopVirtualization PowerShell 模块（系统出现提示时，选择 **“全部确认”**）：

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **备注**： 忽略有关正在使用的现有 PowerShell 模块的任何警告。

1. 在与 **az140-dc-vm11** 的远程桌面会话中，启动 Microsoft Edge，导航到 [Azure 门户](https://portal.azure.com)。如果出现提示，请使用在本实验室使用的订阅中具有所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，在 Azure 门户中，使用 Azure 门户页顶部的 **“搜索资源、服务和文档”** 文本框，搜索并导航到 **“虚拟网络”**，然后在 **“虚拟网络”** 边栏选项卡中选择 **“az140-adds-vnet11”**。 
1. 在 **“az140-adds-vnet11”** 边栏选项卡上，选择 **“子网”**，在 **“子网”** 边栏选项卡中选择 **“+ 子网”**，在 **“添加子网”** 边栏选项卡上，指定以下设置（所有其他设置保留默认值），并单击 **“保存”**：

   |设置|值|
   |---|---|
   |名称|**hp3-Subnet**|
   |子网地址范围|**10.0.3.0/24**|

1. 在与 **az140-dc-vm11** 的远程桌面会话中，在 Azure 门户中，使用 Azure 门户页顶部的 **“搜索资源、服务和文档”** 文本框，搜索并导航到 **“网络安全组”**，然后在 **“网络安全组”** 边栏选项卡中选择 **“az140-11-RG”** 资源组中的安全组。
1. 在“网络安全组”边栏选项卡上的左侧垂直菜单中，在 **“设置”** 部分中，单击 **“属性”**。
1. 在 **“属性”** 边栏选项卡中，单击 **“资源 ID”** 文本框右侧的 **“复制到剪贴板”** 图标。 

   > **备注**： 虽然订阅 ID 会有所不同，但该值的格式应类似于 `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`。记录该值，因为你在下一个任务中需要使用它。

#### 任务 2：使用 PowerShell 创建 Azure 虚拟桌面主机池

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令登录到 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请提供具有本实验室所用订阅的所有者角色的用户帐户的凭据。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令确定托管 Azure 虚拟网络 **az140-adds-vnet11** 的 Azure 区域：

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令创建将托管主机池及其资源的资源组：

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令创建空主机池：

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **备注**： 通过 **New-AzWvdHostPool** cmdlet 可创建主机池、工作区和桌面应用组，以及将桌面应用组注册到工作区。可选择创建新工作区或使用现有的工作区。

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令检索名为 **az140-wvd-pooled** 的 Azure AD 组的 objectID 属性：

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令将名为 **az140-wvd-pooled** 的 Azure AD 组分配到新建的主机池的默认桌面应用组：

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### 任务 3：使用 PowerShell 对运行 Windows 10 企业版的 Azure VM 执行基于模板的部署

1. 在实验室计算机上，使用与 **az140-dc-vm11** Azure VM 的远程桌面会话，将实验室文件 **“\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json”** 和 **“\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json”** 复制到 **“C:\\AllFiles\\Labs\\02”** 文件夹（如果需要请创建）。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令部署运行 Windows 10 企业版（多会话）的 Azure VM，该 VM 将用作在前面的任务中创建的主机池中的 Azure 虚拟桌面会话主机：

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **备注**： 等待部署完成后，再继续执行下一个任务。该过程大约需要 5 分钟。 

   > **备注**： 该部署使用 Azure 资源管理器模板来预配 Azure VM，并应用一个 VM 扩展，自动将操作系统加入 **“adatum.com”** AD DS 域。

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令验证第三方会话主机是否成功加入 **“adatum.com”** AD DS 域：

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### 任务 4：使用 PowerShell 将运行 Windows 10 企业版的 Azure VM 作为主机添加到 Azure 虚拟桌面主机池

1. 在与 **az140-dc-vm11** 的远程桌面会话中，在显示 Azure 门户的浏览器窗口，搜索并选择 **“虚拟机”**，并且在 **“虚拟机”** 边栏选项卡的虚拟机列表中选择 **“az140-24-p3-0”**。
1. 在 **“az140-24-p3-0”** 边栏选项卡中，选择 **“连接”**，在下拉菜单中选择 **“RDP”**，在 **“az140-24-p3-0 \| 连接”** 边栏选项卡的 **“RDP”** 选项卡的 **“IP 地址”** 下拉列表中，选择 **“公共 IP 地址 (10.0.3.4)”** 条目，然后选择 **“下载 RDP 文件”**。
1. 系统出现提示时，请使用以下凭据登录：

   |设置|值|
   |---|---|
   |用户名|**ADATUM\\Student**|
   |密码|**Pa55w.rd1234**|

1. 在与 **az140-24-p3-0** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**。
1. 在与 **az140-24-p3-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令创建一个文件夹，用于托管将新部署的 Azure VM 作为会话主机添加到本实验室前面预配的主机池所需的文件：

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

1. 在与 **az140-24-p3-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令下载 Azure 虚拟桌面代理和启动加载器安装程序（将会话主机添加到主机池必需的操作）：

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. 在与 **az140-24-p3-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令安装最新版本的 PowerShellGet 模块（如果提示确认，请选择 **“是”**）：

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令安装最新版本的 Az.DesktopVirtualization PowerShell 模块：

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -AllowClobber -Force
   Install-Module -Name Az -AllowClobber -Force
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令修改 PowerShell 执行策略并登录 Azure 订阅：

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser -Force
   Connect-AzAccount
   ```

1. 如果出现提示，请提供具有本实验室所用订阅的所有者角色的用户帐户的凭据。
1. 在与 **az140-24-p3-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令以生成将新会话主机加入在本练习前面配置的池中所需的令牌：

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **备注**： 需要注册令牌才能授权会话主机加入主机池。令牌的到期日期值必须介于从当前日期和时间起一小时到一个月之间。

1. 在与 **az140-24-p3-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令安装 Azure 虚拟桌面代理：

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. 在与 **az140-24-p3-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令安装 Azure 虚拟桌面启动加载器：

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

#### 任务 5：验证 Azure 虚拟桌面主机的部署

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择 **“Azure 虚拟桌面”**，在 **“Azure 虚拟桌面”** 边栏选项卡上，选择 **“主机池”**，然后在 **“Azure 虚拟桌面 \| 主机池”** 边栏选项卡上，选择表示新修改的池的条目 **az140-24-hp3**。
1. 在 **“az140-24-hp3”** 边栏选项卡左侧垂直菜单中的 **“管理”** 部分中，单击 **“会话主机”**。 
1. 在 **“az140-24-hp3 \| 会话主机”** 边栏选项卡上，验证该部署是否包含单个主机。

#### 任务 6：使用 PowerShell 管理应用组

1. 在实验室计算机上，切换到与 **az140-dc-vm11** 的远程桌面会话，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令创建远程应用组：

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令在池主机上列出 **“开始”** 菜单应用并查看输出：

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **备注**： 对于任何要发布的应用程序，都应记录包含在输出中的信息，其中包括 **FilePath**、 **IconPath** 和 **IconIndex** 等参数。

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令发布 Microsoft Word：

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令发布 Microsoft Word：

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器中，在 **“az140-24-hp3 \| 会话主机”** 边栏选项卡左侧垂直菜单中的 **“管理”** 部分，选择 **“应用程序组”**。
1. 在 **“az140-24-hp3 \| 应用程序组”** 边栏选项卡的应用程序组列表中，选择 **“az140-24-hp3-Office365-RAG”** 条目。
1. 在 **“az140-24-hp3-Office365-RAG”** 边栏选项卡上，验证应用程序组的配置，其中包括应用程序和分配。

### 练习 2：停止并解除分配在实验室中预配的 Azure VM

本练习的主要任务如下：

1. 停止并解除分配在实验室中预配的 Azure VM

>**备注**： 在此练习中，你将解除分配此实验室中预配的 Azure VM，以最大程度减少相应的计算费用

#### 任务 1：解除分配在实验室中预配的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开 **“Cloud Shell”** 窗格内的 **“PowerShell”** shell 会话。
1. 在 “Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**备注**： 该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能停止和解除分配 Azure VM。
