---
lab:
  title: 实验室：为 AVD 实现和管理存储 (Azure AD DS)
  module: 'Module 2: Implement a AVD Infrastructure'
---

# <a name="lab---implement-and-manage-storage-for-avd-azure-ad-ds"></a>实验室 - 实现和管理 AVD 的存储 (Azure AD DS)
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- Azure 订阅
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色，并且在 Azure 订阅中具有所有者或参与者角色
- 已完成实验室“准备部署 Azure 虚拟桌面 (Azure AD DS)”

## <a name="estimated-time"></a>预计用时

30 分钟

## <a name="lab-scenario"></a>实验室方案

你需要在 Azure Active Directory 域服务 (Azure AD DS) 环境中实现和管理 Azure 虚拟桌面部署的存储。

## <a name="objectives"></a>目标
  
完成本实验室后，你将能够：

- 配置 Azure 文件存储，以在 Azure AD DS 环境中存储 Azure 虚拟桌面的配置文件容器

## <a name="lab-files"></a>实验室文件 

- 无

## <a name="instructions"></a>说明

### <a name="exercise-1-configure-azure-files-to-store-profile-containers-for-azure-virtual-desktop"></a>练习 1：配置 Azure 文件存储，以存储 Azure 虚拟桌面的配置文件容器

此练习的主要任务如下：

1. 创建 Azure 存储帐户
1. 创建 Azure 文件存储共享
1. 为 Azure 存储帐户启用 Azure AD DS 身份验证 
1. 配置 Azure 文件存储共享权限
1. 配置 Azure 文件存储目录和文件级别权限

#### <a name="task-1-create-an-azure-storage-account"></a>任务 1：创建 Azure 存储帐户

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在实验室计算机上，在 Azure 门户中搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az140-cl-vm11a”条目  。 此时会打开“az140-cl-vm11a”边栏选项卡。
1. 在“az140-cl-vm11a”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-cl-vm11a \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**aadadmin1@adatum.com**|
   |密码|之前定义的密码|

1. 在与 az140-cl-vm11a Azure VM 的远程桌面中，启动 Microsoft Edge 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供 aadadmin1 用户帐户的用户主体名称和创建此帐户时设置的密码进行登录 。

   >**注意**：通过从 Active Directory 用户和计算机控制台查看 aadadmin1 帐户的“属性”对话框，或切换回实验室计算机并从 Azure 门户中的“Azure AD 租户”边栏选项卡查看其属性，可以标识 aadadmin1 帐户的用户主体名称 (UPN) 属性。

1. 在与 az140-cl-vm11a 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中搜索并选择“存储帐户”，并在“存储帐户”边栏选项卡上选择“+ 创建”   。
1. 在“创建存储帐户”边栏选项卡的“基本信息”选项卡上，指定以下设置（其他设置保留默认值） ：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|新资源组名称 az140-22a-RG|
   |存储帐户名称|由小写字母和数字组成、以字母开头、长度介于 3 到 15 之间的全局唯一名称|
   |位置|托管 Azure 虚拟桌面实验室环境的 Azure 区域的名称|
   |性能|**标准**|
   |复制|**本地冗余存储 (LRS)**|

   >**注意**：请确保存储帐户名称的长度不超过 15 个字符。 该名称将用于在 Active Directory 域服务 (AD DS) 域中创建与包含存储帐户的 Azure 订阅关联的 Azure AD 租户集成的计算机帐户。 这将允许在访问此存储帐户中托管的文件共享时进行基于 AD DS 的身份验证。

1. 在“创建存储帐户”边栏选项卡的“基本信息”选项卡上，选择“查看 + 创建”，等待验证过程完成，然后选择“创建”。

   >**注意**：请等待存储帐户创建完成。 这应该需要大约两分钟。

#### <a name="task-2-create-an-azure-files-share"></a>任务 2：创建 Azure 文件存储共享

1. 在与 az140-cl-vm11a 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中导航回“存储帐户”边栏选项卡，并选择表示新创建的存储帐户的条目 。
1. 在“存储帐户”边栏选项卡左侧垂直菜单的“数据存储”部分中，选择“文件共享”，然后选择“+ 文件共享”  。
1. 在“新建文件共享”边栏选项卡上，指定以下设置并选择“创建”（其他设置保留默认值） ：

   |设置|值|
   |---|---|
   |名称|az140-22a-profiles|

#### <a name="task-3-enable-azure-ad-ds-authentication-for-the-azure-storage-account"></a>任务 3：为 Azure 存储帐户启用 Azure AD DS 身份验证

1. 依次在 az140-cl-vm11a 的远程桌面会话中，Microsoft Edge 窗口中，Azure 门户中，显示你在上一个任务中创建的存储帐户属性的边栏选项卡上，左侧的垂直菜单中，“数据存储”部分中，选择“文件共享”  。 
1. 在“文件共享设置”部分的“Active Directory”标签旁边，选择“未配置”链接  。
1. 在“启用 Active Directory 源”部分中，在标有“Azure Active Directory 域服务”的矩形中，选择“设置”  。
1 在“基于标识的访问”边栏选项卡上，选择“已启用”选项，然后选择“保存”  。

#### <a name="task-4-configure-the-azure-files-rbac-based-permissions"></a>任务 4：配置 Azure 文件存储基于 RBAC 的权限

1. 依次在与 az140-cl-vm11a 的远程桌面会话中，显示 Azure 门户的 Microsoft Edge 窗口中，显示你之前在本练习中创建的存储帐户属性的边栏选项卡上，左侧的垂直菜单中，“数据存储”部分中，选择“文件共享”，然后在共享列表中，选择“az140-22a-profiles”条目   。
1. 在“az140-22a-profiles”边栏选项卡的左侧垂直菜单中，选择“访问控制(IAM)” 。
1. 在“az140-22a-profiles \| 访问控制(IAM)”边栏选项卡上，选择“+ 添加”，然后从下拉菜单中选择“添加角色分配”  。
1. 在“添加角色分配”边栏选项卡上，选择“存储文件数据 SMB 共享参与者”，然后选择“下一步”  ：
1. 在“成员”边栏选项卡上，选择“将访问权限分配给”，然后单击“+ 选择成员”  。
1. 在“选择成员”边栏选项卡上的“选择”文本框中，键入“az140-wvd-ausers”，然后单击“选择”   。
1. 在“成员”边栏选项卡上，选择两次“查看 + 分配” 。
1. 重复上述步骤 3-8，指定以下设置：

   |设置|值|
   |---|---|
   |角色|**存储文件数据 SMB 共享提升参与者**|
   |选择|az140-wvd-aadmins|

   > **注意**：你将使用 aadadmin1 用户帐户，它是 az140-wvd-aadmins 组的成员，用于配置文件共享权限 。 

#### <a name="task-5-configure-the-azure-files-directory-and-file-level-permissions"></a>任务 5：配置 Azure 文件存储目录和文件级别权限

1. 在 az140-cl-vm11a 的远程桌面会话中，启动“命令提示符”，然后从“命令提示符”窗口，运行以下命令，将驱动器映射到目标共享（将 `<storage-account-name>` 占位符替换为存储帐户的名称）  ：

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-22a-profiles
   ```

1. 在 az140-cl-vm11a 的远程桌面会话中，打开文件资源管理器，导航到新映射的 Z: 驱动器，显示其“属性”对话框，依次选择“安全性”选项卡、“编辑”、“添加”，在“选择用户、计算机、服务帐户和组”对话框中，确保“查找位置”文本框包含“adatum.com”条目，在“输入对象名称以进行选择”文本框中，键入“az140-wvd-ausers”，然后单击“确定”          。
1. 返回显示映射驱动器权限的对话框的“安全性”选项卡，确保已选择“az140-wvd-ausers”条目，选择“允许”列表中的“修改”复选框，单击“确定”，查看“Windows 安全”文本框中显示的消息，然后单击“是”      。 
1. 返回显示映射驱动器权限的对话框的“安全性”选项卡，依次选择“编辑”、“添加”，在“选择用户、计算机、服务帐户和组”对话框中，确保“查找位置”文本框包含“adatum.com”条目，在“输入对象名称以进行选择”文本框中，键入“az140-wvd-aadmins”，然后单击“确定”        。
1. 返回显示映射驱动器权限的对话框的“安全性”选项卡，确保已选择“az140-wvd-aadmins”条目，选择“允许”列表中的“完全控制”复选框，单击“确定”    。 
1. 在显示映射驱动器权限的对话框的“安全性”选项卡上，选择“编辑”，在组和用户名的列表中，选择“以通过身份验证的用户”条目。然后选择“删除”   。
1. 仍在“编辑”屏幕上时，在组和用户名列表中，选择“用户”条目，选择“删除”，单击“确定”，然后单击“确定”两次以完成该过程   。 

   >**注意**：或者，你可以使用 icacls 命令行实用工具设置权限。 
