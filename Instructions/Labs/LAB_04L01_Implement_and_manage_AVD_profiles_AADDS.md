---
lab:
  title: 实验室：实施和管理 Azure 虚拟桌面配置文件 (Microsoft Entra DS)
  module: 'Module 4: Manage User Environments and Apps'
---

# 实验室 - 实施和管理 Azure 虚拟桌面配置文件 (Microsoft Entra DS)
# 学生实验室手册

## 实验室依赖项

- Azure 订阅
- Microsoft 帐户或 Microsoft Entra 帐户，在与 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色并在 Azure 订阅中具有所有者或参与者角色
- 在实验室 **Azure 虚拟桌面简介 (Microsoft Entra DS)** 中预配的 Azure 虚拟桌面环境

## 预计用时

30 分钟

## 实验室方案

需要在 Microsoft Entra DS 环境中实施 Azure 虚拟桌面配置文件管理。

## 目标
  
完成本实验室后，你将能够：

- 配置 Azure 文件存储，以便存储 Microsoft Entra DS 环境中的 Azure 虚拟桌面的配置文件容器
- 在 Microsoft Entra DS 环境中为 Azure 虚拟桌面实施基于 FSLogix 的配置文件

## 实验室文件

- 无

## 说明

### 练习 1：为 Azure 虚拟桌面实施基于 FSLogix 的配置文件

此练习的主要任务如下：

1. 在 Azure 虚拟桌面会话主机 VM 上配置本地管理员组
1. 在 Azure 虚拟桌面会话主机 VM 上配置基于 FSLogix 的配置文件
1. 使用 Azure 虚拟桌面测试基于 FSLogix 的配置文件
1. 删除 Azure 实验室资源

#### 任务 1：在 Azure 虚拟桌面会话主机 VM 上配置本地管理员组

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”**** 窗格。
1. 如果系统提示选择 Bash 或 PowerShell，请选择 PowerShell  。 

   >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。 

1. 从“Cloud Shell”**** 窗格中的 PowerShell 会话运行以下命令，以启动将在本实验室中使用的 Azure 虚拟桌面会话主机 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Start-AzVM
   ```

   >**备注**：等到 Azure VM 开始运行后再继续执行下一步操作。
   
      
1. 从“Cloud Shell”**** 窗格中的 PowerShell 会话运行以下命令，以在会话主机上启用 PowerShell 远程处理。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Enable-AzVMPSRemoting
   ```
   
1. 关闭 Cloud Shell
1. 在实验室计算机上，从 Azure 门户中搜索并选择“虚拟机”****，然后从“虚拟机”**** 边栏选项卡中，选择“az140-cl-vm11a”**** 条目。 这将打开“az140-cl-vm11a”**** 边栏选项卡。
1. 在“az140-cl-vm11a”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-cl-vm11a \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”************************。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**aadadmin1@adatum.com**|
   |密码|之前配置的密码|

1. 在与 az140-cl-vm11a**** 的 Bastion 会话中，在“开始”菜单中，导航到“Windows 管理工具”**** 文件夹，展开它，然后选择“Active Directory 用户和计算机”****。
1. 在“Active Directory 用户和计算机”**** 控制台中，右键单击域节点，选择“新建”****，然后选择“组织单位”****，在“新建对象 - 组织单位”**** 对话框的“名称”**** 文本框中，键入“ADDC 用户”****，然后选择“确定”****。
1. 在“Active Directory 用户和计算机”**** 控制台中，右键单击“ADDC 用户”****，选择“新建”****，然后选择“组”****，在“新建对象 - 组”**** 对话框中，指定以下设置，然后选择“确定”****。

   |设置|值|
   |---|---|
   |组名称|本地管理员****|
   |组名称（Windows 2000 以前的版本）|本地管理员****|
   |团范围|**全球**|
   |组类型|**安全性**|

1. 在“Active Directory 用户和计算机”控制台中，显示“本地管理员”组的属性，切换到“成员”选项卡，选择“添加”，在“选择用户、联系人、计算机、服务帐户或组”对话框中，在“输入要选择的对象名称”中，键入“aadadmin1;wvdaadmin1”并选择“确定”********************************。
1. 在与 az140-cl-vm11a**** 的 Bastion 会话中，在“开始”菜单中，导航到“Windows 管理工具”**** 文件夹，将其展开，然后选择“组策略管理”****。
1. 在“组策略管理”**** 控制台中，导航到“AADDC 计算机”**** OU，右键单击“AADDC 计算机 GPO”**** 图标，然后选择“编辑”****。
1. 在“组策略管理编辑器”**** 控制台中，展开“计算机配置”****、“策略”****、“Windows 设置”****、“安全设置”****，右键单击“受限制的组”****，然后选择“添加组”****。
1. 在“添加组”**** 对话框中的“组”**** 文本框中，选择“浏览”****，在“选择组”**** 对话框中的“输入要选择的对象名称”**** 中，键入“本地管理员”****，然后选择“确定”****。
1. 返回“添加组”**** 对话框，选择“确定”****。
1. 在“ADATUM\本地管理员属性”**** 对话框中，在标记为“此组属于”**** 的部分中，选择“添加”****，在“组成员资格”**** 对话框中，键入“管理员”****，选择“确定”****，然后再次选择“确定”**** 以完成更改。

   >**备注**：请确保使用标记为“此组属于”**** 的部分

1. 在与名为 az140-cl-vm11a 的 Azure VM 的 Bastion 会话中，以管理员身份启动 PowerShell ISE，并运行以下命令以重启两个 Azure 虚拟桌面主机，从而触发组策略处理：

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. 等待脚本完成。 这大约需要 3 分钟。

#### 任务 2：在 Azure 虚拟桌面会话主机 VM 上配置基于 FSLogix 的配置文件

1. 在与 az140-cl-vm11a**** 的 Bastion 会话中，启动与 az140-21-p1-0**** 的远程桌面会话，在出现提示时使用 ADATUM\wvdaadmin1**** 用户名和在创建此用户帐户时设置的密码登录。 

   > **备注**：如果 RDP 连接无法连接，请使用 Azure 门户通过 Bastion 连接到 VM。

1. 在与 az140-21-p1-0 的远程桌面会话中，启动 Microsoft Edge，浏览到 [FSLogix 下载页](https://aka.ms/fslogix_download)，下载 FSLogix 压缩安装二进制文件，将其解压缩到 C:\\Source 文件夹中，导航到 x64\\Release 子文件夹并使用 FSLogixAppsSetup.exe 以默认设置安装 Microsoft FSLogix 应用****************。

   > **备注**：可能不需要安装 FXLogic，具体取决于映像是否已包含它。 安装 FX Logix 后需要重启。

1. 在与 az140-21-p1-0**** 的远程桌面会话中，以管理员身份启动“Windows PowerShell ISE”****，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以安装最新版本的 PowerShellGet 模块（如果提示确认，请选择“是”****）：

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以安装最新版本的 Az PowerShell 模块（如果提示确认，请选择“全部确认”）：

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. 从“管理员: Windows PowerShell ISE”**** 控制台中运行以下命令，以修改执行策略：

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以登录 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请使用在此实验室中用到的订阅中具有所有者角色的用户帐户的 Microsoft Entra 凭据登录。
1. 在“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下脚本，以检索之前在本实验室中配置的 Azure 存储帐户的名称：

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. 在与 az140-21-p1-0**** 的远程桌面会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下脚本，对配置文件注册表设置进行配置：

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```
   >注意：如果命令产生错误，请继续进行下一步****。
   
1. 在与 az140-21-p1-0**** 的远程桌面会话中，右键单击“开始”****，在右键单击菜单中选择“运行”****，在“运行”**** 对话框中的“打开”**** 文本框中，键入以下内容，然后选择“确定”**** 以启动“本地用户和组”**** 窗口：

   ```cmd
   lusrmgr.msc
   ```

1. 在“本地用户和组”**** 控制台中，记下四个名称以“FSLogix”**** 字符串开头的组：

   - FSLogix ODFC 排除列表
   - FSLogix ODFC 包括列表
   - FSLogix 配置文件排除列表
   - FSLogix 配置文件包括列表

1. 在“本地用户和组”控制台中，双击“FSLogix Profile Include List”组，注意它包括“\\Everyone”组，然后选择“确定”关闭组“属性”窗口********************。 
1. 在“本地用户和组”**** 控制台中，双击“FSLogix 配置文件排除列表”**** 组条目，注意它默认不包括任何组成员，然后选择“确定”**** 以关闭组“属性”**** 窗口。 

   > **备注**：若要提供一致的用户体验，需要在所有 Azure 虚拟桌面会话主机上安装和配置 FSLogix 组件。 你将在实验室环境中的其他会话主机上以无人参与的方式执行此任务。 

1. 在与 az140-21-p1-0**** 的远程桌面会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以在 az140-21-p1-1**** 会话主机上安装 FSLogix 组件：

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. 在与 az140-21-p1-0**** 的远程桌面会话中，以管理员身份启动 Windows PowerShell ISE****，并从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以在 az140-21-p1-1**** 会话主机上对配置文件注册表设置进行配置：

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **备注**：在测试基于 FSLogix 的配置文件功能之前，需要从上一个实验室中使用的 Azure 虚拟桌面会话主机中删除将用于测试的 ADATUM\wvdaadmin1 帐户的本地缓存配置文件。

1. 切换到与 az140-cl-vm11a**** 的 Bastion 会话，在与 az140-cl-vm11a**** 的 Bastion 会话中，切换到“管理员: Windows PowerShell ISE”**** 窗口，然后从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以删除 ADATUM\aaduser1 帐户的本地缓存配置文件：

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### 任务 3：使用 Azure 虚拟桌面测试基于 FSLogix 的配置文件

1. 在与 az140-cl-vm11a**** 的 Bastion 会话中，切换到远程桌面客户端。
1. 在与 az140-cl-vm11a**** 的 Bastion 会话中，在“远程桌面”**** 客户端窗口的应用程序列表中，双击“命令提示符”****，在出现提示时提供密码，并验证其是否启动了“命令提示符”**** 窗口   。 

   > **注意**：最开始应用程序可能需要几分钟才能启动，但之后启动速度应该会快得多。

1. 在“命令提示符”**** 窗口的左上角，右键单击命令提示符**** 图标，然后在下拉菜单中选择“属性”****。
1. 在“命令提示符属性”**** 对话框中，选择“字体”**** 选项卡，修改大小和字体设置，然后选择“确定”****。
1. 在“命令提示符”**** 窗口中，键入“logoff”**** ，然后按 Enter**** 键从远程桌面会话注销。
1. 在与 az140-cl-vm11a**** 的 Bastion 会话中，在“远程桌面”**** 客户端窗口的应用程序列表中，双击“SessionDesktop”****，然后验证其是否启动了远程桌面会话。 
1. 在 SessionDesktop**** 会话中，右键单击“开始”****，在右键单击菜单中，选择“运行”****，在“运行”**** 对话框中的“打开”**** 文本框中，键入“cmd”****，然后选择“确定”**** 以启动“命令提示符”**** 窗口：
1. 验证“命令提示符”**** 窗口属性是否与之前在此任务中设置的属性匹配。
1. 在 SessionDesktop**** 会话中，最小化所有窗口，右键单击桌面，在右键单击菜单中，选择“新建”****，然后在级联菜单中选择“快捷方式”****。 
1. 在“创建快捷方式”**** 向导的“想为哪个项目创建快捷方式?”**** 页的“请键入项目的位置”**** 文本框中，键入“Notepad”**** 并选择“下一步”****。
1. 在“创建快捷方式”**** 向导的“想将快捷方式命名为什么”**** 页中，在“键入该快捷方式的名称”**** 文本框中键入“Notepad”**** 并选择“完成”****。
1. 在 SessionDesktop**** 会话中，右键单击“开始”****，在右键单击菜单中，选择“关闭或注销”****，然后在级联菜单中，选择“注销”****。
1. 返回与 az140-cl-vm11a**** 的 Bastion 会话，在“远程桌面”**** 客户端窗口的应用程序列表中，双击“SessionDesktop”****，以启动新的远程桌面会话  。 
1. 在 SessionDesktop**** 会话中，验证“Notepad”**** 快捷方式是否显示在桌面上。
1. 在 SessionDesktop**** 会话中，右键单击“开始”****，在右键单击菜单中，选择“关闭或注销”****，然后在级联菜单中，选择“注销”****。
1. 切换到与 az140-cl-vm11a**** 的 Bastion 会话，切换到显示 Azure 门户的 Microsoft Edge 窗口。
1. 在显示 Azure 门户的 Microsoft Edge 窗口中，导航回“存储帐户”**** 边栏选项卡，然后选择表示在上一练习中创建的存储帐户的条目。
1. 在存储帐户边栏选项卡的“文件服务”**** 部分中，选择“文件共享”****，然后在文件共享列表中选择 az140-22a-profiles****。 
1. 在“az140-22a-profiles”**** 边栏选项卡上，选择“浏览”**** 并验证其内容是否包含一个文件夹，该文件夹名称由“ADATUM\\aaduser1”**** 帐户的安全标识符 (SID) 和“_aaduser1”**** 后缀组成。
1. 选择在上一步中辨认的文件夹，并注意它仅包含一个名为 Profile_aaduser1.vhd**** 的文件。

### 练习 2：删除 Azure 实验室资源（可选）

1. 按照[使用 Azure 门户删除 Azure Active Directory 域服务托管域]( https://docs.microsoft.com/en-us/azure/active-directory-domain-services/delete-aadds)中所述的说明删除 Microsoft Entra DS 部署。
1. 按照 [Azure 资源管理器资源组和资源删除](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal) 中所述的说明，删除本课程的 Microsoft Entra DS 实验室中预配的所有 Azure 资源组。
