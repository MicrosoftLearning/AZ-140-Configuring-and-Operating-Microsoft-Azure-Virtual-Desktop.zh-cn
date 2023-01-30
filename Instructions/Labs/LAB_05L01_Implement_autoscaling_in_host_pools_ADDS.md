---
lab:
  title: 实验室：在主机池中实现自动扩展 (AD DS)
  module: 'Module 5: Monitor and Maintain a WVD Infrastructure'
---

# <a name="lab---implement-autoscaling-in-host-pools-ad-ds"></a>实验室 - 在主机池中实现自动缩放 (AD DS)
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- 本实验室将使用的 Azure 订阅。
- 一个 Microsoft 帐户或 Azure AD 帐户，该帐户具有将在本实验室中使用的 Azure 订阅的所有者或参与者角色，以及与 Azure 订阅关联的 Azure AD 租户的全局管理员角色。
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”
- 已完成实验室“使用 Azure 门户部署主机池和会话主机 (AD DS)”

## <a name="estimated-time"></a>预计用时

60 分钟

## <a name="lab-scenario"></a>实验室方案

你需要在 Active Directory 域服务 (AD DS) 环境中配置 Azure 虚拟桌面会话主机的自动缩放功能。

## <a name="objectives"></a>目标
  
完成本实验室后，你将能够：

- 配置 Azure 虚拟桌面会话主机的自动缩放功能
- 验证 Azure 虚拟桌面会话主机的自动缩放功能

## <a name="lab-files"></a>实验室文件 

- 无

## <a name="instructions"></a>说明

### <a name="exercise-1-configure-autoscaling-of-azure-virtual-desktop-session-hosts"></a>练习 1：配置 Azure 虚拟桌面会话主机的自动缩放功能

此练习的主要任务如下：

1. 准备自动缩放 Azure 虚拟桌面会话主机
1. 创建并配置一个 Azure 自动化帐户
1. 创建 Azure 逻辑应用

#### <a name="task-1-prepare-for-autoscaling-of-azure-virtual-desktop-session-hosts"></a>任务 1：准备自动缩放 Azure 虚拟桌面会话主机

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在实验室计算机显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”窗格内的“PowerShell”shell 会话。
1. 从“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，以启动你将在本实验室中使用的 Azure 虚拟桌面会话主机 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**注意**：该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能启动 Azure VM。 

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>任务 2：创建并配置一个 Azure 自动化帐户

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡中，选择“az140-dc-vm11”  。
1. 在“az140-dc-vm11”边栏选项卡中，选择“连接”，在下拉菜单中选择“Bastion”，在“az140-dc-vm11 \| 连接”边栏选项卡的“Bastion”选项卡中，选择“使用 Bastion”     。
1. 出现提示时，提供以下凭据并选择“连接”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 在与 az140-dc-vm11 的远程桌面会话中，以管理员身份启动 Windows PowerShell ISE 。
1. 在“管理员:Windows PowerShell ISE”控制台中运行以下命令，以登录 Azure 订阅：

   ```powershell
   Connect-AzAccount
   ```

1. 如果出现提示，请使用在具有本实验室所用订阅的所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格运行以下脚本，以下载 PowerShell 脚本，你将使用该脚本创建作为自动缩放解决方案一部分的 Azure 自动化帐户：

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   New-Item -ItemType Directory -Path $labFilesfolder -Force
   Set-Location -Path $labFilesfolder
   $uri = 'https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1'
   Invoke-WebRequest -Uri $Uri -OutFile '.\CreateOrUpdateAzAutoAccount.ps1'
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格运行以下脚本，以设置将分配给脚本参数的变量值：

   ```powershell
   $aadTenantId = (Get-AzContext).Tenant.Id
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   $suffix = Get-Random
   $automationAccountName = "az140-automation-51$suffix"
   $workspaceName = "az140-workspace-51$suffix"
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格运行以下脚本，以创建将在本实验室中使用的资源组：

   ```powershell
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格运行以下脚本，以创建将在本实验室中使用的 Azure Log Analytics 工作区：

   ```powershell
   New-AzOperationalInsightsWorkspace -Location $location -Name $workspaceName -ResourceGroupName $resourceGroupName
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员: Windows PowerShell ISE”的顶部菜单选择“文件”，打开“C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzAutoAccount.ps1”脚本，在第 97、98 和 99 行中添加单行注释字符，使其如下所示     ：

   ```powershell
   #    'Az.Compute'
   #    'Az.Resources'
   #    'Az.Automation'
   ```

1. 在“C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzAutoAccount.ps1”脚本中，将 82 和 86 行之间的代码包括在多行注释中，使其如下所示，并将更改保存到文件中  ：

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
    throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```
   
1. 在与 az140-dc-vm11 的远程桌面会话中，在“管理员: Windows PowerShell ISE”脚本窗格中打开一个新选项卡，粘贴下面的脚本并运行，以创建作为自动缩放解决方案一部分的 Azure 自动化帐户：

   ```powershell
   $Params = @{
     "AADTenantId" = $aadTenantId
     "SubscriptionId" = $subscriptionId 
     "UseARMAPI" = $true
     "ResourceGroupName" = $resourceGroupName
     "AutomationAccountName" = $automationAccountName
     "Location" = $location
     "WorkspaceName" = $workspaceName
   }

   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   .\CreateOrUpdateAzAutoAccount.ps1 @Params
   ```

   >**注意**：等待脚本完成。 这可能需要大约 10 分钟。

1. 在与 az140-dc-vm11 的远程桌面会话中，在“管理员: Windows PowerShell ISE”脚本窗格中，查看脚本的输出。 

   >**注意**：输出包括 Webhook URI、Log Analytics 工作区 ID 以及在预配作为自动缩放解决方案一部分的 Azure 逻辑应用时需提供的相应主键值。 
   
   >**注意**：记录 Webhook URI 的值。 本实验室中稍后会用到它。

1. 要验证 Azure 自动化帐户的配置，请在与 az140-dc-vm11 的远程桌面会话中，启动 Microsoft Edge，并导航到 [Azure 门户](https://portal.azure.com)。 如果出现提示，请使用在本实验室使用的订阅中具有所有者角色的用户帐户的 Azure AD 凭据登录。
1. 在与 az140-dc-vm11 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中，搜索并选择“自动化帐户”，然后在“自动化帐户”边栏选项卡上，选择代表新预配的 Azure 自动化帐户的条目（名称以 az140-automation-51 前缀开头）   。
1. 在“自动化帐户”边栏选项卡左侧的垂直菜单中，在“流程自动化”部分选择“Runbook”，然后在 Runbook 列表中验证是否存在“WVDAutoScaleRunbookARMBased”Runbook  。
1. 在“自动化帐户”边栏选项卡左侧垂直菜单的“帐户设置”部分中，选择“运行方式帐户”，然后在右侧帐户列表中，单击“+ Azure 运行方式帐户”旁边的“+ 创建”   。
1. 在“添加 Azure 运行方式帐户”边栏选项卡上，单击“创建”并验证新帐户是否已成功创建 。 
<!--
1. On the Automation Account blade, in the vertical menu on the left side, in the **Account Settings** section, select **Identity**.
1. On the **System assigned** tab of the Identity blade of the automation account, set the **Status** to **On**, select **Save**, and, when prompted to  confirm, select **Yes**.
1. On the **System assigned** tab of the Identity blade of the automation account, select **Azure role assignments**.
1. On the **Azure role assignments** blade, select **+ Add role assignment (Preview)**.
1. On the **Add role assignment (Preview)** blade, specify the following information and select **Save**.

   |Setting|Value|
   |---|---|
   |Scope|**Subscription**|
   |Subscription|the name of the Azure subscription where you provisioned the host pool resources|
   |Role|**Contributor**|
-->   

#### <a name="task-3-create-an-azure-logic-app"></a>任务 3：创建 Azure 逻辑应用

1. 在与 az140-dc-vm11 的远程桌面会话中，切换到“管理员: Windows PowerShell ISE”窗口，然后从“管理员:Windows PowerShell ISE”脚本窗格运行以下脚本，以下载 PowerShell 脚本，你将使用该脚本创建作为自动缩放解决方案一部分的 Azure 逻辑应用：

   ```powershell
   $labFilesfolder = 'C:\Allfiles\Labs\05'
   Set-Location -Path $labFilesfolder
   $uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
   Invoke-WebRequest -Uri $uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员: Windows PowerShell ISE”的顶部菜单选择“文件”，打开“C:\\Allfiles\\Labs\\05\\CreateOrUpdateAzLogicApp.ps1”脚本，将第 134 和 138 行之间的代码包含在多行注释中，如下所示，然后保存更改     ：

   ```powershell
   <#
   # Get the Role Assignment of the authenticated user
   $RoleAssignments = Get-AzRoleAssignment -SignInName $AzContext.Account -ExpandPrincipalGroups
   if (!($RoleAssignments | Where-Object { $_.RoleDefinitionName -in @('Owner', 'Contributor') })) {
    throw 'Authenticated user should have the Owner/Contributor permissions to the subscription'
   }
   #>
   ```

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格运行以下命令以设置将分配给脚本参数的变量值（将 `<webhook_URI>` 占位符替换为你之前在本实验室中记录的 Webhook URI 的值）：

   ```powershell
   $AADTenantId = (Get-AzContext).Tenant.Id
   $AzSubscription = (Get-AzContext).Subscription.Id
   $ResourceGroup = Get-AzResourceGroup -Name 'az140-51-RG'
   $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" -Name 'az140-21-hp1'
   $LogAnalyticsWorkspace = (Get-AzOperationalInsightsWorkspace -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $LogAnalyticsWorkspaceId = $LogAnalyticsWorkspace.CustomerId
   $LogAnalyticsWorkspaceKeys = (Get-AzOperationalInsightsWorkspaceSharedKey -ResourceGroupName $ResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspace.Name)
   $LogAnalyticsPrimaryKey = $LogAnalyticsWorkspaceKeys.PrimarySharedKey
   $RecurrenceInterval = 2
   $BeginPeakTime = '1:00'
   $EndPeakTime = '1:01'
   $TimeDifference = '0:00'
   $SessionThresholdPerCPU = 1
   $MinimumNumberOfRDSH = 1
   $MaintenanceTagName = 'CustomMaintenance'
   $LimitSecondsToForceLogOffUser = 5
   $LogOffMessageTitle = 'Autoscaling'
   $LogOffMessageBody = 'Forcing logoff due to autoscaling'

   $AutoAccount = (Get-AzAutomationAccount -ResourceGroupName $ResourceGroup.ResourceGroupName)[0]
   $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

   $WebhookURIAutoVar = '<webhook_URI>'
   ```

   >**注意**：参数值旨在加速自动缩放行为。 在生产环境中，你应该对它们进行调整以符合自己的特定要求。

1. 在与 az140-dc-vm11 的远程桌面会话中，从“管理员:  Windows PowerShell ISE”脚本窗格运行以下脚本，以创建作为自动缩放解决方案一部分的 Azure 逻辑应用：

   ```powershell
   $Params = @{
     "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
     "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
     "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
     "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
     "UseARMAPI"                     = $true
     "HostPoolName"                  = $WVDHostPool.Name
     "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
     "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
     "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
     "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
     "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
     "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
     "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
     "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
     "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
     "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
     "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
     "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
     "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shut down."
     "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
     "WebhookURI"                    = $WebhookURIAutoVar
   }

   .\CreateOrUpdateAzLogicApp.ps1 @Params
   ```

   >**注意**：等待脚本完成。 这可能需要大约 2 分钟。

1. 要验证 Azure 逻辑应用的配置，请在与 az140-dc-vm11 的远程桌面会话中，切换到显示 Azure 门户的“Microsoft Edge”窗口，搜索并选择“逻辑应用”，然后在“逻辑应用”边栏选项卡上，选择表示新预配的名为“az140-21-hp1_Autoscale_Scheduler”的 Azure 逻辑应用条目   。
1. 在“az140-21-hp1_Autoscale_Scheduler”边栏选项卡左侧垂直菜单中的“开发工具”部分，选择“逻辑应用设计器”  。 
1. 在设计器窗格中，单击标记为“定期”的矩形，并注意你可以使用它来控制评估自动缩放需求的频率。 

### <a name="exercise-2-verify-and-review-autoscaling-of-azure-virtual-desktop-session-hosts"></a>练习 2：验证和查看 Azure 虚拟桌面会话主机的自动缩放功能

此练习的主要任务如下：

1. 验证 Azure 虚拟桌面会话主机的自动缩放功能
1. 使用 Azure Log Analytics 跟踪 Azure 虚拟桌面事件

#### <a name="task-1-verify-autoscaling-of-azure-virtual-desktop-session-hosts"></a>任务 1：验证 Azure 虚拟桌面会话主机的自动缩放功能

1. 要验证 Azure 虚拟桌面会话主机的自动缩放功能，请在与 az140-dc-vm11 的远程桌面会话中，在显示 Azure 门户的“Microsoft Edge”窗口中搜索并选择“虚拟机”，然后在“虚拟机”边栏选项卡上，查看“az140-21-RG”资源组中三个 Azure VM 的状态   。
1. 验证这三个 Azure VM 中的两个 VM 是否正在解除分配或已停止（已解除分配）。

   >**注意**：验证自动缩放正常运行后，应禁用 Azure 逻辑应用以最大限度地降低相应的费用。

1. 要禁用 Azure 逻辑应用，请在与 az140-dc-vm11 的远程桌面会话中，在显示 Azure 门户的“Microsoft Edge”窗口中，搜索并选择“逻辑应用”，然后在“逻辑应用”边栏选项卡上，选择代表新预配的名为“az140-21-hp1_Autoscale_Scheduler”的 Azure 逻辑应用条目   。
1. 在“az140-21-hp1_Autoscale_Scheduler”边栏选项卡上的工具栏中，单击“禁用”。 
1. 在“az140-21-hp1_Autoscale_Scheduler”边栏选项卡上的“基本信息”部分中，查看过去 24 小时内成功运行的次数等信息，以及提供重复频率的“摘要”部分  。 
1. 在与 az140-dc-vm11 的远程桌面会话中，在显示 Azure 门户的 Microsoft Edge 窗口中，搜索并选择“自动化帐户”，然后在“自动化帐户”边栏选项卡上，选择代表新预配的 Azure 自动化帐户的条目（名称以 az140-automation-51 前缀开头）   。
1. 在“自动化帐户”边栏选项卡左侧的垂直菜单中，在“流程自动化”部分选择“作业”，然后查看与“WVDAutoScaleRunbookARMBased”runbook 的各个调用对应的作业列表   。
1. 选择最近的作业，然后在其边栏选项卡上单击“所有日志”选项卡标题。 随即将显示作业执行步骤的详细列表。

#### <a name="task-2-use-azure-log-analytics-to-track-azure-virtual-desktop-events"></a>任务 2：使用 Azure Log Analytics 跟踪 Azure 虚拟桌面事件

>**注意**：要分析自动缩放和任何其他 Azure 虚拟桌面事件，可以使用 Log Analytics。

1. 在与 az140-dc-vm11 的远程桌面会话中，在显示 Azure 门户的“Microsoft Edge”窗口中，搜索并选择“Log Analytics 工作区”，然后在“Log Analytics 工作区”边栏选项卡上，选择代表本实验室使用的 Azure Log Analytics 工作区条目（名称以 az140-workspace-51 前缀开头）   。
1. 在“Log Analytics 工作区”边栏选项卡左侧垂直菜单的“常规”部分中，单击“日志”，如果需要，请关闭“欢迎使用 Log Analytics”窗口，然后前往“查询”窗格   。
1. 在“查询”窗格左侧的“所有查询”垂直菜单中，选择“Azure 虚拟桌面”并查看预定义的查询  。
1. 关闭“查询”窗格。 这将自动显示“新建查询 1”选项卡。
1. 在“查询”窗口中，粘贴以下查询，单击“运行”以显示本实验室中使用的主机池的所有事件：

   ```kql
   WVDTenantScale_CL
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

   >**注意**：如果在使用剪切和粘贴结构时第二行中有一个额外的竖线字符 (|)，请将其删除以避免失败。 这适用于每个查询。
   >**注意**：如果未显示任何结果，请等待几分钟，然后重试。

1. 在“查询”窗口中，粘贴以下查询，单击“运行”以显示目标主机池中当前运行的会话主机和活动用户会话的总数：

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Number of running session hosts:"
     or logmessage_s contains "Number of user sessions:"
     or logmessage_s contains "Number of user sessions per Core:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. 在“查询”窗口中，粘贴以下查询，单击“运行”以显示主机池中所有会话主机 VM 的状态：

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "Session host:"
   | where hostpoolName_s == "az140-21-hp1"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

1. 在“查询”窗口中，粘贴以下查询，单击“运行”以显示任何与缩放相关的错误和警告：

   ```kql
   WVDTenantScale_CL
   | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
   | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
   ```

>**注意**：忽略有关 `TenantId` 的错误消息

### <a name="exercise-3-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>练习 3：停止并解除分配在实验室中预配的 Azure VM

此练习的主要任务如下：

1. 停止并解除分配在实验室中预配的 Azure VM

>**注意**：在此练习中，你将解除分配此实验室中预配的 Azure VM，以最大程度减少相应的计算费用

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>任务 1：解除分配在实验室中预配的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开 Cloud Shell 窗格内的“PowerShell”shell 会话 。
1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以列出本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注意**：该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能停止和解除分配 Azure VM。
