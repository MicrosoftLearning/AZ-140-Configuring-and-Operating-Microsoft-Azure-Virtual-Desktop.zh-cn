---
lab:
    title: '实验室：创建和管理会话主机映像 (AD DS)'
    module: '模块 2：实现 WVD 基础结构'
---

# 实验室 - 创建和管理会话主机映像 (AD DS)
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有将在本实验室中使用的 Azure 订阅的所有者或参与者角色，以及与 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
- 已完成实验室 **“准备部署 Azure 虚拟桌面 (AD DS)”**

## 预计用时

60 分钟

## 实验室场景

你需要在 Active Directory 域服务 (AD DS) 环境中创建和管理 Azure 虚拟桌面主机映像。

## 目标
  
完成本实验室后，你将能够：

- 创建和管理 WVD 会话主机映像

## 实验室文件

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## 说明

### 练习 1：创建和管理会话主机映像
  
本练习的主要任务如下：

1. 准备配置 Azure 虚拟桌面主机映像
1. 部署 Azure Bastion
1. 配置 Azure 虚拟桌面主机映像
1. 创建 Azure 虚拟桌面主机映像
1. 使用自定义映像预配 Azure 虚拟桌面主机池

#### 任务 1：准备配置 Azure 虚拟桌面主机映像

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开 **“Cloud Shell”** 窗格。
1. 提示选择 **“Bash”** 或 **“PowerShell”** 时，选择 **“PowerShell”**。 
1. 在实验室计算机显示 Azure 门户的 Web 浏览器中，从“Cloud Shell”窗格内的 PowerShell 会话运行以下命令，以创建将包含 Azure 虚拟桌面主机映像的资源组：

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. 在 Azure 门户的 Cloud Shell 窗格的工具栏中，选择**上传/下载文件**图标，在下拉菜单中选择 **“上传”**，然后将文件 **“\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json”** 和 **“\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json”** 上传到 Cloud Shell 主目录中。
1. 从“Cloud Shell”窗格中的 PowerShell 会话，运行以下命令将运行 Windows 10 的 Azure VM（该 VM 将用作 Azure 虚拟桌面客户端）部署到新建的子网：

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **备注**：等待部署完成后，再继续下一个练习。部署可能需要大约 10 分钟。

#### 任务 2： 部署 Azure Bastion 

> **备注**：通过 Azure Bastion，可以连接到你在本练习的上一个任务中部署的没有公共终结点的 Azure VM，同时提供针对操作系统级别凭据的暴力攻击防护。

> **备注**：确保你的浏览器启用了弹出功能。

1. 在显示 Azure 门户的浏览器窗口中，打开另一个选项卡，然后在该浏览器选项卡中导航到 Azure 门户。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开 **Cloud Shell** 窗格。
1. 在 Cloud Shell 窗格中的 PowerShell 会话中运行以下命令，将名为 **AzureBastionSubnet** 的子网添加到你在本练习前面部分创建的名为 **az140-25-vnet** 的虚拟网络中：

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户中搜索并选择“**Bastion**”，然后在“**Bastion**”边栏选项卡中，选择“**+ 创建**”。
1. 在“**创建 Bastion**”边栏选项卡的“**基本信息**”选项卡中，指定以下设置并选择“**查看 + 创建**”：

   |设置|值|
   |---|---|
   |订阅|在本实验室中使用的 Azure 订阅的名称|
   |资源组|**az140-25-RG**|
   |名称|**az140-25-bastion**|
   |区域|在本练习前面的任务中向其中部署了资源的同一 Azure 区域|
   |层级|**基本**|
   |虚拟网络|**az140-25-vnet**|
   |子网|**AzureBastionSubnet (10.25.254.0/24)**|
   |公共 IP 地址|**新建**|
   |公共 IP 名称|**az140-25-vnet-ip**|

1. 在“**创建 Bastion**”边栏选项卡的“**查看 + 创建**”选项卡中，选择“**创建**”：

   > **备注**：等待部署完成后，再继续下一个练习。该部署可能需要大约 5 分钟。

#### 任务 3： 配置 Azure 虚拟桌面主机映像

1. 在 Azure 门户中，搜索并选择 **“虚拟机”**，然后在 **“虚拟机”** 边栏选项卡上，选择 **“az140-25-vm0”**。
1. 在“**az140-25-vm0**”边栏选项卡上，选择“**连接**”，在下拉菜单中选择“**Bastion**”，在“**az140-25-vm0 \| 连接**”边栏选项卡的“**Bastion**”选项卡上，选择“**使用 Bastion**”。
1. 出现提示时，请提供以下凭据并选择“**连接**”：

   |设置|值|
   |---|---|
   |用户名|**Student**|
   |密码|**Pa55w.rd1234**|

   > **备注**：首先将安装 FSLogix 二进制文件。

1. 在与 **az140-25-vm0** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**。
1. 在与 **az140-25-vm0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令，创建将用作映像配置临时位置的文件夹：

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. 在与 **az140-25-vm0** 的远程桌面会话中，启动 Microsoft Edge，浏览至 [FSLogix 下载页](https://aka.ms/fslogix_download)，将 FSLogix 压缩安装二进制文件下载到 **“C:\Allfiles\Labs\02”** 文件夹中，然后在文件资源管理器中，将 **x64** 子文件夹解压缩到同一文件夹。
1. 在与 **az140-25-vm0** 的远程桌面会话中，切换到 **“管理员: Windows PowerShell ISE”** 窗口，然后从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令以执行按计算机安装 OneDrive：

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **备注**：请等待安装完成。这大约需要 1 分钟。若安装触发重启，请重新连接到 **az140-25-vm0**。

   > **备注**：接下来，你将逐步安装并配置 Microsoft Teams（出于学习目的，因为 Teams 已经存在于本实验室使用的映像中）。

1. 在与 **az140-25-vm0** 的远程桌面会话中，右键单击 **“开始”**，在右键单击菜单中，选择 **“运行”**，在 **“运行”** 对话框的 **“打开”** 文本框中，键入 **“cmd”** 并按 **Enter** 键，以启动 **“命令提示符”**。
1. 在 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行以下命令以准备按计算机安装 Microsoft Teams：

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. 在与 **az140-25-vm0** 的远程桌面会话中，在 Microsoft Edge 中导航至 [Microsoft Visual C++ 可再发行程序包下载页](https://aka.ms/vs/16/release/vc_redist.x64.exe)，将 **VC_redist.x64** 保存至 **“C:\\Allfiles\\Labs\\02”** 文件夹。
1. 在与 **az140-25-vm0** 的远程桌面会话中，切换到 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行以下命令以执行安装 Microsoft Visual C++ 可再发行程序包：

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. 在与 **az140-25-vm0** 的远程桌面会话中，在 Microsoft Edge 中浏览至标题为[
将 Teams 桌面应用部署到 VM](https://docs.microsoft.com/zh-cn/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm) 的文档页，单击 **“64 位版本”** 链接，然后系统出现提示时，将 **Teams_windows_x64.msi** 文件保存至 **“C:\\Allfiles\\Labs\\02”** 文件夹。
1. 在与 **az140-25-vm0** 的远程桌面会话中，切换到 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行以下命令以执行按计算机安装 Microsoft Teams：

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **备注**： 此安装程序支持 ALLUSER=1 和 ALLUSERS=1 参数。ALLUSER=1 参数用于在 VDI 环境中的按计算机安装。ALLUSERS=1 参数可用于非 VDI 环境和 VDI 环境。 

1. 在与 **az140-25-vm0** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**，并从“**管理员: Windows PowerShell ISE**”控制台中运行以下命令，以安装 Microsoft Edge Chromium（出于学习目的，因为 Microsoft Edge 已存在于本实验室使用的映像中）：

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **备注**： 请等待安装完成。该过程大约需要 2 分钟。

   > **备注**： 在多语言环境进行操作时，可能需要安装语言包。有关此过程的详细信息，请参阅 Microsoft Docs 文章[将语言包添加到 Windows 10 多会话映像](https://docs.microsoft.com/zh-cn/azure/virtual-desktop/language-packs)。

   > **备注**： 接下来，你将禁用 Windows 自动更新和存储感知、配置时区重定向以及配置遥测收集。通常，应先应用所有当前的更新。在本实验室中，跳过此步骤是为了最大程度地缩短实验室时间。

1. 在与 **az140-25-vm0** 的远程桌面会话中，切换到 **“管理员: C:\windows\system32\cmd.exe”** 窗口，从命令提示符运行以下命令以禁用自动更新：

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. 在 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行以下命令以禁用存储感知：

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. 在 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行以下命令以配置时区重定向：

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. 在 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行以下命令以禁用反馈中心遥测数据收集：

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. 在 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行以下命令以删除前面在本任务中创建的临时文件夹：

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. 在 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行磁盘清理实用工具，在完成后单击 **“确定”**：

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### 任务 4：创建 Azure 虚拟桌面主机映像

1. 在与 **az140-25-vm0** 的远程桌面会话中，在 **“管理员: C:\windows\system32\cmd.exe”** 窗口中，从命令提示符运行 Sysprep 实用工具，以便让操作系统准备好生成映像并自动将操作系统关闭：

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown
   ```

   > **备注**： 请等待 Sysprep 过程完成。该过程大约需要 2 分钟。这会自动关闭操作系统。 

1. 在实验室计算机显示 Azure 门户的 Web 浏览器中，搜索并选择 **“虚拟机”**，然后在 **“虚拟机”** 边栏选项卡上，选择 **“az140-25-vm0”** 条目。
1. 在 **“az140-25-vm0”** 边栏选项卡 **“Essentials”** 部分上方的工具栏中，单击 **“刷新”**，验证 Azure VM 的 **“状态”** 是否已更改为 **“已停止”**，单击 **“停止”**，并在出现确认提示时单击 **“确定”**，从而将 Azure VM 的状态转换为 **“已停止（已解除分配）”**。
1. 在 **“az140-25-vm0”** 边栏选项卡中，验证 Azure VM 的 **“状态”** 是否已更改为 **“已停止（已解除分配）”**，并在工具栏中单击 **“捕获”**。这将自动显示 **“创建映像”** 边栏选项卡。
1. 在 **“创建映像”** 边栏选项卡的 **“基本”** 选项卡中，指定以下设置：

   |设置|值|
   |---|---|
   |将映像共享到 Azure 计算库|**是，将其作为映像版本共享到库**|
   |创建映像后自动删除此虚拟机|复选框已清除|
   |目标 Azure 计算库|新库名称 **az14025imagegallery**|
   |操作系统状态|**通用**|

1. 在 **“创建映像”** 边栏选项卡的 **“基本”** 选项卡中，单击 **“目标映像定义”** 文本框下方的 **“新建”**。
1. 在 **“创建映像定义”** 中，指定以下设置，然后单击 **“确定”**：

   |设置|值|
   |---|---|
   |映像定义名称|**az140-25-host-image**|
   |发布者|**MicrosoftWindowsDesktop**|
   |产品/服务|**office-365**|
   |SKU|**20h1-evd-o365pp**|

1. 返回 **“创建映像”** 边栏选项卡的 **“基本”** 选项卡上，指定以下设置并单击 **“查看 + 创建”**：

   |设置|值|
   |---|---|
   |版本号|**1.0.0**|
   |从最新版中排除|复选框已清除|
   |生命周期终结日期|距离当前日期还有一年|
   |默认副本计数|**1**|
   |目标区域副本计数|**1**|
   |存储帐户类型|**高级 SSD**|

1. 在 **“创建映像”** 边栏选项卡的 **“查看 + 创建”** 选项卡上，单击 **“创建”**。

   > **备注**： 请等待部署完成。该操作需要约 20 分钟。

1. 在实验室计算机上显示 Azure 门户的 Web 浏览器中，搜索并选择“**Azure 计算库**”，在“**Azure 计算库**”边栏选项卡上选择“**az14025imagegallery**”条目，然后在“**az14025imagegallery**”边栏选项卡上，验证表示新建映像的 **az140-25-host-image** 条目是否存在。

#### 任务 5：使用自定义映像预配 Azure 虚拟桌面主机池

1. 在实验室计算机的 Azure 门户中，使用 Azure 门户页顶部的 **“搜索资源、服务和文档”** 文本框，搜索并导航到 **“虚拟网络”**，然后在 **“虚拟网络”** 边栏选项卡中选择 **“az140-adds-vnet11”**。 
1. 在 **“az140-adds-vnet11”** 边栏选项卡中选择 **“子网”**，在 **“子网”** 边栏选项卡中选择 **“+ 子网”**，在 **“添加子网”** 边栏选项卡中指定以下设置（保留所有其他设置的默认值）后单击 **“保存”**：

   |设置|值|
   |---|---|
   |名称|**hp4-Subnet**|
   |子网地址范围|**10.0.4.0/24**|

1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，搜索并选择 **“Azure 虚拟桌面”**，在 **“Azure 虚拟桌面”** 边栏选项卡上，选择 **“主机池”**，然后在 **“Azure 虚拟桌面 \| 主机池”** 边栏选项卡中，选择 **“+ 添加”**。 
1. 在 **“创建主机池”** 边栏选项卡的 **“基本信息”** 选项卡中，指定以下设置并选择 **“下一步: 虚拟机 >”**：

   |设置|值|
   |---|---|
   |订阅|在本实验室中使用的 Azure 订阅的名称|
   |资源组|**az140-25-RG**|
   |主机池名称|**az140-25-hp4**|
   |位置|在本实验室的第一个练习中，将资源部署到其中的 Azure 区域的名称|
   |验证环境|**否**|
   |主机池类型|**共用**|
   |最大会话限制|**50**|
   |负载均衡算法|**广度优先**|

1. 在 **“创建主机池”** 边栏选项卡的 **“虚拟机”** 选项卡中，指定以下设置：

   |设置|值|
   |---|---|
   |添加虚拟机|**是**|
   |资源组|**默认与主机池相同**|
   |名称前缀|**az140-25-p4**|
   |虚拟机位置|在本实验室的第一个练习中，将资源部署到其中的 Azure 区域的名称|
   |可用性选项|无需基础结构冗余|
   |映像类型|**库**|

1. 在 **“创建主机池”** 边栏选项卡的 **“虚拟机”** 选项卡上，单击 **“映像”** 下拉列表正下方的 **“查看所有映像”** 链接。
1. 在 **“选择映像”** 边栏选项卡上，依次单击 **“我的项”** 和 **“共享映像”**，然后在共享映像列表中选择 **“az140-25-host-image”**。 
1. 在 **“创建主机池”** 边栏选项卡的 **“虚拟机”** 选项卡中，指定以下设置并选择 **“下一步: 工作区 >”**

   |设置|值|
   |---|---|
   |虚拟机大小|**标准 D2s v3**|
   |VM 数|**1**|
   |OS 磁盘类型|**标准 SSD LRS**|
   |虚拟网络|**az140-adds-vnet11**|
   |子网|**hp4-Subnet (10.0.4.0/24)**|
   |网络安全组|**基本**|
   |公共入站端口|**是**|
   |允许的入站端口|**RDP**|
   |AD 域加入 UPN|**student@adatum.com**|
   |密码|**Pa55w.rd1234**|
   |指定域或单元|**是**|
   |要加入的域|**adatum.com**|
   |组织单位路径|**OU=WVDInfra,DC=adatum,DC=com**|
   |用户名|Student|
   |密码|Pa55w.rd1234|
   |确认密码|Pa55w.rd1234|

1. 在 **“创建主机池”** 边栏选项卡的 **“工作区”** 选项卡中，指定以下设置并选择 **“查看 + 创建”**：

   |设置|值|
   |---|---|
   |注册桌面应用组|**否**|

1. 在 **“创建主机池”** 边栏选项卡的 **“查看 + 创建”** 选项卡中，选择 **“创建”**。

   > **备注**： 请等待部署完成。这需要约 10 分钟时间。

   > **备注**： 在根据自定义映像部署主机后，应考虑运行虚拟桌面优化工具（由[它的 GitHub 存储库](https://github.com/The-Virtual-Desktop-Team/)提供）。


### 练习 2：停止并解除分配在实验室中预配的 Azure VM

本练习的主要任务如下：

1. 停止并解除分配在实验室中预配的 Azure VM

>**备注**：在此练习中，你将解除分配此实验室中预配的 Azure VM，以最大程度减少相应的计算费用

#### 任务 1：解除分配在实验室中预配的 Azure VM

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器窗口中，在 **Cloud Shell** 窗格中打开“**PowerShell**” Shell 会话。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   >**备注**： 该命令异步执行（由 -NoWait 参数确定），因此，尽管可立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上要花几分钟才能停止和解除分配 Azure VM。
