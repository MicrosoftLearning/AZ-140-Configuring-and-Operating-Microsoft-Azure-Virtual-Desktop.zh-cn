---
lab:
    title: '实验室：打包 Azure 虚拟桌面应用程序 (AD DS)'
    module: '模块 4：管理用户环境和应用'
---

# 实验室 - 打包 Azure 虚拟桌面应用程序 (AD DS)
# 学生实验室手册

## 实验室依赖项

- Azure 订阅
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有与 Azure 订阅关联的 Azure AD 租户的全局管理员角色，以及 Azure 订阅的所有者角色或参与者角色
- 已完成实验室 **“准备部署 Azure 虚拟桌面 (AD DS)”**
- 已完成实验室 **“Azure 虚拟桌面配置文件管理 (AD DS)”**
- 已完成实验室“**为 WVD (AD DS) 配置条件访问策略**”

## 预计用时

60 分钟

## 实验室场景

你需要在 Active Directory 域服务 (AD DS) 环境中打包和部署 Azure 虚拟桌面应用程序。

## 目标
  
完成本实验室后，你将能够：

- 准备和创建 MSIX 应用包
- 在 Azure AD DS 环境中实现 Azure 虚拟桌面的 MSIX 应用附加映像
- 在 AD DS 环境中实现 Azure 虚拟桌面的 MSIX 应用附加

## 实验室文件

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## 说明

### 练习 1：准备和创建 MSIX 应用包

本练习的主要任务如下：

1. 准备配置 Azure 虚拟桌面会话主机
1. 使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM
1. 为 MSIX 打包准备运行 Windows 10 的 Azure VM
1. 生成签名证书
1. 下载要打包的软件
1. 安装 MSIX 打包工具
1. 创建 MSIX 包

#### 任务 1：准备配置 Azure 虚拟桌面会话主机

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，打开 **“Cloud Shell”** 窗格内的 **“PowerShell”** shell 会话。
1. 从“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，以启动你将在本实验室中使用的 Azure 虚拟桌面会话主机 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**备注**： 该命令异步执行（由 -NoWait 参数确定），因此尽管可立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上要花几分钟才能启动 Azure VM。 

   >**备注**： 无需等待 Azure VM 启动，直接继续执行下一个任务。

#### 任务 2：使用 Azure 资源管理器快速启动模板部署运行 Windows 10 的 Azure VM

1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，在“Cloud Shell”窗格的工具栏中选择 **“上传/下载文件”** 图标，在下拉菜单中选择 **“上传”**，然后将文件 **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** 和 **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json** 上传到 Cloud Shell 主目录中。
1. 从“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，部署运行 Windows 10 的 Azure Vm，你将使用该 VM 创建 MSIX 包并将其加入 Azure AD DS 域：

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **备注**： 等待部署完成后，再继续执行下一个任务。该过程大约需要 10 分钟。 

#### 任务 3：为 MSIX 打包准备运行 Windows 10 的 Azure VM

1. 在实验室计算机上，在 Azure 门户中搜索并选择 **“虚拟机”**，然后从 **“虚拟机”** 边栏选项卡的虚拟机列表中选择 **“az140-cl-vm42”** 条目。 **“az140-cl-vm42”** 边栏选项卡随即打开。
1. 在“**az140-cl-vm42**”边栏选项卡上，选择“**连接**”，在下拉菜单中选择“**Bastion**”，在“**az140-cl-vm11 \| 连接**”边栏选项卡的“**Bastion**”选项卡上，选择“**使用 Bastion**”。
1. 出现提示时，使用 **ADATUM\wvdadmin1** 用户名并使用创建此用户帐户时设置的密码登录。 
1. 在与 **az140-cl-vm42** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**，并从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，为 MSIX 打包准备操作系统：

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   ```

   > **备注**：上面的最后一个注册表更改会禁用用户访问控制。这在技术上不是必需的，但可以简化本实验室中说明的过程。

#### 任务 4：生成签名证书

> **备注**：在本实验室中，你将使用自签名证书。在生产环境中，应使用由公共证书颁发机构或内部证书颁发机构颁发的证书，具体取决于预期用途。

1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以生成“公用名称”属性设置为 **“Adatum”** 的自签名证书，并将该证书存储在 **“本地计算机”** 证书存储的 **“个人”** 文件夹中：

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以启动面向本地计算机证书存储的 **“证书”** 控制台：

   ```powershell
   certlm.msc
   ```

1. 在 **“证书”** 控制台窗格中，展开 **“个人”** 文件夹，选择 **“证书”** 子文件夹，右键单击 **“Adatum”** 证书，在右键单击菜单中依次选择 **“所有任务”** 和 **“导出”**。这会启动 **“证书导出向导”**。 
1. 在 **“证书导出向导”** 的 **“欢迎使用证书导出向导”** 页上，选择 **“下一步”**。
1. 在 **“证书导出向导”** 的 **“导出私钥”** 页上，选择选项 **“是，导出私钥”**，然后选择 **“下一步”**。
1. 在 **“证书导出向导”** 的 **“导出文件格式”** 页上，选中 **“导出所有扩展属性”** 复选框，清除 **“启用证书隐私”** 复选框，然后选择 **“下一步”**。
1. 在 **“证书导出向导”** 的 **“安全性”** 页上，选中 **“密码”** 复选框，在下方的文本框中键入 **Pa55w.rd1234**，然后选择 **“下一步”**。
1. 在 **“证书导出向导”** 的 **“要导出的文件”** 页上，在 **“文件名”** 文本框中选择 **“浏览”**，在 **“另存为”** 对话框中导航到 **C:\\Allfiles\\Labs\\04** 文件夹（请先创建该文件夹），在 **“文件名”** 文本框中键入 **adatum.pfx**，然后选择 **“保存”**。
1. 回到 **“证书导出向导”** 的 **“要导出的文件”** 页，确保文本框中包含条目 **“C:\\Allfiles\\Labs\\04\\adatum.pfx”**，然后选择 **“下一步”**。
1. 在 **“证书导出向导”** 的 **“正在完成证书导出向导”** 页上，选择 **“完成”**，然后选择 **“确定”**，确认已成功导出。 

   > **备注**： 由于你将使用自签名证书，因此需要在目标会话主机上的 **“受信任的用户”** 证书存储中安装该证书。

1. 从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以在目标会话主机上的 **“受信任的用户”** 证书存储中安装新生成的证书：

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   $localPath = 'C:\Allfiles\Labs\04'
   ForEach ($wvdhost in $wvdhosts){
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      New-Item -ItemType Directory -Path $remotePath -Force
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### 任务 5：下载要打包的软件

1. 在与 **az140-cl-vm42** 的远程桌面会话中，启动 **Microsoft Edge**，并浏览到 **https://github.com/microsoft/XmlNotepad** 。
1. 在 **microsoft/XmlNotepad** **readme.md** 页上，选择[可下载的独立安装程序](http://www.lovettsoftware.com/downloads/xmlnotepad/xmlnotepadsetup.zip)的下载链接，下载压缩的安装文件。
1. 在与 **az140-cl-vm42** 的远程桌面会话中，启动文件资源管理器，导航到“**下载**”文件夹，打开压缩文件，将压缩文件的文件夹中的内容复制并粘贴到 **C:\\AllFiles\\Labs\\04\\** 目录。 

#### 任务 6：安装 MSIX 打包工具

1. 在与 **az140-cl-vm42** 的远程桌面会话中，启动 **Microsoft Store** 应用。
1. 在 **Microsoft Store** 应用中，搜索并选择 **“MSIX 打包工具”**，然后在 **“MSIX 打包工具”** 页上选择 **“获取”**。
1. 在出现提示时，跳过登录，等待安装完成，选择 **“启动”**，然后在 **“发送诊断数据”** 对话框中选择 **“拒绝”**。 

#### 任务 7：创建 MSIX 包

1. 在与 **az140-cl-vm42** 的远程桌面会话中，切换到 **“管理员: Windows PowerShell ISE”** 窗口，然后从 **“管理员: Windows PowerShell ISE”** 脚本窗格中运行以下命令，以禁用 Windows Search 服务

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，创建将托管 MSIX 包的文件夹：

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台中允许以下命令，从提取的安装程序文件中删除 Zone.Identifier 备用数据流，该数据流的值为“3”，指示其是从 Internet 下载的：

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. 在与 **az140-cl-vm42** 的远程桌面会话中，切换到 **“MSIX 打包工具”** 界面，在 **“选择任务”** 页上选择 **“应用程序包 - 创建你的应用包”** 条目。这将启动 **“创建新包”** 向导。
1. 在 **“创建新包”** 向导的 **“选择环境”** 页上，确保已选中 **“在此计算机上创建包”** 选项，选择 **“下一步”**，等待安装 **“MSIX 打包工具驱动程序”**。
1. 在 **“创建新包”** 向导的 **“准备计算机”** 页上查看建议。如果有挂起的重启，请重启操作系统，使用帐户 **ADATUM\wvdadmin1** 重新登录，然后先重启 **MSIX 打包工具**再继续。 

   >**备注**： MSIX 打包工具会暂时禁用 Windows 更新和 Windows Search。在本例中，Windows Search 服务已处于禁用状态。 

1. 在 **“创建新包”** 向导的 **“准备计算机”** 页上，单击 **“下一步”**。
1. 在 **“创建新包”** 向导的 **“选择安装程序”** 页上，在 **“选择要打包的安装程序”** 文本框旁边选择 **“浏览”**，在 **“打开”** 对话框中浏览到 **C:\\AllFiles\\Labs\\04** 文件夹，选择 **“XmlNotepadSetup.msi”**，然后单击 **“打开”**。 
1. 在 **“创建新包”** 向导的 **“选择安装程序”** 页上，在 **“签名首选项”** 下拉列表中选择 **“使用证书 (.pfx) 签名”** 条目，在 **“浏览证书”** 文本框旁边选择 **“浏览”**，在 **“打开”** 对话框中导航到 **C:\\AllFiles\\Labs\\04** 文件夹，选择 **adatum.pfx** 文件，单击 **“打开”**，在 **“密码”** 文本框中键入 **Pa55w.rd1234**，然后选择 **“下一步”**。
1. 在 **“创建新包”** 向导的 **“包信息”** 页上，查看包信息，验证发布者名称已设置为 **“CN=Adatum”**，然后选择 **“下一步”**。这将触发安装下载的软件。
1. 在 **“XMLNotepad 设置”** 窗口中，接受许可证协议中的条款并选择 **“安装”**，在安装完成时，选中 **“启动 XML Notepad”** 复选框并选择 **“完成”**。
1. 出现提示时，在 **“XML Notepad Analytics”** 窗口中，选择 **“否”**，验证 XML Notepad 是否正在运行并将其关闭，在 **“MSIX 打包工具”** 窗口中切换回 **“创建新包”** 向导，然后选择 **“下一步”**。

   > **备注**：在本例中，不需要重启即可完成安装。

1. 在 **“创建新包”** 向导的 **“首次启动任务”** 页上，查看提供的信息并选择 **“下一步”**。
1. 在系统提示 **“你完成了吗?”** 时，选择 **“是，继续”**。
1. 在 **“创建新包”** 向导的 **“服务报告”** 页上，验证未列出任何服务，然后选择 **“下一步”**。
1. 在 **“创建新包”** 向导的 **“创建包”** 页上，在 **“保存位置”** 文本框中键入 **C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix**，然后单击 **“创建”**。
1. 在 **“已成功创建包”** 对话框中，注意包的保存位置，并选择 **“关闭”**。
1. 切换到“文件资源管理器”窗口，导航到 **C:\\Allfiles\\Labs\\04\\XmlNotepad** 文件夹，并验证其中包含 *.msix 和 *.xml 文件。
1. 将 **XmlNotepad.msix** 文件复制到 **C:\\Allfiles\\Labs\\04** 文件夹。


### 练习 2：在 Azure AD DS 环境中实现 Azure 虚拟桌面的 MSIX 应用附加映像

本练习的主要任务如下：

1. 在运行 Windows 10 企业版的 Azure VM 上启用 Hyper-V
1. 创建 MSIX 应用附加映像

#### 任务 1：在运行 Windows 10 企业版的 Azure VM 上启用 Hyper-V

1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以便为 MSIX 应用附加准备目标 Windows 虚拟桌面主机： 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以在 Azure 虚拟桌面主机上安装 Hyper-V 及其管理工具，包括 Hyper-V PowerShell 模块：

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. 在每个主机上安装 Hyper-V 组件后，键入 **Y** 并按 **Enter** 键，以重启目标操作系统。
1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以在本地计算机上安装 Hyper-V 及其管理工具，包括 Hyper-V PowerShell 模块：

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Hyper-V 组件安装完成后，键入 **Y** 并按 **Enter** 键，以重启操作系统。重启后，使用 **ADATUM\wvdadmin1** 帐户以及创建此用户帐户时设置的密码登录。

#### 任务 2：创建 MSIX 应用附加映像

1. 在与 **az140-cl-vm42** 的远程桌面会话中，启动 **Microsoft Edge**，并浏览到 **https://aka.ms/msixmgr** 。 这会将 **msixmgr.zip** 文件（MSIX mgr 工具存档）自动下载到“**下载**”文件夹。
1. 在文件资源管理器中，导航到 **“下载”** 文件，打开压缩文件，并将 **x64** 文件夹的内容复制到 **C:\\AllFiles\\Labs\\04** 文件夹。 
1. 在与 **az140-cl-vm42** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**，并从 **“管理员: Windows PowerShell ISE”** 脚本窗格中运行以下命令，以创建将用作 MSIX 应用附加映像的 VHD 文件：

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. 从“**管理员: Windows PowerShell ISE**”脚本窗格中运行以下命令，以安装新创建的 VHD 文件：

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. 从“**管理员: Windows PowerShell ISE**”脚本窗格中运行以下命令，以初始化磁盘、创建新分区、格式化该分区，并为该分区分配第一个可用的驱动器号：

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

   > **备注**：如果出现弹出窗口，提示格式化 F: drive，请选择“**取消**”。

1. 从 **“管理员: Windows PowerShell ISE”** 脚本窗格中运行以下命令，以创建将托管 MSIX 文件的文件夹结构，并将你在上一个任务中创建的 MSIX 包解压缩到其中：

   ```powershell
   $appName = 'XmlNotepad'
   New-Item -ItemType Directory -Path "$($partition.DriveLetter):\Apps" -Force
   Set-Location -Path 'C:\AllFiles\Labs\04'
   .\msixmgr.exe -Unpack -packagePath .\$appName.msix -destination "$($partition.DriveLetter):\Apps" -applyacls
   ```

1. 在与 **az140-cl-vm42** 的远程桌面会话中，在文件资源管理器中导航到 **F:\\Apps** 文件夹并查看其内容。如果系统提示获取对该文件夹的访问权限，请选择“**继续**”。
1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以卸载将用作 MSIX 映像的 VHD 文件：

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### 练习 3：在 Azure 虚拟桌面上实现 MSIX 应用附加

本练习的主要任务如下：

1. 配置包含 Azure 虚拟桌面主机的 Active Directory 组
1. 为 MSIX 应用附加设置 Azure 文件存储共享
1. 在 Azure 虚拟桌面会话主机上装载和注册 MSIX 应用附加映像
1. 将 MSIX 应用发布到应用程序组
1. 验证 MSIX 应用附加的功能

#### 任务 1：配置包含 Azure 虚拟桌面主机的 Active Directory 组

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择 **“虚拟机”**，然后在 **“虚拟机”** 边栏选项卡中选择 **“az140-dc-vm11”**。
1. 在“**az140-dc-vm11**”边栏选项卡上，选择“**连接**”，在下拉菜单中选择“**Bastion**”，在“**az140-dc-vm11 \| 连接**”边栏选项卡的“**Bastion**”选项卡上，选择“**使用 Bastion**”。
1. 出现提示时，请提供以下凭据并选择“**连接**”：

   |设置|值|
   |---|---|
   |用户名|**Student**|
   |密码|**Pa55w.rd1234**|

1. 在与 **az140-dc-vm11** 的远程桌面会话中，以管理员身份启动 **Windows PowerShell ISE**。
1. 从 **“管理员: Windows PowerShell ISE”** 脚本窗格中运行以下命令，创建将同步到本实验室中所用 Azure AD 租户的 AD DS 组对象：

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **备注**：你将使用此组向 Azure 虚拟桌面主机授予对 **az140-42-msixvhds** 文件共享的权限。

1. 从 **“管理员: Windows PowerShell ISE”** 控制台中运行以下命令，以向你在前一个步骤中创建的组添加成员：

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 脚本窗格中运行以下命令，以重启属于 'az140-hosts-42-p1' 组成员的服务器：

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | Restart-Computer
   ```

   > **备注**：此步骤可确保组成员身份更改生效。 

1. 在与 **az140-dc-vm11** 的远程桌面会话中，在 **“开始”** 菜单中展开 **Azure AD Connect** 文件夹，并选择 **“Azure AD Connect”**。
1. 在 **“Microsoft Azure Active Directory Connect”** 窗口的 **“欢迎使用 Azure AD Connect”** 页上，选择 **“配置”**。
1. 在 **“Microsoft Azure Active Directory Connect”** 窗口的 **“附加任务”** 页上，依次选择 **“自定义同步选项”** 和 **“下一步”**。
1. 在“**Microsoft Azure Active Directory Connect**”窗口的“**连接到 Azure AD**”页上，使用在本任务稍早部分指定的用户帐户“**aadsyncuser**”的用户主体名称和创建该用户帐户时设置的密码进行身份验证。
1. 在 **“Microsoft Azure Active Directory Connect”** 窗口的 **“连接到目录”** 页上，选择 **“下一步”**。
1. 在 **“Microsoft Azure Active Directory Connect”** 窗口的 **“域和 OU 筛选”** 页上，确保已选中 **“同步所选域和 OU”** 选项，展开 **“adatum.com”** 节点，选中 **“WVDInfra”** OU 旁边的复选框（任何其他已选中的复选框保持不变），然后选择 **“下一步”**。
1. 在 **“Microsoft Azure Active Directory Connect”** 窗口的 **“可选功能”** 页上，接受默认设置，然后选择 **“下一步”**。
1. 在 **“Microsoft Azure Active Directory Connect”** 窗口的 **“准备配置”** 页上，确保已选中 **“配置完成后启动同步过程”** 复选框，然后选择 **“配置”**。
1. 查看 **“配置完成”** 页中的信息，然后选择 **“退出”** 关闭 **“Microsoft Azure Active Directory Connect”** 窗口。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，启动 Microsoft Edge，导航到 [Azure 门户](https://portal.azure.com)。如果出现提示，请使用用户帐户的 Azure AD 凭据登录，该帐户应具有与本实验室所用 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中搜索并选择 **“Azure Active Directory”**，以导航到与本实验室中所用 Azure 订阅关联的 Azure AD 租户。
1. 在 “Azure Active Directory” 边栏选项卡左侧垂直菜单中的 **“管理”** 部分，单击 **“组”**。 
1. 在 **“组 | 全部组”** 边栏选项卡的组列表中，选择 **“az140-hosts-42-p1”** 条目。

   > **备注**： 需要刷新页面以显示组。

1. 在 **“az140-hosts-42-p1”** 边栏选项卡左侧垂直菜单中的 **“管理”** 部分中，单击 **“成员”**。
1. 在 **“az140-hosts-42-p1 | 成员”** 边栏选项卡上，验证“直接成员”列表中包含你在本任务的稍早部分添加到组的 Azure 虚拟桌面池的三个主机。

#### 任务 2：为 MSIX 应用附加设置 Azure 文件存储共享

1. 在实验室计算机上，切换回与 **az140-cl-vm42** 的远程桌面会话。
1. 在与 **“az140-cl-vm42”** 的远程桌面会话中，启动 Microsoft Edge，在 InPrivate 模式下导航到 [Azure 门户](https://portal.azure.com)，然后通过提供具有本实验室所用订阅的所有者角色的用户帐户凭据登录。

   > **备注**：请确保使用 Microsoft Edge InPrivate 模式。

1. 在与 **az140-cl-vm42** 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中搜索并选择 **“存储帐户”**，然后在 **“存储帐户”** 边栏选项卡中选择配置为托管用户配置文件的存储帐户。

   > **备注**：实验室的这一部分视实验室 **“Azure 虚拟桌面配置文件管理 (AD DS)”** 或 **“Azure 虚拟桌面配置文件管理 (Azure AD DS)”** 的完成情况而定

   > **备注**：在生产场景中，应考虑使用单独的存储帐户。这需要为该存储帐户配置 Azure AD DS 身份验证，你已经为托管用户配置文件的存储帐户实现了该身份验证。你将使用同一存储帐户，以尽可能避免在各实验室中执行重复的步骤。

1. 在“存储帐户”边栏选项卡的左侧垂直菜单中，选择“**访问控制(IAM)**”。
1. 在存储帐户的“**访问控制(IAM)**”边栏选项卡上，单击“**+ 添加**”，然后在下拉菜单中选择“**添加角色分配**”，
1. 在 **“添加角色分配”** 边栏选项卡上，指定以下设置并选择 **“保存”**：

   |设置|值|
   |---|---|
   |角色|**存储文件数据 SMB 共享提升的参与者**|
   |将访问权限分配到|**用户、组或服务主体**|
   |选择|**az140-wvd-admins**|

   > **备注**： **az140-wvd-admins** 组包含将用于配置共享权限的 **wvdadmin1** 用户帐户。 

1. 重复前两个步骤以配置以下角色分配：

   |设置|值|
   |---|---|
   |角色|**存储文件数据 SMB 共享提升的参与者**|
   |将访问权限分配到|**用户、组或服务主体**|
   |选择|**az140-hosts-42-p1**|

   |设置|值|
   |---|---|
   |角色|**存储文件数据 SMB 共享读取者**|
   |将访问权限分配到|**用户、组或服务主体**|
   |选择|**az140-wvd-users**|

   > **备注**：Azure 虚拟桌面用户和主机至少需要具有对文件共享的读取访问权限。

1. 在“存储帐户”边栏选项卡左侧垂直菜单的“**数据存储**”部分中，选择“**文件共享**”，然后选择“**+ 文件共享**”。
1. 在“**新建文件共享**”边栏选项卡上，指定以下设置并选择“**创建**”（其他设置保留默认值）：

   |设置|值|
   |---|---|
   |名称|**az140-42-msixvhds**|

1. 在显示 Azure 门户的 Microsoft Edge 中，在文件共享列表中选择新创建的文件共享。 

1. 在与 **az140-cl-vm42** 的远程桌面会话中，启动 **“命令提示符”**，在 **“命令提示符”** 窗口中运行以下命令，以将驱动器映射到 **“az140-42-msixvhds”** 共享（将 `<storage-account-name>` 占位符替换为存储帐户的名称），并验证命令已成功完成：

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. 在与 **az140-cl-vm42** 的远程桌面会话中，在 **“命令提示符”** 窗口中运行以下命令，以向会话主机的计算机帐户授予所需的 NTFS 权限：

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**备注**： 你还可以在以 **ADATUM\\wvdadmin1** 身份登录时使用文件资源管理器设置这些权限。 

   >**备注**：接下来你将验证 MSIX 应用附加的功能

1. 在与 **az140-cl-vm42** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 窗口中运行以下命令，将你在上一个练习中创建的 VHD 文件复制到你在本练习前面部分创建的 Azure 文件存储共享中：

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### 任务 3：在 Azure 虚拟桌面会话主机上装载和注册 MSIX 应用附加映像

1. 在与 **az140-cl-vm42** 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中，搜索并选择 **“Azure 虚拟桌面”**，在 **“Azure 虚拟桌面”** 边栏选项卡左侧垂直菜单的 **“管理”** 部分中，选择 **“主机池”**。
1. 在 **“Azure 虚拟桌面 \| 主机池”** 边栏选项卡的主机池列表中，选择 **“az140-21-hp1”** 条目。
1. 在 **“az140-21-hp1 \| 属性”** 边栏选项卡左侧垂直菜单中的 **“管理”** 部分，选择 **“MSIX 包”**。
1. 在 **“az140-21-hp1 \| MSIX 包”** 边栏选项卡上，单击 **“+ 添加”**。
1. 在 **“添加 MSIX 包”** 边栏选项卡上的 **“MSIX 映像路径”** 文本框中，输入 **XmlNotepad.vhd** 文件的路径，格式为 `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds \packages\XmlNotepad.vhd`（将占位符 `<storage-account-name>` 替换为托管 **az140-42-msixvhds** 文件共享的存储帐户的名称），然后单击 **“添加”**。
1. 在 **“添加 MSIX 包”** 边栏选项卡上，指定以下设置并单击 **“添加”**：

   |设置|值|
   |---|---|
   |MSIX 映像路径|**\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\XmlNotepad.vhd**，其中占位符 `<storage-account-name>` 指定了托管 **az140-42-msixvhds** 文件共享的存储账户的名称|
   |MSIX 包|创建包时生成的名称|
   |显示名称|**XML Notepad**|
   |注册类型|**按需**|
   |状态|**活动**|

#### 任务 4：将 MSIX 应用发布到应用程序组

> **备注**：你需要将 MSIX 应用发布到远程应用和桌面应用组。 

1. 在与 **az140-cl-vm42** 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中，搜索并选择 **“Azure 虚拟桌面”**，在 **“Azure 虚拟桌面”** 边栏选项卡左侧垂直菜单的 **“管理”** 部分中，选择 **“应用程序组”**。
1. 在 **“Azure 虚拟桌面 \| 应用程序组”** 边栏选项卡上，选择 **“az140-21-hp1-Utilities-RAG”** 应用程序组条目。
1. 在 **“az140-21-hp1-Utilities-RAG”** 边栏选项卡左侧垂直菜单中的 **“管理”** 部分，选择 **“应用程序”**。 
1. 在 **“az140-21-hp1-Utilities-RAG | 应用程序”** 边栏选项卡上，单击 **“+ 添加”**。
1. 在 **“添加应用程序”** 边栏选项卡上，指定以下设置并选择 **“保存”**：

   |设置|值|
   |---|---|
   |应用程序源|**MSIX 包**|
   |MSIX 包|**代表映像中包含的包的名称**|
   |MSIX 应用程序|**XMLNOTEPAD**|
   |应用程序名称|**XML Notepad**|
   |显示名称|**XML Notepad**|
   |说明|**XML Notepad**|
   |图标路径|**C:\Program Files\WindowsApps\XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8\VFS\ProgramFilesX86\LovettSoftware\XmlNotepad\XmlNotepad.exe**|
   |图标索引|**0**|

1. 导航回到 **“Azure 虚拟桌面 \| 应用程序组”** 边栏选项卡，然后选择 **“az140-21-hp1-DAG”** 应用程序组条目。
1. 在 **“az140-21-hp1-DAG”** 边栏选项卡左侧垂直菜单中的 **“管理”** 部分，选择 **“应用程序”**。 
1. 在 **“az140-21-hp1-DAG | 应用程序”** 边栏选项卡上，单击 **“+ 添加”**。
1. 在 **“添加应用程序”** 边栏选项卡上，指定以下设置并选择 **“保存”**：

   |设置|值|
   |---|---|
   |应用程序源|**MSIX 包**|
   |MSIX 包|代表映像中包含的包的名称|
   |应用程序名称|**XML Notepad**|
   |显示名称|**XML Notepad**|
   |说明|**XML Notepad**|

#### 任务 5：验证 MSIX 应用附加的功能

1. 在与 **az140-cl-vm42** 的远程桌面会话中，启动 Microsoft Edge 并导航到 [Windows 桌面客户端下载页面](https://go.microsoft.com/fwlink/?linkid=2068602)，下载完成后，选择 **“打开文件”** 开始安装。在 **“远程桌面安装”** 向导的 **“安装范围”** 页上，选择 **“为此计算机的所有用户安装”** 选项，然后单击 **“安装”**。 
1. 安装完成后，请确保 **“安装完成时启动远程桌面”** 复选框处于选中状态，然后单击 **“完成”** 以启动“远程桌面客户端”。
1. 在“**远程桌面**”客户端窗口中，选择“**订阅**”，并在出现提示时使用 **aduser1** 用户主体名称和创建该用户帐户时设置的密码登录。 
1. 出现提示时，在 **“保持登录到所有应用”** 窗口中清除复选框 **“允许组织管理我的设备”**，然后单击 **“否，仅登录到此应用”**。
1. 在 **“远程桌面”** 客户端窗口的 **“az140-21-ws1”** 部分中，双击 **“XML Notepad”** 图标，在出现提示时提供密码，然后验证 XML Notepad 是否成功启动。


### 练习 4：停止并解除分配在实验室中预配和使用的 Azure VM

本练习的主要任务如下：

1. 停止并解除分配在实验室中预配和使用的 Azure VM

>**备注**：在本练习中，你将解除分配在本实验室中预配和使用的 Azure VM，以最大程度减少相应的计算费用

#### 任务 1：解除分配在实验室中预配和使用的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开 **“Cloud Shell”** 窗格内的 **“PowerShell”** shell 会话。
1. 在 “Cloud Shell” 窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建和使用的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建和使用的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**备注**：该命令异步执行（由 -NoWait 参数确定），因此，尽管可立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上要花几分钟才能停止和解除分配 Azure VM。
