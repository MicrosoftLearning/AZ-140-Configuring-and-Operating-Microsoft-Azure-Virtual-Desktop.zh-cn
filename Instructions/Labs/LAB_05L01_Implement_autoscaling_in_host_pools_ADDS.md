---
lab:
  title: 实验室：在主机池中实现自动扩展 (AD DS)
  module: 'Module 5: Monitor and Maintain an AVD Infrastructure'
---

# 实验室 - 在主机池中实现自动缩放 (AD DS)
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- Microsoft 帐户或 Microsoft Entra 帐户，该帐户在本实验室将会使用的 Azure 订阅中具有所有者或参与者角色，在与该 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色。
- 已完成实验室“准备部署 Azure 虚拟桌面 (AD DS)”
- 已完成实验室“使用 Azure 门户部署主机池和会话主机 (AD DS)”

## 预计用时

60 分钟

## 实验室方案

你需要在 Active Directory 域服务 (AD DS) 环境中配置 Azure 虚拟桌面会话主机的自动缩放功能。

## 目标
  
完成本实验室后，你将能够：

- 配置 Azure 虚拟桌面会话主机的自动缩放功能
- 验证 Azure 虚拟桌面会话主机的自动缩放功能

## 实验室文件

- 无

## 说明

### 练习 1：配置 Azure 虚拟桌面会话主机的自动缩放功能

此练习的主要任务如下：

1. 准备缩放 Azure 虚拟桌面会话主机
2. 创建 Azure 虚拟桌面会话主机的缩放计划

#### 任务 1：准备缩放 Azure 虚拟桌面会话主机

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后通过提供在本实验室中将要使用的订阅中具有所有者角色的用户帐户凭据进行登录。
1. 在实验室计算机上的显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”**** 窗格内的 PowerShell **** 会话。

   >**备注**：应使用 MaxSessionLimit**** 参数的非默认值对计划用于自动缩放的主机池进行配置。 可以在 Azure 门户中的主机池设置中配置此值，或者运行“Update-AzWvdHostPool”**** Azure PowerShell cmdlet 配置此值（如本例所示）。 还可以在 Azure 门户中创建池或在运行“New-AzWvdHostPool****”Azure PowerShell cmdlet 时进行显式设置。

1. 在“Cloud Shell”窗格的 PowerShell 会话中运行以下命令，将 az140-21-hp1**** 主机池的 MaxSessionLimit**** 参数值设置为 2****： 

   ```powershell
   Update-AzWvdHostPool -ResourceGroupName 'az140-21-RG' `
   -Name az140-21-hp1 `
   -MaxSessionLimit 2
   ```

   >**备注**：在此实验室中，MaxSessionLimit**** 参数的值被人为地设置得很低，以便触发自动缩放行为。

   >**备注**：在创建第一个缩放计划之前，需要将“桌面虚拟化启停参与者”RBAC 角色**** 分配给 Azure 虚拟桌面，并将 Azure 订阅用作目标范围。 

1. 在显示 Azure 门户的浏览器窗口中，关闭“Cloud Shell”窗格。
1. 在 Azure 门户中，搜索并选择“订阅”****，然后从订阅列表中选择包含 Azure 虚拟桌面资源的订阅。 
1. 在“订阅”页上，选择“访问控制(IAM)”****。
1. 在“访问控制(IAM)”**** 页上的工具栏中，选择“+ 添加”按钮****，然后从下拉列表菜单中选择“添加角色分配”****。
1. 在“添加角色分配”**** 向导的“角色”**** 选项卡上，搜索并选择“桌面虚拟化启停参与者”**** 角色，然后单击“下一步”****。
1. 在“添加角色分配”**** 向导的“成员”**** 选项卡上，选择“+ 选择成员”****，搜索并选择“Azure 虚拟桌面”**** 或“Windows 虚拟桌面”****，单击“选择”**** 并单击“下一步”****。

   >**备注**：该值取决于 Microsoft.DesktopVirtualization**** 资源提供程序首次在 Azure 租户中注册的时间。

1. 在“查看 + 分配”**** 选项卡上，选择“查看 + 分配”****。

#### 任务 2：创建 Azure 虚拟桌面会话主机的缩放计划

1. 在实验室计算机上的显示 Azure 门户的浏览器中，搜索并选择“Azure 虚拟桌面”****。 
1. 在“Azure 虚拟桌面”**** 页上，选择“缩放计划”****，然后选择“+ 创建”****。
1. 在“创建缩放计划”**** 向导的“基本信息”**** 选项卡上，指定以下信息并选择“下一步计划 >”****（其他设置保留默认值）：

   |设置|值|
   |---|---|
   |资源组|新资源组名称 az140-51-RG****|
   |名称|az140-51-scaling-plan****|
   |位置|在上一个实验室中部署会话主机的同一 Azure 区域|
   |友好名称|az140-51 缩放计划****|
   |时区|本地时区|

   >**备注**：排除标记允许为要从缩放操作中排除的会话主机指定标记名称。 例如，你可能希望标记已设置为排出模式的 VM，以便自动缩放不会在维护期间使用排除标记“excludeFromScaling”替代排出模式。 

1. 在“创建缩放计划”**** 向导的“计划”**** 选项卡上，选择“+ 添加计划”****。
1. 在“添加计划”**** 向导的“常规”**** 选项卡上，指定以下信息，然后单击“下一步”****。

   |设置|“值”|
   |---|---|
   |计划名称|az140-51-schedule****|
   |重复|已选择 7****（选择一周的每一天）|

1. 在“添加计划”**** 向导的“增加”**** 选项卡上，指定以下信息，然后单击“下一步”****。

   |设置|“值”|
   |---|---|
   |开始时间（24 小时制）|当前时间减去 9 小时|
   |负载均衡算法|广度优先****|
   |主机的最小百分比 (%)|**20**|
   |容量阈值 (%)|**60**|

   >**备注**：在此处选择的负载均衡首选项将覆盖你为原始主机池设置选择的负载均衡首选项。

   >**备注**：主机的最小百分比指定你希望始终保持开启的会话主机的百分比。 如果输入的百分比不是整数，则会向上舍入为最接近的整数。 

   >**备注**：容量阈值代表将触发缩放操作的可用主机池容量百分比。 例如，如果主机池中启用了两个会话主机（会话上限为 20），则主机池可用容量为 40。 如果将容量阈值设置为 75% 并且会话主机具有超过 30 个用户会话，则自动缩放将启用第三个会话主机。 这会将可用的主机池容量从 40 更改为 60。

1. 在“添加计划”**** 向导的“高峰时段”**** 选项卡上，指定以下信息，然后单击“下一步”****。

   |设置|“值”|
   |---|---|
   |开始时间（24 小时制）|当前时间减去 8 小时|
   |负载均衡算法|**深度优先**|

   >**备注**：开始时间指定增加阶段的结束时间。

   >**备注**：此阶段中的容量阈值由增加容量阈值确定。

1. 在“添加计划”**** 向导的“减少”**** 选项卡上，指定以下信息，然后单击“下一步”****。

   |设置|“值”|
   |---|---|
   |开始时间（24 小时制）|当前时间减去 2 小时|
   |负载均衡算法|**深度优先**|
   |主机的最小百分比 (%)|**10**|
   |容量阈值 (%)|**90**|
   |强制注销用户|**是**|
   |注销用户并关闭 VM 之前的延迟时间（分钟）|**30**|

   >**备注**：如果启用了“强制注销用户”**** ，则自动缩放会将具有最低用户会话数量的会话主机置于排出模式，向所有活动用户会话发送关于即将发生的关闭的通知，并在指定的延迟时间后强制注销这些用户。 在自动缩放注销所有用户会话后，它会解除分配 VM。 

   >**备注**：如果在减少期间未启用强制注销，则将解除分配无活动会话或已断开连接的会话的会话主机。

1. 在“添加计划”**** 向导的“非高峰时段”**** 选项卡上，指定以下信息，然后单击“添加”****。

   |设置|“值”|
   |---|---|
   |开始时间（24 小时制）|当前时间减去 1 小时|
   |负载均衡算法|**深度优先**|

   >**备注**：此阶段中的容量阈值由减少容量阈值确定。

1. 返回“创建缩放计划”**** 向导的“计划”**** 选项卡，选择“下一步: 主机池分配 >”****：
1. 在“主机池分配”**** 页上的“选择主机池”**** 下拉列表中，选择“az140-21-hp1”****，确保选中“启用自动缩放”**** 复选框，选择“查看 + 创建”****，然后选择“创建”****。


### 练习 2：验证 Azure 虚拟桌面会话主机的自动缩放

此练习的主要任务如下：

1. 设置诊断以跟踪 Azure 虚拟桌面自动缩放
1. 验证 Azure 虚拟桌面会话主机的自动缩放功能

#### 任务 1：设置诊断以跟踪 Azure 虚拟桌面自动缩放

1. 在实验室计算机上的显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”**** 窗格内的 PowerShell **** 会话。

   >**备注**：将使用 Azure 存储帐户来存储自动缩放事件。 可以直接从 Azure 门户创建，也可以使用 Azure PowerShell，如此任务所示。

1. 在“Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以创建 Azure 存储帐户：

   ```powershell
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   $suffix = Get-Random
   $storageAccountName = "az140st51$suffix"
   New-AzStorageAccount -Location $location -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName Standard_LRS
   ```

   >**备注**：等待至存储帐户预配完成。

1. 在显示 Azure 门户的浏览器窗口中，关闭“Cloud Shell”窗格。
1. 在实验室计算机上显示 Azure 门户的浏览器中，导航到在上一练习中创建的缩放计划页。
1. 在“az140-51-scaling-plan”**** 页上，选择“诊断设置”****，然后选择“+ 添加诊断设置”****。
1. 在“诊断设置”**** 页上的“诊断设置名称”**** 文本框中，输入“az140-51-scaling-plan-diagnostics”****，然后在“类别组”**** 部分选择“allLogs”****。 
1. 在同一页上，在“目标详细信息”**** 部分，选择“存档到存储帐户”****，然后在“存储帐户”**** 下拉列表中，选择以 az140st51**** 前缀开头的存储帐户名称。
1. 选择“保存”。

#### 任务 2：验证 Azure 虚拟桌面会话主机的自动缩放

1. 在实验室计算机上的显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”**** 窗格内的 PowerShell **** 会话。
1. 从“Cloud Shell”窗格中的 PowerShell 会话运行以下命令，以启动你将在本实验室中使用的 Azure 虚拟桌面会话主机 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**备注**：等待至会话主机 Azure VM 运行。

1. 在实验室计算机上显示 Azure 门户的 Web 浏览器窗口中，导航到“az140-21-hp1”**** 主机池页。
1. 在“az140-21-hp1”**** 页中，选择“会话主机”****。
1. 等待至少一个会话主机列出并显示“关闭”**** 状态。

   >**备注**：可能需要刷新页面以更新会话主机的状态。

   >**备注**：如果所有会话主机保持可用，请导航回“az140-51-scaling-plan”**** 页，并减少“最低主机百分比(%)”****“减少”**** 设置的值。

   >**备注**：一个或多个会话主机的状态发生更改后，自动缩放日志应会显示在 Azure 存储帐户中。 

1. 在 Azure 门户中，搜索并选择“存储帐户”****，然后在“存储帐户”**** 页上，选择表示之前在本练习中创建的存储帐户（名称以 az140st51**** 前缀开头）的条目。
1. 在“存储帐户”页上，选择“容器”****。
1. 在容器列表中，选择“insights-logs-autoscale”****。
1. 在“insights-logs-autoscale”**** 页上，在文件夹层次结构中导航，找到表示存储在容器中的 JSON 格式 blob 条目。
1. 选择 Blob 条目，选择页面最右侧的省略号图标，然后在下拉菜单中选择“下载”****。
1. 在实验室计算机上，在所选的文本编辑器中打开下载的 Blob 并检查其内容。 你应该能够找到对自动缩放事件的引用。 

   >**备注**：下面是一个示例 Blob 内容，其中包括对自动缩放事件的引用：

   ```json
   host_Ring    "R0"
   Level    4
   ActivityId   "00000000-0000-0000-0000-000000000000"
   time "2023-03-26T19:35:46.0074598Z"
   resourceId   "/SUBSCRIPTIONS/AAAAAAAE-0000-1111-2222-333333333333/RESOURCEGROUPS/AZ140-51-RG/PROVIDERS/MICROSOFT.DESKTOPVIRTUALIZATION/SCALINGPLANS/AZ140-51-SCALING-PLAN"
   operationName    "ScalingEvaluationSummary"
   category "Autoscale"
   resultType   "Succeeded"
   level    "Informational"
   correlationId    "ddd3333d-90c2-478c-ac98-b026d29e24d5"
   properties   
   Message  "Active session hosts are at 0.00% capacity (0 sessions across 3 active session hosts). This is below the minimum capacity threshold of 90%. 2 session hosts can be drained and deallocated."
   HostPoolArmPath  "/subscriptions/aaaaaaaa-0000-1111-2222-333333333333/resourcegroups/az140-21-rg/providers/microsoft.desktopvirtualization/hostpools/az140-21-hp1"
   ScalingEvaluationStartTime   "2023-03-26T19:35:43.3593413Z"
   TotalSessionHostCount    "3"
   UnhealthySessionHostCount    "0"
   ExcludedSessionHostCount "0"
   ActiveSessionHostCount   "3"
   SessionCount "0"
   CurrentSessionOccupancyPercent   "0"
   CurrentActiveSessionHostsPercent "100"
   Config.ScheduleName  "az140-51-schedule"
   Config.SchedulePhase "OffPeak"
   Config.MaxSessionLimitPerSessionHost "2"
   Config.CapacityThresholdPercent  "90"
   Config.MinActiveSessionHostsPercent  "5"
   DesiredToScaleSessionHostCount   "-2"
   EligibleToScaleSessionHostCount  "1"
   ScalingReasonType    "DeallocateVMs_BelowMinSessionThreshold"
   BeganForceLogoffOnSessionHostCount   "0"
   BeganDeallocateVmCount   "1"
   BeganStartVmCount    "0"
   TurnedOffDrainModeCount  "0"
   TurnedOnDrainModeCount   "1"
   ```


### 练习 3：停止并解除分配在实验室中预配的 Azure VM

此练习的主要任务如下：

1. 停止并解除分配在实验室中预配的 Azure VM

>**备注**：在此练习中，你将解除分配此实验室中使用的 Azure VM，以最大程度减少相应的计算费用。

#### 任务 1：解除分配在实验室中预配的 Azure VM

1. 切换到实验室计算机，然后在显示 Azure 门户的 Web 浏览器窗口中，打开“Cloud Shell”**** 窗格内的 PowerShell**** 会话 。
1. 从“Cloud Shell”窗格的 PowerShell 会话中运行以下命令，以列出本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. 在“Cloud Shell”窗格的 PowerShell 会话中，运行以下命令以停止和解除分配本实验室中创建的所有 Azure VM：

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注意**：该命令异步执行（由 -NoWait 参数确定），因此尽管此后可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上也要花几分钟才能停止和解除分配 Azure VM。
