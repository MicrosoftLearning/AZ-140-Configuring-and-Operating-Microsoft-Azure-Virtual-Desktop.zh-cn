---
lab:
  title: 实验室：使用 Azure 资源管理器模板 (AD DS) 部署主机池和主机
  module: 'Module 2: Implement a WVD Infrastructure'
---

# <a name="lab---deploy-host-pools-and-hosts-by-using-azure-resource-manager-templates"></a>实验室 - 使用 Azure 资源管理器模板部署主机池和主机
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- 本实验室将使用的 Azure 订阅。
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有将在本实验室中使用的 Azure 订阅的所有者或参与者角色，以及与 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”
- 已完成实验室“使用 Azure 门户部署主机池和会话主机 (AD DS)”

## <a name="estimated-time"></a>预计用时

45 分钟

## <a name="lab-scenario"></a>实验室方案

你需要使用 Azure 资源管理器模板自动部署 Azure 虚拟桌面主机池和主机。

## <a name="objectives"></a>目标
  
完成本实验室后，你将能够：

- 使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机

## <a name="lab-files"></a>实验室文件 

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## <a name="instructions"></a>说明

### <a name="exercise-1-deploy-azure-virtual-desktop-host-pools-and-hosts-by-using-azure-resource-manager-templates"></a>练习 1：使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机
  
此练习的主要任务如下：

1. 准备使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池
1. 使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机
1. 验证 Azure 虚拟桌面主机池和主机的部署
1. 使用 Azure 资源管理器模板准备将主机添加到现有的 Azure 虚拟桌面
1. 使用 Azure 资源管理器模板将主机添加到现有的 Azure 虚拟桌面主机池
1. 验证对 Azure 虚拟桌面主机池的更改
1. 管理 Azure 虚拟桌面主机池中的个人桌面分配

#### <a name="task-1-prepare-for-deployment-of-an-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>任务 1：准备使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az140-dc-vm11”  。
1. 在“az140-dc-vm11”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-dc-vm11 \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-dc-vm11 的远程桌面会话中，以管理员身份启动 Windows PowerShell ISE 。
1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”控制台，运行以下命令以标识名为 WVDInfra 的组织单位的可分辨名称，该组织单位将托管 Azure 虚拟桌面池主机的计算机对象：

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格，运行以下命令以标识用于将 Azure 虚拟桌面主机加入 AD DS 域 (student@adatum.com) 的 ADATUM\\Student 帐户的用户主体名称属性 ：

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格，运行以下命令以标识稍后将在本实验室中用于测试个人桌面分配的 ADATUM\\aduser7 和 ADATUM\\aduser8 帐户的用户主体名称 ：

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **注意**：记录标识的所有用户主体名称值。 稍后将在本实验室用到它们。

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格，运行以下命令以计算执行基于模板的部署所需的令牌到期时间：

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **注意**：值应类似于格式 `2022-03-27T00:51:28.3008055Z`。 记录该值，因为你在下一个任务中需要使用它。

   > **注意**：需要注册令牌才能授权主机加入池。 令牌的到期日期值必须介于从当前日期和时间起一小时到一个月之间。

1. 在与 az140-dc-vm11 的远程桌面会话中，启动 Microsoft Edge，并导航到 [Azure 门户](https://portal.azure.com)。 如果出现提示，请使用在本实验室使用的订阅中具有所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在与 az140-dc-vm11 的远程桌面会话中，在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“虚拟网络”，然后在“虚拟网络”边栏选项卡中选择“az140-adds-vnet11”    。 
1. 在“az140-adds-vnet11”边栏选项卡上，选择“子网”，在“子网”边栏选项卡中选择“+ 子网”，在“添加子网”边栏选项卡上，指定以下设置（所有其他设置保留默认值），并单击“保存”     ：

   |设置|值|
   |---|---|
   |名称|hp2-Subnet|
   |子网地址范围|**10.0.2.0/24**|

1. 在与 az140-dc-vm11 的远程桌面会话中，在 Azure 门户中，使用 Azure 门户页顶部的“搜索资源、服务和文档”文本框，搜索并导航到“网络安全组”，然后在“网络安全组”边栏选项卡中选择“az140-11-RG”资源组中的网络安全组    。
1. 在“网络安全组”边栏选项卡上的左侧垂直菜单中，在“设置”部分中，单击“属性” 。
1. 在“属性”边栏选项卡中，单击“资源 ID”文本框右侧的“复制到剪贴板”图标  。 

   > **注意**：该值应类似于格式 `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`，尽管订阅 ID 会有所不同。 记录该值，因为你在下一个任务中需要使用它。

#### <a name="task-2-deploy-an-azure-virtual-desktop-host-pool-and-hosts-by-using-an-azure-resource-manager-template"></a>任务 2：使用 Azure 资源管理器模板部署 Azure 虚拟桌面主机池和主机

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在实验室计算机显示的同一 Web 浏览器窗口中，打开另一个 Web 浏览器选项卡并导航到 GitHub Azure RDS 模板存储库页面[用于创建和预配新 Azure 虚拟桌面主机池的 ARM 模板](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool)。 
1. 在“用于创建和预配新 Azure 虚拟桌面主机池的 ARM 模板”页面上，选择“部署到 Azure” 。 这会将浏览器自动重定向到 Azure 门户中的“自定义部署”边栏选项卡。
1. 在“自定义部署”边栏选项卡上，选择“编辑参数” 。
1. 在“编辑参数”边栏选项卡上，选择“加载文件”，在“打开”对话框中，选择 \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json，选择“打开”，然后选择“保存”     。 
1. 返回“自定义部署”边栏选项卡，指定以下设置（其他设置保留现有值）：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|新资源组名称 az140-23-RG|
   |区域|实验室“准备部署 Azure 虚拟桌面 (AD DS)”中向其部署托管 AD DS 域控制器的 Azure VM 的 Azure 区域的名称|
   |位置|设置为 Region 参数值的同一 Azure 区域的名称|
   |工作区位置|设置为 Region 参数值的同一 Azure 区域的名称|
   |工作区资源组|无，如果为空，其值将自动设置为匹配部署目标资源组|
   |所有应用程序组引用|无，因为目标工作区中没有现有应用程序组（没有工作区）|
   |VM 位置|与设置为 Location 参数值的 Azure 区域相同的名称|
   |创建网络安全组|**false**|
   |网络安全组 ID|在上一个任务中标识的现有网络安全组的 resourceID 参数值|
   |令牌到期时间| 在上一个任务中计算的令牌到期时间值|

   > **注意**：部署预配具有个人桌面分配类型的池。

1. 在“自定义部署”边栏选项卡上，选择“查看 + 创建”，然后选择“创建”  。

   > 注意：在继续下一个任务之前，请等待部署完成。 这可能需要大约 15 分钟。 

#### <a name="task-3-verify-deployment-of-the-azure-virtual-desktop-host-pool-and-hosts"></a>任务 3：验证 Azure 虚拟桌面主机池和主机的部署

1. 在实验室计算机中显示 Azure 门户的 Web 浏览器中，搜索并选择“Azure 虚拟桌面”，在“Azure 虚拟桌面”边栏选项卡上，选择“主机池”，然后在“Azure 虚拟桌面 \| 主机池”边栏选项卡上，选择表示新部署池的条目“az140-23-hp2”    。
1. 在“az140-23-hp2”边栏选项卡左侧垂直菜单中的“管理”部分中，单击“会话主机”  。 
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡上，验证该部署是否包含两个主机。
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡左侧垂直菜单中的“管理”部分，单击“应用程序组”  。
1. 在“az140-23-hp2 \| 应用程序组”边栏选项卡上，验证部署是否包括名为 az140-23-hp2-DAG 的“默认桌面”应用程序组  。

#### <a name="task-4-prepare-for-adding-of-hosts-to-the-existing-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>任务 4：使用 Azure 资源管理器模板准备将主机添加到现有的 Azure 虚拟桌面

1. 在实验室计算机上，切换到与 az140-dc-vm11 的远程桌面会话。 
1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”控制台，运行以下命令以生成将新主机加入在本练习前面配置的池中所需的令牌：

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”控制台，运行以下命令以检索令牌的值并将其粘贴到剪贴板中：

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **注意**：记录复制到剪贴板的值（例如，启动记事本，并按 Ctrl+V 组合键将剪贴板的内容粘贴到记事本），因为在下一个任务中需要使用它。 确保所使用的值包含一行文本，不带任何换行符。 

   > **注意**：需要注册令牌才能授权主机加入池。 令牌的到期日期值必须介于从当前日期和时间起一小时到一个月之间。

#### <a name="task-5-add-hosts-to-the-existing-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>任务 5：使用 Azure 资源管理器模板将主机添加到现有的 Azure 虚拟桌面主机池

1. 在实验室计算机显示的同一 Web 浏览器窗口中，打开另一个 Web 浏览器选项卡并导航到 GitHub Azure RDS 模板存储库页面[用于将会话主机添加到现有 Azure 虚拟桌面主机池的 ARM 模板](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool)。 
1. 在“用于将会话主机添加到现有 Azure 虚拟桌面主机池的 ARM 模板”页面上，选择“部署到 Azure” 。 这会将浏览器自动重定向到 Azure 门户中的“自定义部署”边栏选项卡。
1. 在“自定义部署”边栏选项卡上，选择“编辑参数” 。
1. 在“编辑参数”边栏选项卡上，选择“加载文件”，在“打开”对话框中，选择 \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json，选择“打开”，然后选择“保存”     。 
1. 返回“自定义部署”边栏选项卡，指定以下设置（其他设置保留现有值）：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|az140-23-RG|
   |主机池令牌|在上一个任务中生成的令牌值|
   |主机池位置|本实验室初期向其部署主机池的 Azure 区域的名称|
   |VM 管理员帐户用户名|student 请不要使用 @adatum.com|
   |VM 管理员帐户密码|**Pa55w.rd1234**|
   |VM 位置|与设置为 Hostpool Location 参数值的 Azure 区域相同的名称|
   |创建网络安全组|**false**|
   |网络安全组 ID|在上一个任务中标识的现有网络安全组的 resourceID 参数值|

1. 在“自定义部署”边栏选项卡上，选择“查看 + 创建”，然后选择“创建”  。

   > **注意**：在继续下一个任务之前，请等待部署完成。 这可能需要大约 5 分钟。

#### <a name="task-6-verify-changes-to-the-azure-virtual-desktop-host-pool"></a>任务 6：验证对 Azure 虚拟桌面主机池的更改

1. 在实验室计算机显示 Azure 门户的 Web 浏览器中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡上，注意该列表包括一个名为 az140-23-p2-2 的附加虚拟机  。
1. 在实验室计算机上，切换到与 az140-dc-vm11 的远程桌面会话。 
1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”控制台，运行以下命令以验证第三方主机是否成功加入 adatum.com AD DS 域：

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. 切换回实验室计算机，在显示 Azure 门户的 Web 浏览器中，搜索并选择“Azure 虚拟桌面”，在“Azure 虚拟桌面”边栏选项卡上，选择“主机池”，然后在“Azure 虚拟桌面 \| 主机池”边栏选项卡上，选择表示新修改池的条目“az140-23-hp2”    。
1. 在“az140-23-hp2”边栏选项卡上，查看“必须”部分并确认“主机池类型”设置为“个人”，并将“分配类型”设置为“自动”     。
1. 在“az140-23-hp2”边栏选项卡左侧垂直菜单中的“管理”部分中，单击“会话主机”  。 
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡上，验证该部署是否包含三个主机。 

#### <a name="task-7-manage-personal-desktop-assignments-in-the-azure-virtual-desktop-host-pool"></a>任务 7：管理 Azure 虚拟桌面主机池中的个人桌面分配

1. 在实验室计算机中，在显示Azure 门户的 Web 浏览器中，在“az140-23-hp2 \| 会话主机”边栏选项卡上，在左侧垂直菜单中的“管理”部分，选择“应用程序组”  。 
1. 在“az140-23-hp2 \| 应用程序组”边栏选项卡的应用程序组列表中，选择“az140-23-hp2-DAG” 。
1. 在“az140-23-hp2-DAG”边栏选项卡左侧垂直菜单中，选择“分配” 。 
1. 在“az140-23-hp2-DAG \| 分配”边栏选项卡上，选择“+ 添加” 。
1. 在“选择 Azure AD 用户或用户组”边栏选项卡上，选择“az140-wvd-personal”，然后单击“选择”  。

   > **注意**：现在我们回顾一下连接到 Azure 虚拟桌面主机池的用户体验。

1. 在实验室计算机显示 Azure 门户的浏览器窗口中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡上，选择“az140-cl-vm11”条目  。
1. 在“az140-cl-vm11”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-cl-vm11 \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**Student@adatum.com**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-cl-vm11 的远程桌面会话中，单击“开始”，然后在“开始”菜单中选择“远程桌面”客户端应用   。
2. 在远程桌面窗口中，单击右上角的省略号图标，在下拉菜单中单击“取消订阅”，并在系统提示确认时单击“继续” 。
3. 在与 az140-cl-vm11 的远程桌面会话中，在“远程桌面窗口”的“让我们开始吧”页面上，单击“订阅”  。
4. 在“远程桌面”客户端窗口中，选择“订阅”，并在系统出现提示时，通过提供凭据的 userPrincipalName 并将 Pa55w.rd1234 作为其密码，使用 aduser7 凭据进行登录   。

   > **注意**：或者，在“远程桌面”客户端窗口中，选择“通过 URL 订阅”，在“订阅工作区”窗格中的“电子邮件或工作区 URL”中，键入 https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery ，选择“下一步”，然后在出现提示时，使用 aduser7 凭据（使用其 userPrincipalName 属性作为用户名和在创建此帐户时设置的密码）进行登录      。 

1. 在“远程桌面”页面上，双击“SessionDesktop”图标，当系统提示输入凭据时，再次键入同一密码，并选中“记住我”复选框，然后单击“确定”   。
1. 在“保持登录到所有应用”窗口中，清除“允许组织管理我的设备”复选框，然后选择“否，仅登录到此应用”  。 
1. 验证 aduser7 是否已通过远程桌面成功登录到主机。
1. 在与其中一台主机 aduser7 的远程桌面会话中，右键单击“开始”，在右键单击菜单中，选择“关闭或注销”，然后在级联菜单中单击“注销”   。

   > **注意**：现在我们将个人桌面分配从直接模式切换到自动模式。 

1. 切换到实验室计算机，转到显示 Azure 门户的 Web 浏览器，在“az140-23-hp2-DAG \| 分配”边栏选项卡上，在分配列表上方的信息栏中，单击“分配 VM”链接 。 这会重定向到“az140-23-hp2 \| 会话主机”边栏选项卡。 
1. 在“az140-23-hp2 \| 会话主机”边栏选项卡上，确认其中一台主机在“分配的用户”列中列出了 aduser7  。

   > **注意**：这在意料之中，因为主机池配置为自动分配。

1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”窗格内的“PowerShell”shell 会话。
1. 从 Cloud Shell 窗格中的 PowerShell 会话，运行以下命令以切换到直接分配模式：

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，导航至 az140-23-hp2 主机池边栏选项卡，查看“必须”部分并确认“主机池类型”设置为“个人”，并将“分配类型”设置为“直接”     。
1. 切换回与 az140-cl-vm11 的远程桌面会话，在“远程桌面”窗口中，单击右上角的省略号图标，在下拉菜单中，单击“取消订阅”，并在出现确认提示时，单击“继续”   。
1. 在与 az140-cl-vm11 的远程桌面会话中，在“远程桌面窗口”的“让我们开始吧”页面上，单击“订阅”   。
1. 当系统提示登录时，在“选择帐户”窗格中，单击“使用其他帐户”，并在出现提示时使用 aduser8 用户帐户的用户主体名称和创建此帐户时设置的密码进行登录  。
1. 在“保持登录到所有应用”窗口中，清除“允许组织管理我的设备”复选框，然后选择“否，仅登录到此应用”  。 
1. 在“远程桌面”页面上，双击“SessionDesktop”图标，确认收到一条指示“无法连接，因为当前没有可用资源 。如果这种情况频繁发生，请稍后再试或联系技术支持人员寻求帮助”的错误消息，然后单击“确定”。

   > **注意**：这在意料之中，因为主机池配置为直接分配，但 aduser8 尚未分配有主机。

1. 切换到实验室计算机，转到显示 Azure 门户的 Web 浏览器，在“az140-23-hp2 \| 会话主机”边栏选项卡上，在剩余两个未分配主机之一旁边的“分配的用户”列中选择“(分配)”链接  。
1. 在“分配用户”上，选择 aduser8，单击“选择”，并在出现确认提示时单击“确定”   。
1. 切换回与 az140-cl-vm11 的远程桌面会话，在“远程桌面”窗口中，双击“SessionDesktop”图标，当提示输入密码时，键入在创建此用户帐户时设置的密码，单击“确定”，并验证是否可以成功登录到分配的主机   。

### <a name="exercise-2-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>练习 2：停止并解除分配在实验室中预配的 Azure VM

此练习的主要任务如下：

1. 停止并解除分配在实验室中预配的 Azure VM

>**注意**：在此练习中，你将解除分配此实验室中预配的 Azure VM，以最大程度减少相应的计算费用

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>任务 1：解除分配在实验室中预配的 Azure VM

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