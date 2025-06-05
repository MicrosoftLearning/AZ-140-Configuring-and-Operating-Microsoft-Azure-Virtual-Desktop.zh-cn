---
lab:
  title: 实验室：使用 PowerShell (AD DS) 部署和管理主机池和主机
  module: 'Module 2: Implement a WVD Infrastructure'
---

# 实验室 - 使用 PowerShell 部署和管理主机池和主机
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有将在本实验室中使用的 Azure 订阅的所有者或参与者角色，以及与 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”

## 预计用时

60 分钟

## 实验室方案

需要在 Active Directory 域服务 (AD DS) 环境中使用 PowerShell 自动部署 Azure 虚拟桌面主机池和主机。

## 目标
  
完成本实验室后，你将能够：

- 使用 PowerShell 部署 Azure 虚拟桌面主机池和主机
- 使用 PowerShell 将主机添加到 Azure 虚拟桌面主机池

## 实验室文件

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## 说明

### 练习 1：使用 PowerShell 实施 Azure 虚拟桌面主机池和会话主机
  
此练习的主要任务如下：

1. 准备使用 PowerShell 部署 Azure 虚拟桌面主机池
1. 使用 PowerShell 创建 Azure 虚拟桌面主机池
1. 使用 PowerShell 对运行 Windows 11 企业版的 Azure VM 执行基于模板的部署
1. 使用 PowerShell 将运行 Windows 11 企业版的 Azure VM 作为会话主机添加到 Azure 虚拟桌面主机
1. 验证 Azure 虚拟桌面会话主机的部署

#### 任务 1：准备使用 PowerShell 部署 Azure 虚拟桌面主机池

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az140-dc-vm11”  。
1. 在“az140-dc-vm11”边栏选项卡上，选择“连接”，在下拉菜单中选择“通过 Bastion 进行连接”************。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，以管理员身份启动 Windows PowerShell ISE****。
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员:  Windows PowerShell ISE”**** 脚本窗格运行以下命令，以安装 DesktopVirtualization PowerShell 模块（如果出现提示，请单击“全是”****）：

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **备注**：忽略有关正在使用的现有 PowerShell 模块的任何警告。

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，启动 Microsoft Edge，并导航到 [Azure 门户](https://portal.azure.com)。 如果出现提示，请使用在本实验室使用的订阅中具有所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”**** 文本框搜索并导航到“虚拟网络”****，然后在“虚拟网络”**** 边栏选项卡上选择“az140-adds-vnet11”****。 
1. 在“az140-adds-vnet11”边栏选项卡上，选择“子网”，在“子网”边栏选项卡中选择“+ 子网”，在“添加子网”边栏选项卡上，指定以下设置（所有其他设置保留默认值），并单击“保存”     ：

   |设置|值|
   |---|---|
   |名称|hp3-Subnet****|
   |子网地址范围|**10.0.3.0/24**|

#### 任务 2：使用 PowerShell 创建 Azure 虚拟桌面主机池

1. 在与 **az140-dc-vm11** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以标识托管 Azure 虚拟网络“az140-adds-vnet11”**** 的 Azure 区域：

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以创建将托管主机池及其资源的资源组：

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. 在与 **az140-dc-vm11** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以创建空主机池：

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **备注**：使用 New-AzWvdHostPool**** cmdlet 可以创建主机池、工作区和桌面应用组，以及向工作区注册桌面应用组。 可选择新建工作区或使用现有的工作区。

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台中运行以下命令，以检索名为“az140-wvd-pooled”**** 的 Azure AD 组的 objectID 属性：

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台中运行以下命令，以将名为“az140-wvd-pooled”**** 的 Azure AD 组分配到新建主机池的默认桌面应用组：

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### 任务 3：使用 PowerShell 对运行 Windows 11 企业版的 Azure VM 执行基于模板的部署

1. 从实验室计算机导航到已部署的存储帐户。 在“文件共享”边栏选项卡上，选择 az140-22-profiles 文件共享****。

1. 选择“上传”并将实验室文件 \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json 和 \\\\AZ-140 AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json\\ 上传到文件共享************。

1. 在与 az140-dc-vm11 的 Bastion 会话中，打开文件资源管理器并导航到先前配置的 Z:，或分配给文件共享连接的驱动器号****。 将上传的部署文件复制到 C：\AllFiles\Labs\02****。

1. 在与 az140-dc-vm11 的 Bastion 会话中，从“管理员: ******Windows PowerShell ISE**控制台，运行以下命令部署运行 Windows 11 企业版（多会话）的 Azure VM，该 VM 将用作在前面的任务中创建的主机池中的 Azure 虚拟桌面会话主机：

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

   > 注意：在继续下一个任务之前，请等待部署完成。 这可能需要大约 5-10 分钟。 

   > **备注**：该部署会使用 Azure 资源管理器模板来预配 Azure VM，并应用可自动将操作系统添加到 adatum.com**** AD DS 域的 VM 扩展。

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员：Windows PowerShell ISE”**** 控制台中运行以下命令，以验证第三方会话主机是否成功加入了 AD DS 域 adatum.com****：

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### 任务 4：使用 PowerShell 将运行 Windows 11 企业版的 Azure VM 作为主机添加到 Azure 虚拟桌面主机池

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，在显示 Azure 门户的浏览器窗口中，搜索并选择“虚拟机”****，然后在“虚拟机”**** 边栏选项卡的虚拟机列表中选择“az140-24-p3-0”****。
1. 在“az140-24-p3-0”边栏选项卡上，选择“连接”，在下拉菜单中选择“连接”************。
1. 确保**连接使用**显示**专用 IP 地址 | 10.0.3.4**
1. 在**本机 RDP**部分中，选择“下载 RDP 文件”****。
1. 如果出现提示，请选择“保留”，然后点击下载的**az140-24-p3-0.rdp**文件****。
1. 系统出现提示时，请使用以下凭据登录：

   |设置|值|
   |---|---|
   |用户名|ADATUM\\Student|
   |密码|**Pa55w.rd1234**|

1. 在与 **az140-24-p3-0** 的远程桌面会话中，以管理员身份启动 Windows PowerShell ISE****。
1. 在与 az140-24-p3-0**** 的远程桌面会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以创建一个文件夹，用于保存将新部署的 Azure VM 作为会话主机添加到此前在本实验室中预配的主机池所需的文件：

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

   >请注意，谨慎使用 [T] 构造复制 PowerShell cmdlet****。 在某些情况下，复制的文本可能不正确，例如 $ 符号显示为 4 个数字字符。 发出 cmdlet 之前，需要更正这些内容。 复制到 PowerShell ISE“脚本”窗格，在此进行更正，然后突出显示更正的文本并按 F8（运行选择）************。

1. 在与 az140-24-p3-0**** 的远程桌面会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，下载 Azure 虚拟桌面代理和启动加载器安装程序（将会话主机添加到主机池所必需的操作）：

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. 在“管理员:**Windows PowerShell ISE**控制台，运行以下命令以安装最新版本的 Az PowerShell 模块，当提示安装 NuGet 提供程序时，请按**Y**：

   ```powershell
   Install-Module -Name Az -Force
   ```

   > **注意**：可能需要等待 3-5 分钟，然后才会显示 Az 模块安装的任何输出。 在输出停止后，可能需要再等待 5 分钟****。 这是预期的行为。

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以登录 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请提供具有本实验室所用订阅的所有者角色的用户帐户的凭据。
1. 在与 az140-24-p3-0 的远程桌面会话中，从“管理员: ******Windows PowerShell ISE**控制台，运行以下命令以生成将此会话主机加入在本练习前面配置的池中所需的令牌：

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **备注**：授权会话主机加入主机池需要使用注册令牌。 令牌到期日期的值必须介于当前日期和时间之后的 1 小时到 1 个月之间。

1. 在与 az140-24-p3-0**** 的远程桌面会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，以安装 Azure 虚拟桌面代理：

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. 在与 az140-24-p3-0**** 的远程桌面会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，以安装 Azure 虚拟桌面启动加载程序：

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

1. 在**az140-24-p3-0**的远程桌面会话中，右击“开始”，在右击菜单中，选择“关闭或注销”，然后在级联菜单中点击“注销”************。

#### 任务 5：验证 Azure 虚拟桌面主机的部署

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择“Azure 虚拟桌面”，在“Azure 虚拟桌面”边栏选项卡上，选择“主机池”，然后在“Azure 虚拟桌面 \| 主机池”边栏选项卡上，选择表示新修改池的条目“az140-24-hp3”********************。
1. 在“az140-24-hp3”边栏选项卡左侧垂直菜单中的“管理”部分中，单击“会话主机”************。 
1. 在“az140-24-hp3 \| 会话主机”边栏选项卡上，验证该部署是否包含单个主机****。

#### 任务 6：使用 PowerShell 管理应用组

1. 在实验室计算机上，切换到与 az140-dc-vm11**** 的 Bastion 会话，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，以创建远程应用组：

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，以在池主机上列出“开始”**** 菜单应用并查看输出：

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **备注**：对于要发布的任何应用程序，应记录输出中包含的信息，包括 FilePath****、IconPath**** 和 IconIndex **** 等参数。

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台中运行以下命令以发布 Microsoft Word：

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令以发布 Microsoft Word：

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. 切换到实验室计算机，在显示Azure 门户的 Web 浏览器中，在“az140-24-hp3 \| 会话主机”边栏选项卡上，在左侧垂直菜单中的“管理”部分，选择“应用程序组”************。
1. 在“az140-24-hp3 \|应用程序组”边栏选项卡的应用程序组列表中，选择“az140-24-hp3-Office365-RAG”条目********。
1. 在“az140-24-hp3-Office365-RAG”**** 边栏选项卡上，验证应用程序组的配置，包括应用程序和分配。

### 练习 2：停止并解除分配在实验室中预配的 Azure VM

此练习的主要任务如下：

1. 停止并解除分配在实验室中预配的 Azure VM

>**注意**：在此练习中，你将解除分配此实验室中预配的 Azure VM，以最大程度减少相应的计算费用

#### 任务 1：解除分配在实验室中预配的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开 Cloud Shell 窗格内的“PowerShell”shell 会话 。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注意**：该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能停止和解除分配 Azure VM。
