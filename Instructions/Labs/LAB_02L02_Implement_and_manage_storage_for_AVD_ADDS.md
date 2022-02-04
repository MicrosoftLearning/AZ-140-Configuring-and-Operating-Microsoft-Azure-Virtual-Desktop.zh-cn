---
lab:
    title: '实验室：实现和管理 AVD 的存储 (AD DS)'
    module: '模块 2：实现 AVD 基础结构'
---

# 实验室 - 实现和管理 AVD 的存储 (AD DS)
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有将在本实验室中使用的 Azure 订阅的所有者或参与者角色，以及与 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
- 已完成实验室 **“准备部署 Azure 虚拟桌面 (AD DS)”**

## 预计用时

30 分钟

## 实验室场景

你需要在 Azure Active Directory 域服务 (Azure AD DS) 环境中实现和管理 Azure 虚拟桌面部署的存储。

## 目标
  
完成本实验室后，你将能够：

- 配置 Azure 文件存储，以存储 Azure 虚拟桌面的配置文件容器

## 实验室文件

- 无

## 说明

### 练习 1：配置 Azure 文件存储，以存储 Azure 虚拟桌面的配置文件容器

本练习的主要任务如下：

1. 创建 Azure 存储帐户
1. 创建 Azure 文件存储共享
1. 为 Azure 存储帐户启用 AD DS 身份验证 
1. 配置 Azure 文件存储基于 RBAC 的权限
1. 配置 Azure 文件存储文件系统权限

#### 任务 1：创建 Azure 存储帐户

1. 在实验室计算机上，启动 Web 浏览器，导航至 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，搜索并选择 **“虚拟机”**，然后在 **“虚拟机”** 边栏选项卡中，选择 **“az140-dc-vm11”**。
1. 在“**az140-dc-vm11**”边栏选项卡上，选择“**连接**”，在下拉菜单中选择“**Bastion**”，在“**az140-dc-vm11 \| 连接**”边栏选项卡的“**Bastion**”选项卡上，选择“**使用 Bastion**”。
1. 出现提示时，请提供以下凭据并选择“**连接**”：

   |设置|值|
   |---|---|
   |用户名|**Student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 **az140-dc-vm11** 的远程桌面会话中，启动 Microsoft Edge，并导航到 [Azure 门户](https://portal.azure.com)。如果出现提示，请使用在本实验室使用的订阅中具有所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中搜索并选择 **“存储帐户”**，并在 **“存储帐户”** 边栏选项卡上选择 **“+ 添加”**。
1. 在 **“创建存储帐户”** 边栏选项卡的 **“基本设置”** 选项卡上，指定以下设置（其他设置则保留为默认值）：

   |设置|值|
   |---|---|
   |订阅|在本实验室中使用的 Azure 订阅的名称|
   |资源组|新资源组名称 **az140-22-RG**|
   |存储帐户名称|由小写字母和数字组成、以字母开头、长度介于 3 到 15 之间的全局唯一名称|
   |区域|托管 Azure 虚拟桌面实验室环境的 Azure 区域的名称|
   |性能|**标准**|
   |冗余|**异地冗余存储 (GRS)**|
   |在区域不可用的情况下，提供对数据的读取访问|已启用|

   >**备注**： 请确保存储帐户名称的长度不超过 15 个字符。该名称将用于在 Active Directory 域服务 (AD DS) 域中创建与包含存储帐户的 Azure 订阅关联的 Azure AD 租户集成的计算机帐户。这将允许在访问此存储帐户中托管的文件共享时进行基于 AD DS 的身份验证。

1. 在 **“创建存储帐户”** 边栏选项卡的 **“基本信息”** 选项卡中，选择 **“查看 + 创建”**，等待验证过程完成，然后选择 **“创建”**。

   >**备注**： 请等待存储帐户创建完成。该操作需约 2 分钟。

#### 任务 2：创建 Azure 文件存储共享

1. 在与 **az140-dc-vm11** 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中导航回 **“存储帐户”** 边栏选项卡，并选择表示新创建的存储帐户的条目。
1. 在“存储帐户”边栏选项卡的 **“数据存储”** 部分，选择 **“文件共享”**，然后选择 **“+ 文件共享”**。
1. 在 **“新建文件共享”** 边栏选项卡上，指定以下设置并选择 **“创建”** （其他设置保留默认值）：

   |设置|值|
   |---|---|
   |名称|**az140-22-profiles**|
   |层级|**事务优化**|

#### 任务 3：为 Azure 存储帐户启用 AD DS 身份验证 

1. 在与 **az140-dc-vm11** 的远程桌面会话中，在 Microsoft Edge 窗口中打开一个新标签页，导航到[Azure 文件存储示例 GitHub 存储库](https://github.com/Azure-Samples/azure-files-samples/releases)，下载最新版本的压缩“**AzFilesHybrid.zip**”PowerShell 模块，并将内容解压缩到 **C:\\Allfiles\\Labs\\02** 文件夹（如有需要，请创建该文件夹）。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，以管理员的身份启动 **Windows PowerShell ISE**，然后从 **“管理员: Windows PowerShell ISE”** 脚本窗格中，运行以下命令以删除 **Zone.Identifier** 备用数据流，该数据流具有一个值 **3**，指示其是从 Internet 下载的：

   ```powershell
   Get-ChildItem -Path C:\Allfiles\Labs\02 -File -Recurse | Unblock-File
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台运行以下命令，以登录 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请使用在具有本实验室所用订阅的所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格，运行以下命令，设置运行后续脚本所需的变量：

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格中，运行以下命令，创建 AD DS 计算机对象，其表示之前在本任务中创建的 Azure 存储帐户，并用于实现其 AD DS 身份验证：

   ```powershell
   Set-Location -Path 'C:\Allfiles\Labs\02'
   .\CopyToPSPath.ps1 
   Import-Module -Name AzFilesHybrid
   Join-AzStorageAccountForAuth `
      -ResourceGroupName $ResourceGroupName `
      -StorageAccountName $StorageAccountName `
      -DomainAccountType 'ComputerAccount' `
      -OrganizationalUnitDistinguishedName 'OU=WVDInfra,DC=adatum,DC=com'
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，从 **“管理员: Windows PowerShell ISE”** 脚本窗格中，运行以下命令，验证 Azure 存储帐户上是否启用了 AD DS 身份验证：

   ```powershell
   $storageaccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
   $storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
   $storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions
   ```

1. 验证命令 `$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties` 的输出是否返回 `AD` （表示存储帐户的目录服务），并验证 `$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions` 命令的输出（表示目录域信息）是否类似以下格式（`DomainGuid`、`DomainSid` 和 `AzureStorageSid` 的值将不同）：

   ```
   DomainName        : adatum.com
   NetBiosDomainName : adatum.com
   ForestName        : adatum.com
   DomainGuid        : 47c93969-9b12-4e01-ab81-1508cae3ddc8
   DomainSid         : S-1-5-21-1102940778-2483248400-1820931179
   AzureStorageSid   : S-1-5-21-1102940778-2483248400-1820931179-2109
   ```

1. 在与 **az140-dc-vm11** 的远程桌面会话中，切换到显示 Azure 门户的 Microsoft Edge 窗口，在显示存储帐户的边栏选项卡上，选择“**文件共享**”，验证“**Active Directory**”设置是否为“**已配置**”。

   >**备注**： 可能必须刷新浏览器页面才能使更改在 Azure 门户中反映出来。

#### 任务 4：配置 Azure 文件存储基于 RBAC 的权限

1. 依次在与 **az140-dc-vm11** 的远程桌面会话中，显示 Azure 门户的 Microsoft Edge 窗口中，显示你之前在本练习中创建的存储帐户属性的边栏选项卡上，左侧的垂直菜单中， **“数据存储”** 部分中，选择 **“文件共享”**。
1. 在 **“文件共享”** 边栏选项卡的共享列表中，选择 **“az140-22-profiles”** 条目。
1. 在 **az140-22-profiles** 边栏选项卡的左侧垂直菜单中，选择 **“访问控制(IAM)”**。
1. 在存储帐户的 **“访问控制(IAM)”** 边栏选项卡上，单击 **“+ 添加”**，然后在下拉菜单中选择 **“添加角色分配”**， 
1. 在 **“添加角色分配”** 边栏选项卡上，指定以下设置并选择 **“保存”**：

   |设置|值|
   |---|---|
   |角色|**存储文件数据 SMB 共享参与者**|
   |将访问权限分配到|**用户、组或服务主体**|
   |选择|**az140-wvd-users**|

1. 在存储帐户的 **“访问控制(IAM)”** 边栏选项卡上，单击 **“+ 添加”**，然后在下拉菜单中选择 **“添加角色分配”**， 
1. 在 **“添加角色分配”** 边栏选项卡上，指定以下设置并选择 **“保存”**：

   |设置|值|
   |---|---|
   |角色|**存储文件数据 SMB 共享提升参与者**|
   |将访问权限分配到|**用户、组或服务主体**|
   |选择|**az140-wvd-admins**|

#### 任务 5：配置 Azure 文件存储文件系统权限

1. 在与 **az140-dc-vm11** 的远程桌面会话中，切换到 **“管理员: Windows PowerShell ISE”** 窗口，然后从 **“管理员: Windows PowerShell ISE”** 脚本窗格中，运行以下命令，创建引用你之前在本练习中创建的存储帐户的名称和密钥的变量：

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0]
   $storageAccountName = $storageAccount.StorageAccountName
   $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 脚本窗格中，运行以下命令，创建映射到你之前在本练习中创建的文件共享的驱动器：

   ```powershell
   $fileShareName = 'az140-22-profiles'
   net use Z: "\\$storageAccountName.file.core.windows.net\$fileShareName" /u:AZURE\$storageAccountName $storageAccountKey
   ```

1. 从 **“管理员: Windows PowerShell ISE”** 控制台，运行以下命令，查看当前文件系统权限：

   ```powershell
   icacls Z:
   ```

   >**备注**： 默认情况下， **NT Authority\\Authenticated Users** 和 **BUILTIN\\Users** 都具有允许用户读取其他用户的配置文件容器的权限。你将删除它们并添加所需的最低权限。

1. 从 **“管理员: Windows PowerShell ISE”** 脚本窗格中，运行以下命令，调整文件系统权限，使其符合最小权限原则：

   ```powershell
   $permissions = 'ADATUM\az140-wvd-admins'+':(F)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'ADATUM\az140-wvd-users'+':(M)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'Creator Owner'+':(OI)(CI)(IO)(M)'
   cmd /c icacls Z: /grant $permissions
   icacls Z: /remove 'Authenticated Users'
   icacls Z: /remove 'Builtin\Users'
   ```

   >**备注**： 或者，你可以通过使用文件资源管理器设置权限。
