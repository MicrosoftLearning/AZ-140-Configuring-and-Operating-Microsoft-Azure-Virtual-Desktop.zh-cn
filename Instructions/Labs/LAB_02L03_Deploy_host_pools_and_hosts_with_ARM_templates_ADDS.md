---
lab:
  title: 实验室：使用 Azure 资源管理器模板 (AD DS) 部署主机池和主机
  module: 'Module 2: Implement an AVD Infrastructure'
---

# 实验室 - 使用 Azure 资源管理器模板部署主机池和主机
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- Microsoft 帐户或 Microsoft Entra 帐户，该帐户在本实验室将会使用的 Azure 订阅中具有所有者或参与者角色，在与该 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色。
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”
- 已完成实验室“使用 Azure 门户部署主机池和会话主机 (AD DS)”

## 预计用时

45 分钟

## 实验室方案

需要使用 Azure 资源管理器模板自动部署 Azure 虚拟桌面主机池和主机。

## 目标
  
完成本实验室后，你将能够：

- 使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机

## 实验室文件

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## 说明

### 练习 1：使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机
  
此练习的主要任务如下：

1. 准备使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池
1. 使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机
1. 验证 Azure 虚拟桌面主机池和主机的部署
1. 准备使用 Azure 资源管理器模板将主机添加到现有 Azure 虚拟桌面主机池
1. 使用 Azure 资源管理器模板将主机添加到现有 Azure 虚拟桌面主机池
1. 验证对 Azure 虚拟桌面主机池的更改
1. 在 Azure 虚拟桌面主机池中管理个人桌面分配

#### 任务 1：准备使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az140-dc-vm11”  。
1. 在“az140-dc-vm11”边栏选项卡上，选择“连接”，在下拉菜单中选择“通过 Bastion 进行连接”************。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，以管理员身份启动 Windows PowerShell ISE****。
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台中运行以下命令，以标识名为“WVDInfra”**** 的组织单位的可分辨名称，该组织单位将托管 Azure 虚拟桌面池主机的计算机对象：

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以标识用于将 Azure 虚拟桌面主机添加到 AD DS 域 (**student@adatum.com**) 的“ADATUM\\Student”**** 帐户的用户主体名称属性：

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以标识稍后将在本实验室中用于测试个人桌面分配的“ADATUM\\aduser7”**** 和“ADATUM\\aduser8”**** 帐户的用户主体名称：

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **注意**：记录确定的所有用户主体名称值**和** WVDInfra OU 的可分辨名称。 稍后将在本实验室用到它们。

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 脚本窗格运行以下命令，以计算执行基于模板的部署所需的令牌到期时间：

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **备注**：该值的格式应类似于 `2022-03-27T00:51:28.3008055Z`。 请将其记录下来，因为会在下一个任务中用到它。

   > **备注**：授权主机加入池需要使用注册令牌。 令牌到期日期的值必须介于当前日期和时间之后的 1 小时到 1 个月之间。

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，启动 Microsoft Edge，并导航到 [Azure 门户](https://portal.azure.com)。 如果出现提示，请使用在本实验室所用订阅中具有所有者角色的用户帐户的 Microsoft Entra 凭据登录。
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”**** 文本框搜索并导航到“虚拟网络”****，然后在“虚拟网络”**** 边栏选项卡中选择“az140-adds-vnet11”****。 
1. 在“az140-adds-vnet11”边栏选项卡上，选择“子网”，在“子网”边栏选项卡中选择“+ 子网”，在“添加子网”边栏选项卡上，指定以下设置（所有其他设置保留默认值），并单击“保存”     ：

   |设置|值|
   |---|---|
   |名称|hp2-Subnet****|
   |子网地址范围|**10.0.2.0/24**|

1. 在与**az140-dc-vm11**的 Bastion 会话中，在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“网络安全组”，然后在“网络安全组”边栏选项卡中选择唯一的网络安全组************。
1. 在“网络安全组”边栏选项卡上的左侧垂直菜单中，单击“设置”**** 部分中的“属性”****。
1. 在“属性”**** 边栏选项卡上，单击“资源 ID”**** 文本框右侧的“复制到剪贴板”**** 图标。 

   > **备注**：该值的格式应类似于 `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`，尽管订阅 ID 会有所不同。 请将其记录下来，因为会在下一个任务中用到它。
1. 现在应已记录**六个**值。 可分辨名称、3 个用户主体名称、DateTime 值和资源 ID。 如果未记录 6 个值，请在继续**之前**再次阅读此任务。 

#### 任务 2：使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在实验室计算机上的同一个 Web 浏览器窗口中，打开另一个 Web 浏览器标签页，导航到 GitHub Azure RDS 模板存储库页[用于创建和预配新 Azure 虚拟桌面主机池的 ARM 模板](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool)。 
1. 在**用于创建和预配新 Azure 虚拟桌面主机池的 ARM 模板**页上，选择“部署到 Azure”****。 这会自动将浏览器重定向到 Azure 门户中的“自定义部署”**** 边栏选项卡。
1. 在“自定义部署”**** 边栏选项卡上，选择“编辑参数”****。
1. 在“编辑参数”边栏选项卡上，选择“加载文件”，在“打开”对话框中，选择 \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json，选择“打开”，然后选择“保存”************************。 
1. 返回“自定义部署”**** 边栏选项卡，指定以下设置（其他设置保留现有值）：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|**新建**名为**az140-23-RG**的资源组|
   |区域|实验室“准备部署 Azure 虚拟桌面 (AD DS)”中向其部署托管 AD DS 域控制器的 Azure VM 的 Azure 区域的名称****|
   |位置|与设为“区域”**** 参数的值相同的 Azure 区域名称|
   |工作区位置|与设为“区域”**** 参数的值相同的 Azure 区域名称|
   |工作区资源组|无，因为如果为 null，则其值将自动设置为与部署目标资源组相匹配|
   |所有应用程序组引用|无，因为目标工作区中没有现成的应用程序组（没有工作区）|
   |VM 位置|与设为“位置”**** 参数的值相同的 Azure 区域名称|
   |创建网络安全组|**false**|
   |网络安全组 ID|在上一个任务中标识的现有网络安全组的 resourceID 参数的值|
   |令牌过期时间| 在上一个任务中计算的令牌过期时间的值|

   > **备注**：该部署使用个人桌面分配类型来预配池。

1. 在“自定义部署”**** 边栏选项卡上，选择“查看 + 创建”****，然后选择“创建”****。

   > 注意：在继续下一个任务之前，请等待部署完成。 这可能需要大约 15 分钟。 

#### 任务 3：验证 Azure 虚拟桌面主机池和主机的部署

1. 在实验室计算机中显示 Azure 门户的 Web 浏览器中，搜索并选择“Azure 虚拟桌面”，在“Azure 虚拟桌面”边栏选项卡上，选择“主机池”，然后在“Azure 虚拟桌面 \| 主机池”边栏选项卡上，选择表示新部署池的条目“az140-23-hp2”********************。
1. 在“az140-23-hp2”**** 边栏选项卡上的左侧垂直菜单中的“管理”**** 部分中，单击“会话主机”****。 
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡上，验证该部署是否包含两个主机****。
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡左侧垂直菜单中的“管理”部分，单击“应用程序组”************。
1. 在“az140-23-hp2 \| 应用程序组”边栏选项卡上，验证部署是否包括名为 az140-23-hp2-DAG 的“默认桌面”应用程序组************。

#### 任务 4：准备使用 Azure 资源管理器模板将主机添加到现有 Azure 虚拟桌面主机池

1. 从实验室计算机切换到与 az140-dc-vm11**** 的 Bastion 会话。 
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，以生成将新主机添加到先前在本练习中配置的池中所需的令牌：

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，以检索令牌的值并将其粘贴到剪贴板中：

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **备注**：记录复制到剪贴板的值（例如，通过启动记事本并按 Ctrl+V 组合键将剪贴板的内容粘贴到记事本），因为下一个任务中需要用到它。 请确保所使用的值只包含一行文本，而不包含任何换行符。 

   > **备注**：授权主机加入池需要使用注册令牌。 令牌到期日期的值必须介于当前日期和时间之后的 1 小时到 1 个月之间。

#### 任务 5：使用 Azure 资源管理器模板将主机添加到现有 Azure 虚拟桌面主机池

1. 在实验室计算机上的同一个 Web 浏览器窗口中，打开另一个 Web 浏览器标签页，导航到 GitHub Azure RDS 模板存储库页[用于将会话主机添加到现有 Azure 虚拟桌面主机池的 ARM 模板](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool)。 
1. 在**用于将会话主机添加到现有 Azure 虚拟桌面主机池的 ARM 模板**页上，选择“部署到 Azure”****。 这会将浏览器自动重定向到 Azure 门户中的“自定义部署”**** 边栏选项卡。
1. 在“自定义部署”**** 边栏选项卡上，选择“编辑参数”****。
1. 在“编辑参数”边栏选项卡上，选择“加载文件”，在“打开”对话框中，选择 \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json，选择“打开”，然后选择“保存”************************。 
1. 返回“自定义部署”**** 边栏选项卡，指定以下设置（其他设置保留现有值）：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|az140-23-RG****|
   |主机池令牌|在上一个任务中生成的令牌的值|
   |主机池位置|先前在本实验室中将虚拟机部署到的 Azure 区域的名称|
   |VM 位置|与设为“主机池位置”**** 参数的值相同的 Azure 区域名称|
   |创建网络安全组|**false**|
   |网络安全组 ID|在上一个任务中标识的现有网络安全组的 resourceID 参数的值|

1. 在“自定义部署”**** 边栏选项卡上，选择“查看 + 创建”****，然后选择“创建”****。

   > 注意：在继续下一个任务之前，请等待部署完成。 这可能需要大约 10 分钟。

#### 任务 6：验证对 Azure 虚拟桌面主机池的更改

1. 在实验室计算机上的显示 Azure 门户的 Web 浏览器中，搜索并选择“虚拟机”****，然后在“虚拟机”**** 边栏选项卡中找到额外包含一个名为“az140-23-p2-2”**** 的虚拟机的列表。
1. 从实验室计算机切换到与 az140-dc-vm11**** 的 Bastion 会话。 
1. 在与 az140-dc-vm11**** 的 Bastion 会话中，从“管理员: Windows PowerShell ISE”**** 控制台运行以下命令，以验证第三方会话主机是否成功加入了 AD DS 域 adatum.com****：

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. 切换回实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择“Azure 虚拟桌面”，在“Azure 虚拟桌面”边栏选项卡上，选择“主机池”，然后在“Azure 虚拟桌面 \| 主机池”边栏选项卡上，选择表示新修改池的条目“az140-23-hp2”********************。
1. 在“az140-23-hp2”**** 边栏选项卡上，查看“概要”**** 部分，并验证“主机池类型”**** 是否设置为“个人”****，并将“分配类型”**** 设置为“自动”****。
1. 在“az140-23-hp2”**** 边栏选项卡上的左侧垂直菜单中的“管理”**** 部分中，单击“会话主机”****。 
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡上，验证该部署是否包含三个主机****。 

#### 任务 7：在 Azure 虚拟桌面主机池中管理个人桌面分配

1. 在实验室计算机中，在显示Azure 门户的 Web 浏览器中，在“az140-23-hp2 \| 会话主机”边栏选项卡上，在左侧垂直菜单中的“管理”部分，选择“应用程序组”************。 
1. 在“az140-23-hp2 \| 应用程序组”边栏选项卡的应用程序组列表中，选择“az140-23-hp2-DAG”********。
1. 在“az140-23-hp2-DAG”**** 边栏选项卡上的左侧垂直菜单中，选择“分配”****。 
1. 在“az140-23-hp2-DAG \| 分配”边栏选项卡上，选择“+ 添加”********。
1. 在“选择 Microsoft Entra 用户或用户组”边栏选项卡上，依次选择“组”、“az140-wvd-personal”，然后点击“选择”****************。

   > **备注**：现在让我们检查一下连接到 Azure 虚拟桌面主机池的用户的体验。

1. 在实验室计算机上的显示 Azure 门户的浏览器窗口中，搜索并选择“虚拟机”****，然后在“虚拟机”**** 边栏选项卡上，选择“az140-cl-vm11”**** 条目。
1. 在“az140-cl-vm11”边栏选项卡上，选择“连接”，在下拉菜单中选择“通过 Bastion 进行连接”************。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**Student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-cl-vm11**** 的 Bastion 会话中，单击“开始”****，然后在“开始”**** 菜单中，选择“远程桌面”**** 客户端应用。
2. 在远程桌面窗口中，单击右上角的省略号图标，在下拉菜单中单击“取消订阅”，并在系统提示确认时单击“继续”********。
3. 在与 az140-cl-vm11**** 的 Bastion 会话中，在“远程桌面”窗口的“让我们开始吧”**** 页上，单击“订阅”****。
4. 在“远程桌面”**** 客户端窗口中，选择“订阅”****，然后在出现提示时，通过提供 userPrincipalName 属性值和之前在创建用户帐户时设置的密码，使用 aduser7**** 的凭据进行登录。

   > **备注**：或者，在“远程桌面”**** 客户端窗口中，选择“通过 URL 订阅”****，在“订阅工作区”**** 窗格中的“电子邮件或工作区 URL”**** 中，键入 **https://client.wvd.microsoft.com/api/arm/feeddiscovery**，选择“下一步”****，然后在出现提示时，使用 aduser7 **** 的凭据（使用其 userPrincipalName 属性作为用户名，并使用在创建此帐户时设置的密码）进行登录。 

1. 在“远程桌面”页面上，双击“SessionDesktop”图标，当系统提示输入凭据时，再次键入同一密码，并选中“记住我”复选框，然后单击“确定”****************。
1. 验证 aduser7**** 是否已通过远程桌面成功登录到主机。
1. 在以 aduser7**** 的身份与其中一个主机建立的远程桌面会话中，右键单击“开始”****，在右键单击菜单中选择“关闭或注销”****，然后在级联菜单中单击“注销”****。

   > **备注**：现在，让我们将个人桌面分配从直接模式切换到自动。 

1. 切换到实验室计算机，转到显示 Azure 门户的 Web 浏览器，在“az140-23-hp2-DAG \| 分配”边栏选项卡上，在分配列表上方的信息栏中，单击“分配 VM”链接********。 这会重定向到“az140-23-hp2 \| 会话主机”边栏选项卡****。 
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡上，确认其中一台主机在“分配的用户”列中列出了 aduser7************。

   > **备注**：这是预期情况，因为主机池配置为自动分配。

1. 在实验室计算机上的显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”**** 窗格内的“PowerShell”**** shell 会话。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以切换到直接分配模式：

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. 在实验室计算机上的显示 Azure 门户的 Web 浏览器窗口中，导航到“az140-23-hp2”**** 主机池边栏选项卡，查看“概要”**** 部分，并验证“主机池类型”**** 是否设置为“个人”****，“分配类型”**** 是否设置为“直接”****。
1. 切换回与 az140-cl-vm11**** 的 Bastion 会话，在“远程桌面”**** 窗口中，单击右上角的省略号图标，在下拉菜单中，单击“取消订阅”****，并提示确认时，单击“继续”****。
1. 在与 az140-cl-vm11**** 的 Bastion 会话中，在“远程桌面”窗口**** 的“让我们开始吧”**** 页上，单击“订阅”****。
1. 当系统提示登录时，在“选择帐户”窗格中，单击“使用其他帐户”，并在出现提示时使用 aduser8 用户帐户的用户主体名称和创建此帐户时设置的密码进行登录************。
1. 在“远程桌面”**** 页上，双击“SessionDesktop”**** 图标，确认是否收到一条错误消息，指出“无法连接，因为当前没有可用的资源。请稍后再试，如果持续存在这种情况，请联系技术支持寻求帮助”****，然后单击“确定”****。

   > **备注**：这是预期情况，因为主机池配置为直接分配，并且还没有为 aduser8**** 分配主机。

1. 切换到实验室计算机，转到显示 Azure 门户的 Web 浏览器，在“az140-23-hp2 \| 会话主机”边栏选项卡上，在剩余两个未分配主机之一旁边的“分配的用户”列中选择“(分配)”链接************。
1. 在“分配用户”**** 上，选择“aduser8”****，单击“选择”****，然后在系统提示确认时单击“确定”****。
1. 切换回与 az140-cl-vm11**** 的 Bastion 会话，在“远程桌面”**** 窗口中，双击“SessionDesktop”**** 图标，当提示输入密码时，键入在创建此用户帐户时设置的密码，单击“确定”****，并验证是否可以成功登录到分配的主机。
1. 在**aduser8**分配主机的会话桌面中，右击“开始”，在右击菜单中，选择“关闭或注销”，然后在级联菜单中，点击“注销”************。
1. 在**az140-cl-vm11**的 Bastion 会话中，右击“开始”，在右击菜单中，选择“关闭或注销”，然后在级联菜单中，点击“注销”，然后点击“关闭”****************。

### 练习 2：停止并解除分配在实验室中预配的 Azure VM

此练习的主要任务如下：

1. 停止并解除分配在实验室中预配的 Azure VM

>**注意**：在此练习中，你将解除分配此实验室中预配的 Azure VM，以最大程度减少相应的计算费用

#### 任务 1：解除分配在实验室中预配的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开 Cloud Shell 窗格内的“PowerShell”shell 会话 。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注意**：该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能停止和解除分配 Azure VM。
