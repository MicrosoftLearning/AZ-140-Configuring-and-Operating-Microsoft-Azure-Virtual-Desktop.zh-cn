---
lab:
  title: 实验室：实施和管理 Azure 虚拟桌面配置文件 (AD DS)
  module: 'Module 4: Manage User Environments and Apps'
---

# 实验室 - 实施和管理 Azure 虚拟桌面配置文件 (AD DS)
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- Microsoft 帐户或 Microsoft Entra 帐户，该帐户在本实验室将会使用的 Azure 订阅中具有所有者或参与者角色，在与该 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色。
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”
- 已完成实验室**实现和管理 AVD 存储 (AD DS)**

## 预计用时

30 分钟

## 实验室方案

你需要在 Active Directory 域服务 (AD DS) 环境中实施 Azure 虚拟桌面配置文件管理。

## 目标
  
完成本实验室后，你将能够：

- 为 Azure 虚拟桌面实现基于 FSLogix 的配置文件

## 实验室文件

- 无

## 说明

### 练习 1：为 Azure 虚拟桌面实施基于 FSLogix 的配置文件

此练习的主要任务如下：

1. 在 Azure 虚拟桌面会话主机 VM 上配置基于 FSLogix 的配置文件
1. 使用 Azure 虚拟桌面测试基于 FSLogix 的配置文件

#### 任务 1：在 Azure 虚拟桌面会话主机 VM 上配置基于 FSLogix 的配置文件

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，通过选择搜索文本框右侧的“工具栏”图标，打开“Cloud Shell”窗格。
1. 在“Cloud Shell”窗格中，运行以下命令，以启动你将在本实验室中使用的 Azure 虚拟桌面会话主机 Azure VM****：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**备注**：等到 Azure VM 开始运行后再继续执行下一步操作。

1. 在“Cloud Shell”窗格中，运行以下命令，以在会话主机上启用 PowerShell 远程处理****。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```

1. 关闭 Cloud Shell
1. 在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az140-21-p1-0”************。
1. 在“az140-21-p1-0”边栏选项卡上，选择“连接”，在下拉菜单中选择“通过 Bastion 进行连接”************。
1. 系统出现提示时，请使用以下凭据登录：

   |设置|值|
   |---|---|
   |用户名|**student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-21-p1-0**** 的 Bastion 会话中，启动 Microsoft Edge，浏览到 FSLogix 下载页[](https://aka.ms/fslogix_download)，下载 FSLogix 压缩安装二进制文件，将其解压到 C:\\Allfiles\\Labs\\04**** 文件夹（如果需要，请创建该文件夹），导航到“x64\\Release”**** 子文件夹，双击“FSLogixAppsSetup.exe”**** 文件以启动 Microsoft FSLogix 应用安装**** 向导，然后使用默认设置逐步安装 Microsoft FSLogix 应用。

   > **注意**：如果映像已包含 FSLogix，则无需进行安装。

1. 在与**az140-21-p1-0**的 Bastion 会话中，以管理员身份启动**Windows PowerShell ISE**。
1. 在“管理员:**Windows PowerShell ISE**”控制台中运行以下命令，以安装最新版本的 Az PowerShell 模块（提示安装和导入 NuGet 时输入**Y**）：

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck -Force
   ```

   > **注意**：可能需要等待 3-5 分钟，然后才会显示 Az 模块安装的任何输出。 在输出停止后，可能需要再等待 5 分钟****。 这是预期的行为。

1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以登录 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请使用在本实验室所用订阅中具有所有者角色的用户帐户的 Microsoft Entra 凭据登录。
1. 在与 az140-21-p1-0**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以检索之前在本实验室中配置的 Azure 存储帐户的名称：

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. 在与 az140-21-p1-0**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以配置配置文件注册表设置：

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. 在与 az140-21-p1-0**** 的 Bastion 会话中，右键单击“开始”****，在右键单击菜单中，选择“运行”****，在“运行”**** 对话框的“打开”**** 文本框中，键入以下内容，然后选择“确定”**** 以启动“本地用户和组”**** 控制台：

   ```cmd
   lusrmgr.msc
   ```

1. 在“本地用户和组”**** 控制台中，记下四个以 FSLogix**** 字符串开头的组名称：

   - FSLogix ODFC 排除列表
   - FSLogix ODFC 包括列表
   - FSLogix 配置文件排除列表
   - FSLogix 配置文件包括列表

1. 在“本地用户和组”控制台的组列表中，双击“FSLogix Profile Include List”组，注意它包括“\\Everyone”组，然后选择“确定”关闭组“属性”窗口********************。 
1. 在“本地用户和组”**** 控制台的组列表中，双击“FSLogix 配置文件排除列表”**** 组，注意默认情况下这不包括任何组成员组，然后选择“确定”**** 关闭组“属性”**** 窗口。 

   > **备注**：若要提供一致的用户体验，需要在所有 Azure 虚拟桌面会话主机上安装和配置 FSLogix 组件。 你将在实验室环境中其他会话主机上以无人参与的方式执行此任务。 

   > **注意**：如果已在会话主机上安装 FSLogix，则不需要执行以下步骤。

1. 在与 az140-21-p1-0**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以在 az140-21-p1-1**** 和az140-21-p1-2**** 会话主机上安装 FSLogix 组件：

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **备注**：请等待脚本执行完成。 这可能需要大约 2 分钟。

1. 在与 az140-21-p1-0**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以在 az140-21-p1-1**** 和 az140-21-p1-1**** 会话主机上配置配置文件注册表设置：

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **备注**：在测试基于 FSLogix 的配置文件功能之前，需要从上一个实验室中使用的 Azure 虚拟桌面会话主机中删除将用于测试的 ADATUM\\aduser1**** 帐户的本地缓存配置文件。

1. 在与 az140-21-p1-0**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格中运行以下命令，以删除用作会话主机的所有 Azure VM 上 ADATUM\\aduser1**** 帐户的本地缓存配置文件：

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

1. 在与**az140-21-p1-0**的 Bastion 会话中，右键单击“启动”，在右键菜单中选择“关闭或退出登录”，然后在级联菜单中选择“退出登录”************。
1. 在“断开连接”窗口中，选择“关闭”********。

#### 任务 2：使用 Azure 虚拟桌面测试基于 FSLogix 的配置文件

1. 切换到你的实验室计算机，在该实验室计算机的显示 Azure 门户的浏览器窗口中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡上，选择“az140-cl-vm11”条目  。
1. 在“az140-cl-vm11”边栏选项卡上，选择“连接”，在下拉菜单中选择“通过 Bastion 进行连接”************。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**Student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-cl-vm11**** 的 Bastion 会话中，单击“开始”****，然后在“开始”**** 菜单中单击“远程桌面”**** 以启动远程桌面客户端。
1. 在与 az140-cl-vm11**** 的 Bastion 会话中，在“远程桌面”**** 客户端窗口中，选择“订阅”****，在出现提示时使用 aduser1**** 的凭据登录。

   >注意：如果系统未要求你订阅，则可能需要取消之前的订阅****。

1. 在应用程序列表中，双击命令提示符，出现提示时，提供 aduser1 帐户的密码，并验证“命令提示符”窗口是否成功打开************。
1. 在“命令提示符”**** 窗口的左上角，右键单击“命令提示符”**** 图标，然后在下拉菜单中选择“属性”****。
1. 在“命令提示符属性”**** 对话框中，选择“字体”**** 选项卡，修改大小和字体设置，然后选择“确定”****。
1. 在“命令提示符”**** 窗口中，键入“logoff”****，然后按 Enter**** 键从远程桌面会话注销。
1. 在与 az140-cl-vm11**** 的 Bastion 会话中，在“远程桌面”**** 客户端窗口的应用程序列表中，双击 az140-21-ws1 下的“SessionDesktop”****，并验证是否启动了远程桌面会话。 
1. 在 SessionDesktop**** 会话中，右键单击“开始”****，在右键单击菜单中，选择“运行”****，在“运行”**** 对话框的“打开”**** 文本框中，键入“cmd”****，然后选择“确定”**** 以启动“命令提示符”**** 窗口：
1. 验证“命令提示符”**** 窗口设置是否与之前在此任务中配置的设置匹配。
1. 在 SessionDesktop**** 会话中，最小化所有窗口，右键单击桌面，在右键单击菜单中，选择“新建”****，然后在级联菜单中选择“快捷方式”****。 
1. 在“创建快捷方式”**** 向导的“想为哪个项目创建快捷方式?”**** 页的“请键入项目的位置”**** 文本框中，键入“Notepad”**** 并选择“下一步”****。
1. 在“创建快捷方式”**** 向导的“想将快捷方式命名为什么”**** 页中，在“键入该快捷方式的名称”**** 文本框中键入“Notepad”**** 并选择“完成”****。
1. 在 SessionDesktop**** 会话中，右键单击“开始”****，在右键单击菜单中，选择“关闭或注销”****，然后在级联菜单中，选择“注销”****。
1. 在与 az140-cl-vm11**** 的 Bastion 会话中，在“远程桌面”**** 客户端窗口的应用程序列表中，双击“SessionDesktop”****，以启动新的远程桌面会话。 
1. 在 SessionDesktop**** 会话中，验证记事本**** 快捷方式是否显示在桌面上。
1. 在 SessionDesktop**** 会话中，右键单击“开始”****，在右键单击菜单中，选择“关闭或注销”****，然后在级联菜单中，选择“注销”****。
1. 切换到实验室计算机，然后在显示 Azure 门户的 Microsoft Edge 窗口中，导航到“存储帐户”**** 边栏选项卡，然后选择表示在上一练习中创建的存储帐户的条目。
1. 在“存储帐户”边栏选项卡上的“文件服务”**** 部分中，选择“文件共享”****，然后在文件共享列表中，选择“az140-22-profiles”****。 
1. 在“az140-22-profiles”**** 边栏选项卡上，选择“浏览”**** 并验证其内容是否包含一个文件夹，该文件夹名称由 ADATUM\\aduser1**** 帐户的安全标识符 (SID) 和“_aduser1”**** 后缀组成。
1. 选择在上一步中标识的文件夹，注意其中包含了名为 Profile_aduser1.vhd**** 的单个文件。

### 练习 2：停止并解除分配在实验室中预配和使用的 Azure VM

此练习的主要任务如下：

1. 停止并解除分配在实验室中预配和使用的 Azure VM

>**备注**：在此练习中，你将解除分配此实验室中预配和使用的 Azure VM，以最大程度减少相应的计算费用

#### 任务 1：解除分配在实验室中预配和使用的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开 Cloud Shell 窗格内的“PowerShell”shell 会话 。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建和使用的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注意**：该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能停止和解除分配 Azure VM。
