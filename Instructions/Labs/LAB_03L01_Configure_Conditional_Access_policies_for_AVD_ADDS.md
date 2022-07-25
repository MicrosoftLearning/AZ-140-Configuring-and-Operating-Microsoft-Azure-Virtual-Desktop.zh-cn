---
lab:
  title: 实验室：为 WVD (AD DS) 配置条件访问策略
  module: 'Module 3: Manage Access and Security'
---

# <a name="lab---configure-conditional-access-policies-for-wvd-ad-ds"></a>实验室 - 为 WVD (AD DS) 配置条件访问策略
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- Azure 订阅
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色，并且在 Azure 订阅中具有所有者或参与者角色
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”
- 已完成实验室“使用 Azure 门户部署主机池和会话主机 (AD DS)”

## <a name="estimated-time"></a>预计用时

60 分钟

## <a name="lab-scenario"></a>实验室方案

在 Active Directory 域服务 (AD DS) 环境中，你需要使用 Azure Active Directory (Azure AD) 条件访问控制对 Azure 虚拟桌面的部署的访问。

## <a name="objectives"></a>目标
  
完成本实验室后，你将能够：

- 为 Azure 虚拟桌面准备基于 Azure Active Directory (Azure AD) 的条件访问
- 为 Azure 虚拟桌面实现基于 Azure AD 的条件访问

## <a name="lab-files"></a>实验室文件 

- 无 

## <a name="instructions"></a>说明

### <a name="exercise-1-prepare-for-azure-ad-based-conditional-access-for-azure-virtual-desktop"></a>练习 1：为 Azure 虚拟桌面准备基于 Azure AD 的条件访问

此练习的主要任务如下：

1. 配置 Azure AD Premium P2 许可
1. 配置 Azure AD 多重身份验证 (MFA)
1. 为 Azure AD MFA 注册用户
1. 配置混合 Azure AD 联接
1. 触发 Azure AD Connect 增量同步

#### <a name="task-1-configure-azure-ad-premium-p2-licensing"></a>任务 1：配置 Azure AD Premium P2 许可

>**注意**：为了实现 Azure AD 条件访问，需要 Azure AD 的高级 P1 或 P2 许可。 你将试用此实验室 30 天。

1. 在实验室计算机上，启动 Web 浏览器，导航至 [Azure 门户](https://portal.azure.com)，然后通过提供用户帐户（该帐户具有你将在本实验室使用的订阅中的所有者角色以及与该订阅关联的 Azure AD 租户中的全局管理员角色）的凭据进行登录。
1. 在 Azure 门户中，搜索并选择 Azure Active Directory，导航到与用于此实验室的 Azure 订阅相关联的 Azure AD 租户。
1. 在 Azure Active Directory 边栏选项卡左侧的垂直菜单栏的“管理”部分中，单击“用户” 。 
1. 在“用户 | 所有用户(预览版)”边栏选项卡上，选择“aduser5” 。
1. 在“aduser5 | 配置文件”边栏选项卡上的工具栏中，单击“编辑”，在“设置”部分的“使用位置”下拉列表中，选择实验室环境所在的国家/地区，然后在工具栏中，单击“保存”    。
1. 在“aduser5 | 配置文件”边栏选项卡上的“标识”部分中，标识“aduser5”帐户的用户主体名称  。

    >**注意**：记下此值。 本实验室中稍后会用到它。

1. 在“用户 | 所有用户（预览）”边栏选项卡上，选择在此任务开始时用于登录的用户帐户，如果帐户没有分配“使用位置”，则重复前面的步骤 。 

    >**注意**：必须设置“使用位置”属性才能为用户帐户分配 Azure AD Premium P2 许可证。

1. 在“用户 | 所有用户（预览）”边栏选项卡上，选择“aadsyncuser”用户帐户并识别其用户主体名称 。

    >**注意**：记下此值。 本实验室中稍后会用到它。

1. 在 Azure 门户中，导航回 Azure AD 租户的“概述”边栏选项卡，并在左侧垂直菜单栏的“管理”部分中，单击“许可证”  。
1. 在“许可证 \| 概述”边栏选项卡左侧垂直菜单栏的“管理”部分中，单击“所有产品”  。
1. 在“许可证 \| 所有产品“边栏选项卡上的工具栏中，单击“+ 试用/购买” 。
1. 在“激活”边栏选项卡上的“企业移动性 + 安全性 E5”部分中单击“免费试用”，然后单击“激活”   。 
1. 在“许可证 \| 概述”边栏选项卡上时，刷新浏览器窗口以验证激活是否成功。 
1. 在“许可证 - 所有产品”边栏选项卡上，选择“企业移动性 + 安全性 E5”条目 。 
1. 在“企业移动性 + 安全性 E5”边栏选项卡上的工具栏中，单击“+ 分配” 。
1. 在“分配许可证”边栏选项卡上，单击“添加用户和组”，然后在“添加用户和组”边栏选项卡上，选择“aduser5”和你的用户帐户，并单击“选择”    。
1. 回到“分配许可证”边栏选项卡上，单击“分配选项”，在“分配选项”边栏选项卡上，验证所有选项都已启用，然后依次单击“查看 + 分配”和“分配”    。

#### <a name="task-2-configure-azure-ad-multi-factor-authentication-mfa"></a>任务 2：配置 Azure AD 多重身份验证 (MFA)

1. 在实验室计算机上显示 Azure 门户的 Web 浏览器中，导航回 Azure AD 租户的“概述”边栏选项卡，并在左侧垂直菜单的“管理”部分中，单击“安全性”  。
1. 在“安全 | 启动”边栏选项卡上左侧垂直菜单栏的“管理”部分中，单击“标识保护”  。
1. 在“标识保护 | 概述”边栏选项卡上左侧垂直菜单栏的“保护”部分中，单击“MFA 注册策略”（如有必要，请刷新 Web 浏览器页面）  。
1. 在“标识保护 | MFA 注册策略”边栏选项卡上，在“多重身份验证注册策略”的“分配”部分中，单击“所有用户”，在“包括”选项卡上，单击“选择个人和组”选项，在“选择用户”上，依次单击“aduser5”和“选择”，在边栏选项卡底部，将“执行策略”切换为“开”，然后单击“保存”            。

#### <a name="task-3-register-a-user-for-azure-ad-mfa"></a>任务 3：为 Azure AD MFA 注册用户

1. 在实验室计算机上，打开“InPrivate”web 浏览器会话，导航到 [Azure 门户](https://portal.azure.com)，并通过提供此前在此练习中标识的用户主体名称 aduser5 和在创建此用户帐户时设置的密码来登录 。
1. 出现消息“需要更多信息”时，单击“下一步” 。 这会自动将浏览器重定向到“Microsoft Authenticator”页面。
1. 在“附加安全验证”页面上，在“步骤 1: 我们应该如何与你联系？”部分，选择你首选的身份验证方法并按照说明完成注册过程。 
1. 在 Azure 门户页的右上角，单击代表用户头像的图标，单击“注销”，并关闭“In private”浏览器窗口 。 

#### <a name="task-4-configure-hybrid-azure-ad-join"></a>任务 4：配置混合 Azure AD 联接

> **注意**：当根据设备的 Azure AD 联接状态为其设置条件访问时，可以利用此功能实现额外的安全性。

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中选择“az140-dc-vm11”  。
1. 在“az140-dc-vm11”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-dc-vm11 \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-dc-vm11 的远程桌面会话中，在“开始”菜单中展开 Azure AD Connect 文件夹，并选择“Azure AD Connect”   。
> 注意：如果收到同步服务未运行的故障错误窗口，请转到 PowerShell 命令窗口并输入 Start-Service "ADSync"，然后再次尝试执行步骤 4 。
3. 在“Microsoft Azure Active Directory Connect”窗口的“欢迎使用 Azure AD Connect”页上，选择“配置”  。
4. 在“Microsoft Azure Active Directory Connect”窗口的“其他任务”页上，选择“配置设备选项”，并选择“下一步”   。
5. 在“Microsoft Azure Active Directory Connect”窗口中的“概述”页上，查看与“混合 Azure AD 联接”和“设备写回”相关的信息，并选择“下一步”    。
6. 在“Microsoft Azure Active Directory Connect”窗口中的“连接到 Azure AD”页上，使用在前面的练习中创建的用户帐户“aadsyncuser”的凭据进行身份验证，然后选择“下一步”   。  

   > **注意**：提供在本实验室前面记录的 aadsyncuser 帐户的 userPrincipalName 属性，并指定在创建此用户帐户时设置的密码。 

1. 在“Microsoft Azure Active Directory Connect”窗口中的“设备选项”页上，确保已选择“配置混合 Azure AD 联接”选项，并选择“下一步”   。 
1. 在“Microsoft Azure Active Directory Connect”窗口中的“设备操作系统”页上，选择“Windows 10 或更高版本的已加入域的设备”复选框，并选择“下一步”   。 
1. 在“Microsoft Azure Active Directory Connect”窗口中的“SCP 配置”页上，选择“adatum.com”条目旁边的复选框，在“身份验证服务”下拉列表中，选择“Azure Active Directory”条目，并选择“添加”     。 
1. 出现提示时，在“企业管理员凭据”对话框中指定以下凭据，并选择“确定”： 

   |设置|值|
   |---|---|
   |用户名|ADATUM\Student|
   |密码|**Pa55w.rd1234**|

1. 回到“Microsoft Azure Active Directory Connect”窗口的“SCP 配置”页，选择“下一步”  。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“准备配置”页上，选择“配置”，配置完成后选择“退出”   。
1. 在与 az140-dc-vm11 的远程桌面会话中，以管理员身份启动 Windows PowerShell ISE 。
1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”控制台，运行以下命令，将“az140-cl-vm11”计算机帐户移动到“WVDClients”组织单位 (OU) ：

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，在“开始”菜单中展开 Azure AD Connect 文件夹，并选择“Azure AD Connect”   。
1. 在“Microsoft Azure Active Directory Connect”窗口的“欢迎使用 Azure AD Connect”页上，选择“配置”  。
1. 在“Microsoft Azure Active Directory Connect”窗口的“附加任务”页上，依次选择“自定义同步选项”和“下一步”   。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“连接到 Azure AD”页上，使用在前面的练习中创建的用户帐户“aadsyncuser”的凭据进行身份验证，然后选择“下一步”   。 

   > **注意**：提供在本实验室前面记录的 aadsyncuser 帐户的 userPrincipalName 属性，并指定在创建此用户帐户时设置的密码。 

1. 在“Microsoft Azure Active Directory Connect”窗口的“连接到目录”页上，选择“下一步”  。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“域和 OU 筛选”页上，确保已选择“同步已选择的域和 OU”选项，展开“adatum.com”节点，确保已选择“ToSync”OU 旁的复选框，然后选择“WVDClients”OU 旁的复选框，并选择“下一步”      。
1. 在“Microsoft Azure Active Directory Connect”窗口的“可选功能”页上，接受默认设置，然后选择“下一步”  。
1. 在“Microsoft Azure Active Directory Connect”窗口的“准备配置”页上，确保已选中“配置完成后启动同步过程”复选框，然后选择“配置”   。
1. 查看“配置完成”页中的信息，然后选择“退出”关闭“Microsoft Azure Active Directory Connect”窗口。

#### <a name="task-5-trigger-azure-ad-connect-delta-synchronization"></a>任务 5：触发 Azure AD Connect 增量同步

1. 在与 az140-dc-vm11 的远程桌面会话中，切换到“管理员: Windows PowerShell ISE”窗口。
1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”控制台窗格，运行以下命令，触发 Azure AD Connect 增量同步：

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，启动 Microsoft Edge，并导航到 [Azure 门户](https://portal.azure.com)。 如果出现提示，请使用用户帐户的 Azure AD 凭据登录，该帐户应具有与本实验室所用 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
1. 在与 az140-dc-vm11 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中搜索并选择“Azure Active Directory”，以导航到与本实验室中所用 Azure 订阅关联的 Azure AD 租户 。
1. 在“Azure Active Directory”边栏选项卡左侧垂直菜单中的“管理”部分，单击“设备” 。 
1. 在“设备 | 所有设备”边栏选项卡上，查看设备列表，验证“az140-cl-vm11”设备是否在“联接类型”列中列出了“已建立混合 Azure AD 联接”条目   。

   > **注意**：在设备出现在 Azure 门户中之前，可能需要等待几分钟才能使同步生效。

### <a name="exercise-2-implement-azure-ad-based-conditional-access-for-azure-virtual-desktop"></a>练习 2：为 Azure 虚拟桌面实现基于 Azure AD 的条件访问

此练习的主要任务如下：

1. 为所有 Azure 虚拟桌面连接创建基于 Azure AD 的条件访问策略
1. 为所有 Azure 虚拟桌面连接测试基于 Azure AD 的条件访问策略
1. 修改基于 Azure AD 的条件访问策略，将已加入混合 Azure AD 的计算机从 MFA 要求中排除
1. 测试修改后的基于 Azure AD 的条件访问策略

#### <a name="task-1-create-an-azure-ad-based-conditional-access-policy-for-all-azure-virtual-desktop-connections"></a>任务 1：为所有 Azure 虚拟桌面连接创建基于 Azure AD 的条件访问策略

>**注意**：在此任务中，你将配置一个基于 Azure AD 的条件访问策略，该策略要求 MFA 登录到 Azure 虚拟桌面会话。 该策略还将在身份验证成功后的 4 个小时后强制重新进行身份验证。

1. 在实验室计算机上显示 Azure 门户的 Web 浏览器中，导航回 Azure AD 租户的“概述”边栏选项卡，并在左侧垂直菜单的“管理”部分中，单击“安全性”  。
1. 在“安全 \| 启动”边栏选项卡上左侧垂直菜单栏的“保护”部分中，单击“条件访问”  。
1. 在“条件访问策略”边栏选项卡的工具栏中，单击“+ 新建策略”，然后在上下文菜单中选择“创建新策略” **\|**  。
1. 在“新建”边栏选项卡上，配置以下设置：

   - 在“名称”文本框中键入“az140-31-wvdpolicy1” 
   - 在“分配”部分中，选择“用户或工作负载标识”选项，在“此策略的适用对象”下拉列表中，确保选中“用户和组”，在“选择用户和组”部分，选中“用户和组”复选框，在“选择”边栏选项卡上，单击“aduser5”，然后单击“选择”        。
   - 在“分配”部分中，单击“云应用或操作”，确保已选择“选择要应用的策略”切换中的“云应用”选项，然后单击“选择应用”选项，在“选择”边栏选项卡上，选择“Azure 虚拟桌面”条目旁的复选框，然后单击“选择”       。 
   - 在“分配”部分中，依次单击“条件”和“客户端应用”，在“客户端应用”边栏选项卡上，将“配置”开关设置为“是”，确保已选择“浏览器”和“移动应用和桌面客户端”复选框，然后单击“完成”        。
   - 在“访问控制”部分，单击“授予”，在“授予”边栏选项卡上，确保已选择“授予访问权限”选项，选择“要求多重身份验证”复选框，然后单击“选择”     。
   - 在“访问控制”部分中，单击“会话”，在“会话”边栏选项卡上，选择“登录频率”复选框，然后在第一个文本框中，键入“4”，在“选择单位”下拉列表中，选择“小时”，清除“持续的浏览器会话”复选框，然后单击“选择”        。
   - 将“启用策略”开关设置为“开” 。

1. 在“新建”边栏选项卡中，单击“创建”。******** 

#### <a name="task-2-test-the-azure-ad-based-conditional-access-policy-for-all-azure-virtual-desktop-connections"></a>任务 2：为所有 Azure 虚拟桌面连接测试基于 Azure AD 的条件访问策略

1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”窗格内的“PowerShell”shell 会话。
1. 从“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，以启动你将在本实验室中使用的 Azure 虚拟桌面会话主机 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**注意**：等待命令完成并且 az140-21-RG 资源组中的所有 Azure VM 都在运行。 

1. 在实验室计算机上，打开“InPrivate”web 浏览器会话，导航到 [Azure 门户](https://portal.azure.com)，并通过提供此前在此练习中标识的用户主体名称 aduser5 和在创建此用户帐户时设置的密码来登录 。

   > **注意**：请确认没有提示你通过 MFA 进行身份验证。

1. 在“InPrivate”Web 浏览器会话中，导航到 Azure 虚拟桌面 HTML5 Web 客户端页面（[https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient)）。

   > **注意**：验证这将通过 MFA 自动触发身份验证。

1. 在“输入代码”窗格中，键入短信或注册的身份验证器应用中的代码，然后选择“验证” 。
1. 在“所有资源”页上，单击“命令提示符”，在“访问本地资源”窗格中，清除“打印机”复选框，然后单击“允许”    。
1. 在“输入你的凭据”中出现提示时，在“用户名”文本框中键入用户主体名称“aduser5”，在“密码”文本框中，键入在创建此用户帐户时设置的密码并单击“提交”    。
1. 验证“命令提示符”远程应用是否已成功启动。
1. 在命令提示符远程应用窗口中，在命令提示符处，键入“logoff”并按下 Enter 键  。
1. 返回“所有资源”页，在右上角单击“aduser5”，然后在下拉菜单中单击“注销”，并关闭“InPrivate”Web 浏览器窗口   。

#### <a name="task-3-modify-the-azure-ad-based-conditional-access-policy-to-exclude-hybrid-azure-ad-joined-computers-from-the-mfa-requirement"></a>任务 3：修改基于 Azure AD 的条件访问策略，将已加入混合 Azure AD 的计算机从 MFA 要求中排除

>**注意**：在此任务中，你将修改基于 Azure AD 的条件访问策略，该策略需要 MFA 才能登录到 Azure 虚拟桌面会话，这样源自 Azure AD 联接计算机的连接将不需要 MFA。

1. 在实验室计算机上显示 Azure 门户的浏览器窗口中，在“条件访问 | 策略”边栏选项卡上单击代表 az140-31-wvdpolicy1 策略的条目 。
1. 在“az140-31-wvdpolicy1”边栏选项卡上，在“访问控制”部分中单击“授权”，在“授权”边栏选项卡上选中“要求多重身份验证”和“要求使用已建立混合 Azure AD 联接的设备”复选框，确保“需要某一已选控制”选项已启用，然后单击“选择”       。
1. 在“az140-31-wvdpolicy1”边栏选项卡上，单击“保存” 。

>**注意**：该策略可能需要几分钟的时间才能生效。

#### <a name="task-4-test-the-modified-azure-ad-based-conditional-access-policy"></a>任务 4：测试修改后的基于 Azure AD 的条件访问策略

1. 在实验室计算机上显示 Azure 门户的浏览器中，搜索并选择“虚拟机”，并在“虚拟机”边栏选项卡中，选择“az140-cl-vm11”条目  。
1. 在“az140-cl-vm11”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-cl-vm11 \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**Student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-cl-vm11 的远程桌面会话中，启动 Microsoft Edge 并导航到 Azure 虚拟桌面 HTML5 Web 客户端页（[https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient)）。

   > **注意**：确认这次不会提示你通过 MFA 进行身份验证。 这是因为“az140-cl-vm11”是已建立混合 Azure AD 联接的。

1. 在“所有资源”页上，双击“命令提示符”，在“访问本地资源”窗格中，清除“打印机”复选框，然后单击“允许”    。
1. 在“输入你的凭据”中出现提示时，在“用户名”文本框中键入用户主体名称“aduser5”，在“密码”文本框中，键入在创建此用户帐户时设置的密码并单击“提交”    。
1. 验证“命令提示符”远程应用是否已成功启动。
1. 在命令提示符远程应用窗口中，在命令提示符处，键入“logoff”并按下 Enter 键  。
1. 返回“所有资源”页，在右上角单击“aduser5”，然后在下拉菜单中单击“注销”  。
1. 在与“az140-cl-vm11”的远程桌面会话中，在“启动”按钮正上方的垂直栏中单击“启动”，单击表示已登录的用户帐户的图标，然后在弹出菜单中单击“注销”   。

### <a name="exercise-3-stop-and-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>练习 3：停止并解除分配在实验室中预配和使用的 Azure VM

此练习的主要任务如下：

1. 停止并解除分配在实验室中预配和使用的 Azure VM

>**注意**：在本练习中，你将解除分配在本实验室中预配和使用的 Azure VM，以最大程度减少相应的计算费用

#### <a name="task-1-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>任务 1：解除分配在实验室中预配和使用的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开 Cloud Shell 窗格内的“PowerShell”shell 会话 。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建和使用的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建和使用的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注意**：该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能停止和解除分配 Azure VM。