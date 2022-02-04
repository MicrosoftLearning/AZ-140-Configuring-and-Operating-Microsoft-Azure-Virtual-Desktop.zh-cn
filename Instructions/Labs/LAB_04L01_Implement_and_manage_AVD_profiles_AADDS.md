---
lab:
    title: '实验室：实现和管理 Azure 虚拟桌面配置文件 (Azure AD DS)'
    module: '模块 4：管理用户环境和应用'
---

# 实验室 - 实现和管理 Azure 虚拟桌面配置文件 (Azure AD DS)
# 学生实验室手册

## 实验室依赖项

- Azure 订阅
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有与 Azure 订阅关联的 Azure AD 租户的全局管理员角色，以及 Azure 订阅的所有者角色或参与者角色
- 实验室 **“Azure 虚拟桌面简介 (Azure AD DS)”** 中预配的 Azure 虚拟桌面环境

## 预计用时

30 分钟

## 实验室场景

你需要在 Azure Active Directory 域服务 (Azure AD DS) 环境中实现 Azure 虚拟桌面配置文件管理。

## 目标
  
完成本实验室后，你将能够：

- 配置 Azure 文件存储，以在 Azure AD DS 环境中存储 Azure 虚拟桌面的配置文件容器
- 在 Azure AD DS 环境中实现基于 FSLogix 的 Azure 虚拟桌面配置文件

## 实验室文件

- 无

## 说明

### 练习 1：实现基于 FSLogix 的 Azure 虚拟桌面配置文件

本练习的主要任务如下：

1. 在 Azure 虚拟桌面会话主机 VM 上配置本地管理员组
1. 在 Azure 虚拟桌面会话主机 VM 上配置基于 FSLogix 的配置文件
1. 在 Azure 虚拟桌面中测试基于 FSLogix 的配置文件
1. 删除 Azure 实验室资源

#### 任务 1：在 Azure 虚拟桌面会话主机 VM 上配置本地管理员组

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在实验室计算机上，在 Azure 门户中搜索并选择 **“虚拟机”**，然后在 **“虚拟机”** 边栏选项卡中，选择 **“az140-cl-vm11a”** 条目。此时会打开 **“az140-cl-vm11a”** 边栏选项卡。
1. 在“**az140-cl-vm11a**”边栏选项卡上，选择“**连接**”，在下拉菜单中选择“**Bastion**”，在“**az140-cl-vm11a \| 连接**”边栏选项卡的“**Bastion**”选项卡上，选择“**使用 Bastion**”。
1. 出现提示时，请提供以下凭据并选择“**连接**”：

   |设置|值|
   |---|---|
   |用户名|**Student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 **az140-cl-vm11a** 的远程桌面会话的“开始”菜单中，导航到 **“Windows 管理工具”** 文件夹，将其展开，并选择 **“Active Directory 用户和计算机”**。
1. 在 **“Active Directory 用户和计算机”** 控制台中，右键单击域节点，选择 **“新建”**，然后选择 **“组织单位”**，在 **“新建对象 - 组织单位”** 对话框中，在 **“名称”** 文本框中，键入 **“ADDC 用户”**，并选择 **“确定”**。
1. 在 **“Active Directory 用户和计算机”** 控制台中，右键单击 **“ADDC 用户”**，选择 **“新建”**，然后选择 **“组”**，在 **“新建对象 - 组”** 对话框中，指定以下设置并选择 **“确定”**：

   |设置|值|
   |---|---|
   |组名称|**本地管理员**|
   |组名称 (pre-Windows 2000)|**本地管理员**|
   |组范围|**全局**|
   |组类型|**安全**|

1. 在 **“Active Directory 用户和计算机”** 控制台中，显示 **“本地管理员”** 组的属性，切换到 **“成员”** 选项卡，选择 **“添加”**，在 **“选择用户、联系人、计算机、服务帐户或组”** 对话框中，在 **“输入要选择的对象名称”** 中，键入 **“aadadmin1”** 并选择 **“确定”**。
1. 在与 **az140-cl-vm11a** 的远程桌面会话的“开始”菜单中，导航到 **“Windows 管理工具”** 文件夹，将其展开，并选择 **“组策略管理”**。
1. 在 **“组策略管理”** 控制台中，导航到 **“AADDC 计算机”** OU，右键单击 **“AADDC 计算机 GPO”** 图标并选择 **“编辑”**。
1. 在 **“组策略管理编辑器”** 控制台中，依次展开 **“计算机配置”**、 **“策略”**、 **“Windows 设置”** 和 **“安全性设置”**，右键单击 **“受限制的组”** 并选择 **“添加组”**。
1. 在 **“添加组”** 对话框中，在 **“组”** 文本框中，选择 **“浏览”**，在 **“选择组”** 对话框中，在 **“输入要选择的对象名称”** 中，键入 **“本地管理员”** 并选择 **“确定”**。
1. 回到 **“添加组”** 对话框，选择 **“确定”**。
1. 在 **“ADATUM\本地管理员属性”** 对话框中，在标记为 **“此组是以下成员”** 的部分中，选择 **“添加”**，在 **“组成员身份”** 对话框中，键入 **“管理员”**，选择 **“确定”**，并再次选择 **“确定”** 以完成更改。

   >**备注**：确保使用标记为“**此组是其成员**”的部分

1. 在与 az140-cl-vm11a Azure VM 的远程桌面中，以管理员身份启动 PowerShell ISE，并运行以下命令，重启两个 Azure 虚拟桌面主机，以触发组策略处理：

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. 等待脚本完成。这需要约 3 分钟时间。

#### 任务 2：在 Azure 虚拟桌面会话主机 VM 上配置基于 FSLogix 的配置文件

1. 在与 **az140-cl-vm11a** 的远程桌面会话中，启动与 **az140-21-p1-0** 的远程桌面会话，并在出现提示时，使用 **ADATUM\wvdaadmin1** 用户名称和创建该用户帐户时设置的密码登录。 
1. 在与 **az140-21-p1-0** 的远程桌面会话中，启动 Microsoft Edge，浏览到 [FSLogix 下载页](https://aka.ms/fslogix_download)，下载 FSLogix 压缩安装二进制文件，将其解压缩到 **C:\\Source** 文件夹中，导航到 **x64\\Release** 子文件夹并使用 **FSLogixAppsSetup.exe** 以默认设置安装 Microsoft FSLogix 应用。

   > **备注**：可能不需要安装 FXLogic，具体取决于映像是否已包含它。需要重启才可完成 FX Logix 安装。

1. 在与 **az140-21-p1-0** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**，并从 **“管理员: Windows PowerShell ISE”** 脚本窗格运行以下脚本，以安装最新版本的 PowerShellGet 模块（如果提示确认，请选择 **“是”**）：

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以安装最新版本的 Az PowerShell 模块（如果提示确认，请选择 **“全部确认”**）：

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以修改执行策略：

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台运行以下命令，以登录 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请使用在具有本实验室所用订阅的所有者角色的用户帐户的 Azure AD 凭据登录。
1. 从 **“管理员: Windows PowerShell ISE”** 脚本窗格运行以下脚本，以检索之前在本实验室中配置的 Azure 存储帐户的名称：

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. 在与 **az140-21-p1-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格运行以下脚本，以配置配置文件注册表设置：

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. 在与 **az140-21-p1-0** 的远程桌面会话中，右键单击 **“开始”**，在右键单击菜单中，选择 **“运行”**，在 **“运行”** 对话框的 **“打开”** 文本框中，键入以下命令并选择 **“确定”**，以启动 **“本地用户和组”** 窗口：

   ```cmd
   lusrmgr.msc
   ```

1. 在 **“本地用户和组”** 控制台中，注意名称以 **“FSLogix”** 字符串开头的四个组：

   - FSLogix ODFC Exclude List
   - FSLogix ODFC Include List
   - FSLogix Profile Exclude List
   - FSLogix Profile Include List

1. 在 **“本地用户和组”** 控制台中，双击 **“FSLogix Profile Include List”** 组条目，注意它包括 **“\\Everyone”** 组， 
选择 **“确定”** 以关闭组 **“属性”** 窗口。 
1. 在 **“本地用户和组”** 控制台中，双击 **“FSLogix Profile Exclude List”** 组条目，注意它默认不包括任何组成员，并选择 **“确定”** 关闭组 **“属性”** 窗口。 

   > **备注**：为了提供一致的用户体验，需要在所有 Azure 虚拟桌面会话主机上安装和配置 FSLogix 组件。你将在我们实验室环境中的另一个会话主机上以无人看管的方式执行此任务。 

1. 在与 **az140-21-p1-0** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格运行以下脚本，以在 **“az140-21-p1-1”** 会话主机上安装 FSLogix 组件：

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. 在与 **az140-21-p1-0** 的远程桌面会话中，启动 **Windows PowerShell ISE** 作为管理员，并从 **“管理员: Windows PowerShell ISE”** 脚本窗格运行以下脚本，以在 **“az140-21-p1-1”** 会话主机上配置配置文件注册表设置：

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **备注**：在测试基于 FSLogix 的配置文件功能之前，需要从上一个实验室中使用的 Azure 虚拟桌面会话主机中删除将用于测试的 ADATUM\wvdaadmin1 帐户的本地缓存配置文件。

1. 切换到与 **az140-cl-vm11a** 的远程桌面会话，在与 **az140-cl-vm11a** 的远程桌面会话中，切换到 **“管理员: Windows PowerShell ISE”** 窗口，然后从 **“管理员: Windows PowerShell ISE”** 脚本窗格中，运行以下命令，删除 ADATUM\aaduser1 帐户的本地缓存配置文件：

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### 任务 3：在 Azure 虚拟桌面中测试基于 FSLogix 的配置文件

1. 在与 **az140-cl-vm11a** 的远程桌面会话中，切换到远程桌面客户端。
1. 在与 **“az140-cl-vm11a”** 的远程桌面会话中，在 **“远程桌面”** 客户端窗口中，在应用程序列表中，双击 **“命令提示符”**，出现提示时，提供密码，并验证它是否启动 **“命令提示符”** 窗口。 

   > **备注**：最开始应用程序可能需要几分钟才能启动，但之后启动速度应该会快得多。

1. 在 **“命令提示符”** 窗口的左上角，右键单击 **“命令提示符”** 图标，然后在下拉菜单中选择 **“属性”**。
1. 在 **“命令提示符属性”** 对话框中，选择 **“字体”** 选项卡，修改大小和字体设置，然后选择 **“确定”**。
1. 在 **“命令提示符”** 窗口中，键入 **“logoff”** 并按 **Enter** 键退出远程桌面会话。
1. 在与 **az140-cl-vm11a** 的远程桌面会话中，在 **“远程桌面”** 客户端窗口的应用程序列表中，双击 **“SessionDesktop”**，并验证它是否启动了远程桌面会话。 
1. 在 **“SessionDesktop”** 会话中，右键单击 **“开始”**，在右键单击菜单中，选择 **“运行”**，在 **“运行”** 对话框的 **“打开”** 文本框中，键入 **“cmd”** 并选择 **“确定”**，以启动 **“命令提示符”** 窗口：
1. 验证 **“命令提示符”** 窗口属性是否与之前在此任务中设置的属性匹配。
1. 在 **“SessionDesktop”** 会话中，最小化所有窗口，右键单击桌面，在右键单击菜单中选择 **“新建”**，然后在级联菜单中选择 **“快捷方式”**。 
1. 在 **“创建快捷方式”** 向导的 **“要为哪项创建快捷方式?”** 页面的 **“键入项的位置”** 文本框中，键入 **“记事本”** 并选择 **“下一步”**。
1. 在 **“创建快捷方式”** 向导的 **“为快捷方式命名”** 页面上，在 **“为此快捷方式键入名称”** 文本框中键入 **“记事本”** 并选择 **“完成”**。
1. 在 **“SessionDesktop”** 会话中，右键单击 **“开始”**，在右键单击菜单中，选择 **“关闭或注销”**，然后在级联菜单中选择 **“注销”**。
1. 返回与 **az140-cl-vm11a** 的远程桌面会话，在 **“远程桌面”** 客户端窗口的应用程序列表中，双击 **“SessionDesktop”** 以启动新的远程桌面会话。 
1. 在 **“SessionDesktop”** 会话中，验证桌面是否显示了 **“记事本”** 快捷方式。
1. 在 **“SessionDesktop”** 会话中，右键单击 **“开始”**，在右键单击菜单中，选择 **“关闭或注销”**，然后在级联菜单中选择 **“注销”**。
1. 切换到与 **“az140-cl-vm11a”** 的远程桌面会话，切换到显示 Azure 门户的 Microsoft Edge 窗口。
1. 在显示 Azure 门户的 Microsoft Edge 窗口中，导航回 **“存储帐户”** 边栏选项卡，并选择表示您前面的练习中创建的存储帐户的条目。
1. 在 **“存储帐户”** 边栏选项卡上的 **“文件服务”** 部分，选择 **“文件共享”**，然后在文件共享列表中选择 **“az140-22a-profiles”**。 
1. 在 **“az140-22a-profiles”** 边栏选项卡上，验证其内容是否包含一个文件夹，该文件夹名称由 **“ADATUM\aduser1”** 帐户的安全标识符 (SID) 和 **“_aaduser1”** 后缀组成。
1. 选择你在上一步中标识的文件夹，注意它包含一个名为 **“Profile_aduser1.vhd”** 的文件。

#### 任务 4：删除 Azure 实验室资源

1. 按照[使用 Azure 门户删除 Azure Active Directory 域服务托管域](https://docs.microsoft.com/zh-cn/azure/active-directory-domain-services/delete-aadds)中的说明删除 Azure AD DS 部署。
1. 按照 [Azure 资源管理器资源组和资源删除](https://docs.microsoft.com/zh-cn/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal) 中的说明删除在本课程的 Azure AD DS 实验室中预配的所有 Azure 资源组。
