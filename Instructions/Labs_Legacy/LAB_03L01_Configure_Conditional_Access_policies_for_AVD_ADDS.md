---
lab:
  title: 实验室：为 AVD (AD DS) 配置条件访问策略
  module: 'Module 3: Manage Access and Security'
---

# 实验室 - 为 AVD (AD DS) 配置条件访问策略
# 学生实验室手册

## 实验室依赖项

- Azure 订阅
- Microsoft 帐户或 Microsoft Entra 帐户，在与 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色并在 Azure 订阅中具有所有者或参与者角色
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”
- 已完成实验室“使用 Azure 门户部署主机池和会话主机 (AD DS)”

## 预计用时

60 分钟

## 实验室方案

你需要使用 Microsoft Entra 条件访问来控制对 Active Directory 域服务 (AD DS) 环境中 Azure 虚拟桌面部署的访问。

## 目标
  
完成本实验室后，你将能够：

- 为 Azure 虚拟桌面准备基于 Microsoft Entra 的条件访问
- 为 Azure 虚拟桌面实施基于 Microsoft Entra 的条件访问

## 实验室文件

- 无 

## 说明

>**重要说明**：Microsoft 已将 Azure Active Directory (Azure AD) 重命名为 Microsoft Entra ID。************ 有关此更改的详细信息，请参阅 [Azure Active Directory 的新名称](https://learn.microsoft.com/en-us/entra/fundamentals/new-name)。 这是一项持续的工作，因此，当你逐步完成各个练习时，你可能仍然会遇到实验室说明与界面元素不匹配的情况。 将这一点考虑在内（尤其在本实验室中，Microsoft Entra Connect 指定了新名称 Azure Active Directory Connect，而在练习 1 的任务 4 中配置服务连接点时，仍然使用“Azure Active Directory”这个词）。************

>**重要说明**：要激活 Microsoft Entra ID P2 试用版，需要提供信用卡信息。 因此，此练习完全可选做。 相反，课程讲师可能会选择向学生演示此功能。

### 练习 1：为 Azure 虚拟桌面准备基于 Microsoft Entra 的条件访问

此练习的主要任务如下：

1. 配置 Microsoft Entra 高级版 P2 许可
1. 配置 Microsoft Entra 多重身份验证 (MFA)
1. 注册 Microsoft Entra MFA 用户
1. 配置 Microsoft Entra 混合加入
1. 触发 Microsoft Azure Active Directory Connect 增量同步

#### 任务 1：配置 Microsoft Entra 高级版 P2 许可

>**备注**：需要有 Microsoft Entra 高级版 P1 或 P2 许可才能实施 Microsoft Entra 条件访问。 在本实验室中，你将使用 30 天试用版。

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后提供用户帐户的 Microsoft Entra 凭据进行登录；其中，该用户帐户在你将在本实验室中使用的订阅中具有所有者角色，并在与该订阅关联的 Microsoft Entra 租户中具有全局管理员角色。

    >**重要说明**：确保使用的是工作或学校帐户，而不是 Microsoft 帐户。****

1. 在 Azure 门户中，搜索并选择“Azure Active Directory”**** 以导航到用于此实验室的与 Azure 订阅关联的 Microsoft Entra 租户。
1. 在“Azure Active Directory”边栏选项卡上的左侧垂直菜单中的“管理”**** 部分中，单击“用户”****。 
1. 在“用户 | 所有用户(预览版)”边栏选项卡上，选择“aduser5”********。
1. 在“aduser5 | 配置文件”边栏选项卡上的工具栏中，单击“编辑”，在“设置”部分的“使用位置”下拉列表中，选择实验室环境所在的国家/地区，然后在工具栏中，单击“保存”********************。
1. 在“aduser5 | 配置文件”边栏选项卡上的“标识”部分中，标识“aduser5”帐户的用户主体名称************。

    >**注意**：记下此值。 本实验室中稍后会用到它。

1. 在“用户 | 所有用户（预览）”边栏选项卡上，选择在此任务开始时用于登录的用户帐户，如果帐户没有分配“使用位置”，则重复前面的步骤********。 

    >**备注**：必须设置“使用位置”**** 属性，才能将 Microsoft Entra 高级版 P2 许可证分配给用户帐户。

1. 在“用户 | 所有用户（预览）”边栏选项卡上，选择“aadsyncuser”用户帐户并识别其用户主体名称********。

    >**注意**：记下此值。 本实验室中稍后会用到它。

1. 在 Azure 门户中，导航回 Microsoft Entra 租户的“概述”**** 边栏选项卡，并在左侧垂直菜单的“管理”**** 部分中，单击“许可证”****。
1. 在“许可证 \| 概述”边栏选项卡左侧垂直菜单栏的“管理”部分中，单击“所有产品”************。
1. 在“许可证 \| 所有产品“边栏选项卡上的工具栏中，单击“+ 试用/购买”********。
1. 在“激活”边栏选项卡上，单击“MICROSOFT ENTRA ID P2”部分中的“免费试用版”，单击“激活”，然后按照提示完成激活过程。****************
1. 在“许可证 - 所有产品”**** 边栏选项卡上，选择“企业移动性 + 安全性 E5”**** 条目。 
1. 在“企业移动性 + 安全性 E5”**** 边栏选项卡上的工具栏中，单击“+ 分配”****。
1. 在“分配许可证”边栏选项卡上，单击“添加用户和组”，然后在“添加用户和组”边栏选项卡上，选择“aduser5”和你的用户帐户，并单击“选择”********************。
1. 返回到“分配许可证”**** 边栏选项卡，单击“分配选项”****，在“分配选项”**** 边栏选项卡上，确保已启用所有选项，单击“查看 + 分配”****，然后单击“分配” ****。

#### 任务 2：配置 Microsoft Entra 多重身份验证 (MFA)

1. 在实验室计算机上显示 Azure 门户的 Web 浏览器中，导航回 Microsoft Entra 租户的“概述”**** 边栏选项卡，并在左侧垂直菜单的“管理”**** 部分中，单击“安全”****。
1. 在“安全 | 启动”边栏选项卡上左侧垂直菜单栏的“管理”部分中，单击“标识保护”************。
1. 在“标识保护 | 概述”边栏选项卡上，左侧垂直菜单栏的“保护”部分中，单击“多重身份验证注册策略”（如有必要，请刷新 Web 浏览器页面）************。
1. 在“标识保护 | 多重身份验证注册策略”边栏选项卡上，在“多重身份验证注册策略”的“分配”部分中，单击“所有用户”，在“包括”选项卡上，单击“选择个人和组”选项，在“选择用户”上，依次单击“aduser5”和“选择”，在边栏选项卡底部，将“强制执行策略”切换为“开”，然后单击“保存”************************************************。

#### 任务 3：为用户注册 Microsoft Entra MFA

1. 在实验室计算机上，打开“InPrivate”web 浏览器会话，导航到 [Azure 门户](https://portal.azure.com)，并通过提供此前在此练习中标识的用户主体名称 aduser5 和在创建此用户帐户时设置的密码来登录********。
1. 当显示“需要更多信息”**** 时，单击“下一步”****。 这会自动将浏览器重定向到 Microsoft Authenticator**** 页面。
1. 在“其他安全验证”**** 页上，在 “步骤 1: 如何与你联系？”**** 部分中，选择首选的身份验证方法，并按照说明完成注册过程。 
1. 在 Azure 门户页的右上角，单击用户头像图标，单击“注销”****，然后关闭“In private”**** 浏览器窗口。 

#### 任务 4：配置混合 Microsoft Entra 联接

> **备注**：当根据设备的 Microsoft Entra 联接状态设置条件访问时，可以利用此功能实现额外的安全性。

1. 切换到实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中选择“az140-dc-vm11”  。
1. 在“az140-dc-vm11”边栏选项卡上，选择“连接”，在下拉菜单中选择“通过 Bastion 进行连接”************。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-dc-vm11 的 Bastion 会话中，在“开始”菜单中展开 Azure AD Connect 文件夹，并选择“Azure AD Connect”****************。

   > 注意：如果收到同步服务未运行的故障错误窗口，请转到 PowerShell 命令窗口并输入 Start-Service "ADSync"，然后再次尝试执行上一步********。

1. 在“Microsoft Azure Active Directory Connect”窗口的“欢迎使用 Azure AD Connect”页上，选择“配置”************。
1. 在“Microsoft Azure Active Directory Connect”窗口的“其他任务”页上，选择“配置设备选项”，并选择“下一步”****************。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“概述”页上，查看与“混合 Microsoft Entra 联接”和“设备写回”相关的信息，并选择“下一步”********************。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“连接到 Microsoft Entra”页上，使用在之前的实验室中创建的用户帐户“aadsyncuser”的凭据进行身份验证，然后选择“下一步”****************。  
1. 在“Microsoft Azure Active Directory Connect”窗口中的“设备选项”页上，确保已选择“配置混合 Azure AD 联接”选项，并选择“下一步”****************。 
1. 在“Microsoft Azure Active Directory Connect”窗口中的“设备操作系统”页上，选择“Windows 10 或更高版本的已加入域的设备”复选框，并选择“下一步”****************。 
1. 在“Microsoft Azure Active Directory Connect”窗口中的“SCP 配置”页上，选择“adatum.com”条目旁边的复选框，在“身份验证服务”下拉列表中，选择“Azure Active Directory”条目，并选择“添加”************************。 
1. 出现提示时，在“企业管理员凭据”**** 对话框中，指定以下凭据，然后选择“确定”****：

   |设置|值|
   |---|---|
   |用户名|ADATUM\Student|
   |密码|**Pa55w.rd1234**|

1. 回到“Microsoft Azure Active Directory Connect”窗口的“SCP 配置”页，选择“下一步”************。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“准备配置”页上，选择“配置”，配置完成后选择“退出”****************。
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，以管理员身份启动 Windows PowerShell ISE****。
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，将 az140-cl-vm11**** 计算机帐户移动到 WVDClients**** 组织单位 (OU)：

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. 在与 az140-dc-vm11 的 Bastion 会话中，在“开始”菜单中展开 Azure AD Connect 文件夹，并选择“Azure AD Connect”****************。
1. 在“Microsoft Azure Active Directory Connect”窗口的“欢迎使用 Azure AD Connect”页上，选择“配置”************。
1. 在“Microsoft Azure Active Directory Connect”窗口的“附加任务”页上，依次选择“自定义同步选项”和“下一步”****************。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“连接到 Microsoft Entra”页上，使用在前面的练习中创建的用户帐户“aadsyncuser”的凭据进行身份验证，然后选择“下一步”****************。 
1. 在“Microsoft Azure Active Directory Connect”窗口的“连接到目录”页上，选择“下一步”************。
1. 在“Microsoft Azure Active Directory Connect”窗口中的“域和 OU 筛选”页上，确保已选择“同步已选择的域和 OU”选项，展开“adatum.com”节点，确保已选择“ToSync”OU 旁的复选框，然后选择“WVDClients”OU 旁的复选框，并选择“下一步”****************************。
1. 在“Microsoft Azure Active Directory Connect”窗口的“可选功能”页上，接受默认设置，然后选择“下一步”************。
1. 在“Microsoft Azure Active Directory Connect”窗口的“准备配置”页上，确保已选中“配置完成后启动同步过程”复选框，然后选择“配置”****************。
1. 查看“配置完成”页中的信息，然后选择“退出”关闭“Microsoft Azure Active Directory Connect”窗口。

#### 任务 5：触发 Microsoft Azure Active Directory Connect 完全同步

1. 在实验室计算机上，在 Azure 门户中搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az140-cl-vm11”************ 条目。 此时会打开“az140-cl-vm11”边栏选项卡****。
1. 在“az140-cl-vm11”边栏选项卡中，选择“重启”，然后等待“成功重启虚拟机”通知出现************。
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，切换到“管理员: Windows PowerShell ISE”**** 窗口。
1. 在与 az140-dc-vm11 的 Bastion 会话中，从“管理员: ******Windows PowerShell ISE** 控制台窗格，运行以下命令以触发 Microsoft Azure Active Directory Connect 完全同步：

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，启动 Microsoft Edge，并导航到 [Azure 门户](https://portal.azure.com)。 如果出现提示，请使用在与本实验室所用订阅关联的 Microsoft Entra 租户中具有全局管理员角色的用户帐户的 Microsoft Entra 凭据登录。
1. 在与 az140-dc-vm11 的 Bastion 会话中，在显示 Azure 门户的 Microsoft Edge 窗口中搜索并选择“Microsoft Entra ID”，以导航到与本实验室中所用 Azure 订阅关联的 Microsoft Entra 租户********。
1. 在“Microsoft Entra ID”边栏选项卡上左侧的垂直菜单栏中，在“管理”部分中，单击“设备”********。 
1. 在“设备 | 所有设备”边栏选项卡上，查看设备列表，验证“az140-cl-vm11”设备是否在“联接类型”列中列出了“已建立 Microsoft Entra 混合联接”条目****************。

   > **备注**：同步可能需要等待几分钟才会生效，然后设备才会显示在 Azure 门户中。

### 练习 2：为 Azure 虚拟桌面实现基于 Microsoft Entra 的条件访问

此练习的主要任务如下：

1. 针对所有 Azure 虚拟桌面连接创建基于 Microsoft Entra 的条件访问策略
1. 针对所有 Azure 虚拟桌面连接测试基于 Microsoft Entra 的条件访问策略
1. 修改基于 Microsoft Entra 的条件访问策略，以从 MFA 要求中排除已建立混合 Microsoft Entra 联接的计算机
1. 测试修改后的基于 Microsoft Entra 的条件访问策略

#### 任务 1：针对所有 Azure 虚拟桌面连接创建基于 Microsoft Entra 的条件访问策略

>**备注**：在此任务中，将配置基于 Microsoft Entra 的条件访问策略，该策略要求使用 MFA 登录到 Azure 虚拟桌面会话。 该策略还将在成功进行身份验证后的首个 4 小时过后强制重新执行身份验证。

1. 在实验室计算机上的显示 Azure 门户的 Web 浏览器中，导航回 Microsoft Entra 租户的“概述”**** 边栏选项卡，在左侧垂直菜单的“管理”**** 部分中单击“安全”****。
1. 在“安全 \| 启动”边栏选项卡上左侧垂直菜单栏的“保护”部分中，单击“条件访问”************。
1. 在“条件访问 \| 策略”边栏选项卡上的工具栏中，单击“+ 新建策略”。********
1. 在“新建”边栏选项卡上，配置以下设置：

   - 在“名称”**** 文本框中，键入“az140-31-wvdpolicy1”****
   - 在“分配”部分中，选择“用户或工作负载标识”选项，在“此策略的适用对象”下拉列表中，确保选中“用户和组”，在“选择用户和组”部分，选中“用户和组”复选框，在“选择”边栏选项卡上，单击“aduser5”，然后单击“选择”************************************。
   - 在“分配”部分，单击“云应用或操作”，确保在“选择此策略适用的内容”开关中，选择“云应用”选项，单击“选择应用”选项，在“选择”边栏选项卡上的“搜索”文本框中，输入“Azure 虚拟桌面”，在结果列表中，选中“Azure 虚拟桌面”条目旁的复选框，在“搜索”文本框中，输入“Microsoft 远程桌面”，选中“Microsoft 远程桌面”条目旁的复选框，然后单击“选择”。**************************************************** 

   > 注意****：Azure 虚拟桌面（应用 ID 9cdead84-a844-4324-93f2-b2e6bb768d07）会在用户订阅源并在连接期间向 Azure 虚拟桌面网关进行身份验证时使用。 Microsoft 远程桌面（应用 ID a4a365df-50f1-4397-bc59-1a1564b8bb9c）会在用户在启用单一登录的情况下向会话主机进行身份验证时使用。

   - 在“分配”**** 部分中单击“条件”****，单击“客户端应用”****，在“客户端应用”**** 边栏选项卡上，将“配置”**** 开关设置为“是”****，确保同时选中“浏览器”**** 和“移动应用和桌面客户端”**** 复选框，然后单击“完成”****。
   - 在“访问控制”**** 部分单击“授权”****，然后在“授权”**** 边栏选项卡上，确保选中“授权访问”**** 选项，选中“需要多重身份验证”**** 复选框，然后单击“选择”****。
   - 在“访问控制”**** 部分，单击“会话”****，在“会话”**** 边栏选项卡上，选中“登录频率”**** 复选框，在第一个文本框中键入“4”****，在“选择单位”**** 下拉列表中选择“小时”****，让“持久性浏览器会话”**** 复选框处于清除状态，然后单击“选择”****。
   - 将“启用策略”**** 开关设置为“开启”****。

1. 在“新建”边栏选项卡中，单击“创建”。******** 

#### 任务 2：针对所有 Azure 虚拟桌面连接测试基于 Microsoft Entra 的条件访问策略

1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”窗格内的“PowerShell”shell 会话。
1. 从“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，以启动你将在本实验室中使用的 Azure 虚拟桌面会话主机 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**备注**：等待命令完成并且 az140-21-RG**** 资源组中的所有 Azure VM 都在运行。 

1. 在实验室计算机上，打开“InPrivate”web 浏览器会话，导航到 [Azure 门户](https://portal.azure.com)，并通过提供此前在此练习中标识的用户主体名称 aduser5 和在创建此用户帐户时设置的密码来登录********。

   > **备注**：验证是否未提示你通过 MFA 进行身份验证。

1. 在“InPrivate”Web 浏览器会话中，导航到 Azure 虚拟桌面 HTML5 Web 客户端页面（[https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient)）****。

   > **备注**：验证这是否会自动通过 MFA 触发身份验证。

1. 在“输入代码”窗格中，键入短信或注册的身份验证器应用中的代码，然后选择“验证”********。
1. 在“所有资源”**** 页上，单击“命令提示符”****，在“访问本地资源”**** 窗格上，清除“打印机”**** 复选框，然后单击“允许”****。
1. 在“输入你的凭据”中出现提示时，在“用户名”文本框中键入用户主体名称“aduser5”，在“密码”文本框中，键入在创建此用户帐户时设置的密码并单击“提交”********************。
1. 验证“命令提示符”**** 远程应用是否已成功启动。
1. 在“命令提示符”**** 远程应用窗口中，在命令提示符处键入“注销”****，然后按 Enter**** 键。
1. 返回“所有资源”**** 页，在右上角单击“aduser5”****，在下拉菜单中单击“注销”****，然后关闭“InPrivate”**** Web 浏览器窗口。

#### 任务 3：修改基于 Microsoft Entra 的条件访问策略，以从 MFA 要求中排除已建立混合 Microsoft Entra 联接的计算机

>**备注**：在此任务中，你将修改基于 Microsoft Entra 的条件访问策略，该策略要求使用 MFA 登录到 Azure 虚拟桌面会话，以便源自已建立 Microsoft Entra 联接的计算机的连接不需要 MFA。

1. 在实验室计算机上显示 Azure 门户的浏览器窗口中，在“条件访问 | 策略”边栏选项卡上单击代表 az140-31-wvdpolicy1 策略的条目********。
1. 在“az140-31-wvdpolicy1”**** 边栏选项卡中的“访问控制”**** 部分中，单击“授予”****，在“授予”**** 边栏选项卡上，选择“需要多重身份验证”**** 和“要求已建立混合 Microsoft Entra 联接的设备”**** 复选框，确保“需要一个所选控件”**** 选项处于启用状态，然后单击“选择”****。
1. 在“az140-31-wvdpolicy1”**** 边栏选项卡上，单击“保存”****。

>**备注**：该策略可能需要几分钟的时间才能生效。

#### 任务 4：测试修改后的基于 Microsoft Entra 的条件访问策略

1. 在实验室计算机上的显示 Azure 门户的浏览器窗口中，搜索并选择“虚拟机”****，然后在“虚拟机”**** 边栏选项卡上，选择“az140-cl-vm11”**** 条目。
1. 在“az140-cl-vm11”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-cl-vm11 \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**Student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-cl-vm11**** 的 Bastion 会话中，启动 Microsoft Edge 并导航到 Azure 虚拟桌面 HTML5 Web 客户端页 ([https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient))。

   > **备注**：验证这次是否不会提示通过 MFA 进行身份验证。 这是因为 az140-cl-vm11**** 已建立混合 Microsoft Entra 联接。

1. 在“所有资源”页上，双击“命令提示符”，在“访问本地资源”窗格中，清除“打印机”复选框，然后单击“允许”********************。
1. 在“输入你的凭据”中出现提示时，在“用户名”文本框中键入用户主体名称“aduser5”，在“密码”文本框中，键入在创建此用户帐户时设置的密码并单击“提交”********************。
1. 验证“命令提示符”**** 远程应用是否已成功启动。
1. 在“命令提示符”**** 远程应用窗口中的命令提示符处键入 logoff**** ，然后按 Enter**** 键。
1. 返回“所有资源”**** 页，在右上角单击“aduser5”****，在下拉菜单中，单击“注销”****。
1. 在与 az140-cl-vm11**** 的 Bastion 会话中，单击“开始”**** 按钮，在“开始”**** 按钮正上方的垂直栏中单击表示已登录用户帐户的图标，然后在弹出菜单中单击“注销”****。

### 练习 3：停止并解除分配在实验室中预配和使用的 Azure VM

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
