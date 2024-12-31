---
lab:
  title: 实验室：实现会话主机的自动缩放
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# 实验室 - 实现和监控会话主机的自动缩放
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- Microsoft Entra 用户帐户，该帐户在本实验室将会使用的 Azure 订阅中具有所有者或参与者角色，在与该 Azure 订阅关联的 Entra 租户中具有足够加入设备的权限。
- 已完成实验室*使用 Azure 门户部署主机池和会话主机 (Entra ID)*
- 已完成实验室“*使用 Azure 门户 (Entra ID) 管理主机池和会话主机*”
- 实验室*连接到会话主机 (Entra ID)* 已完成

## 预计用时

45 分钟

## 实验室方案

你有一个 Azure 虚拟桌面环境，其使用情况会定期更改。 你希望通过利用自动缩放计划功能来最小化成本。

## 目标
  
完成本实验室后，你将能够：

- 实现和评估 Azure 虚拟桌面自动缩放

## 实验室文件

- 无

## 说明

### 练习 1：实现 Azure 虚拟桌面自动缩放计划
  
此练习的主要任务如下：

1. 向 Azure 虚拟桌面服务主体分配所需的 RBAC 角色
1. 停止并解除分配所有会话主机
1. 调整主机池设置
1. 创建缩放计划
1. 评估自动缩放功能
1. 禁用主机池自动缩放

#### 任务 1：将所需的 RBAC 角色分配给 Azure 虚拟桌面服务主体

> **备注**：若要使自动缩放计划正常工作，需要向 Azure 虚拟桌面服务主体授予管理会话主机 VM 电源状态的权限。 可以通过使用内置**桌面虚拟化电源关闭参与者** RBAC 角色来授予这些权限。 请务必记住，必须在订阅范围内执行角色分配。 如果在低于你的订阅的任何级别（例如资源组、主机池或 VM）分配此角色，自动缩放将无法正常工作。 

> **备注**：此角色不同于实验室*使用 Azure 门户 (Entra ID) 管理主机池和会话主机*中使用的角色（**桌面虚拟化启用参与者**），后者是支持*连接时启动 VM* 功能所必须的。

1. 如果需要，在实验室计算机上，启动 Web 浏览器，导航到 Azure 门户，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。

    > **备注**：使用实验室会话窗口右侧“资源”选项卡上列出的 `User1-` 帐户的凭据。

1. 在实验室计算机上显示 Azure 门户的 Web 浏览器中，开启 Azure Cloud Shell 中的 PowerShell 会话。

    > **备注**：如果出现提示，请在“**入门**”窗格中的“**订阅**”下拉列表中，选择正在本实验室中使用的 Azure 订阅的名称，然后选择“**应用**”。

1. 在“Azure Cloud Shell”窗格中的 PowerShell 会话中，运行以下命令以检索正在此实验室中使用的 Azure 订阅的 ID 属性的值，并将其存储在变量 `$subId` 中：

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. 运行以下命令以创建一个 $parameters 变量，该变量用于存储包含 RBAC 角色定义名称值的哈希表，表示 **Azure 虚拟桌面**服务主体的 Microsoft Entra 应用程序以及订阅范围：

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Off Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. 运行以下命令以创建 RBAC 角色分配：

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. 关闭 Cloud Shell 窗格。

#### 任务 2：停止并解除分配所有会话主机

> **备注**：若要评估自动缩放功能，将停止并解除分配 Azure 虚拟桌面环境中的所有会话主机。 

1. 在实验室计算机中，在显示 Azure 门户的 Web 浏览器中搜索并选择“**Azure 虚拟桌面**”，然后在“**Azure 虚拟桌面**”页的垂直菜单栏的“**管理**”部分中，选择“**主机池**”。
1. 在“**Azure 虚拟桌面 \| 主机池**”页的主机池列表中，选择“**az140-21-hp1**”。
1. 在“**az140-21-hp1**”页的垂直菜单栏的“**管理**”部分，选择“**会话主机**”。
1. 在“**az140-21-hp1 \| 会话主机**”页上，选中每个会话主机名称旁边的复选框，然后选择“**停止**”。
1. 当系统提示确认时，在“**停止会话主机**”弹出窗口中，选择“**停止**”。

    > **备注**：可能需要选择工具栏中的省略号 (`...`) 图标以显示“**停止**”按钮。

    > **备注**：不要等到会话主机停止并解除分配，而是继续执行下一个任务。 停止和解除分配会话主机可能需要大约 2 分钟。

#### 任务 3：调整主机池设置

> **备注**：对共用主机池使用自动缩放时，必须为该主机池配置 MaxSessionLimit 参数。 在此实验室中，为了便于说明自动缩放功能，你将人为地将其设置为较低值。

1. 在实验室计算机中，在显示 Azure 门户的 Web 浏览器中，在 **az140-21-hp1** 页上的 **“设置”** 部分，选择“**属性**”。
1. 在“**az140-21-hp1\|属性**”页的“**最大会话限制**”文本框中，输入 **1**。
1. 在“**az140-21-hp1\|属性**”页上，选择“**保存**”。

#### 任务 4：创建缩放计划

1. 在实验室计算机上，在显示 Azure 门户的 Web 浏览器中搜索并选择“**Azure 虚拟桌面**”，在“**Azure 虚拟桌面**”页的垂直导航菜单的“**管理**”部分中，选择“**缩放计划**”。
1. 在“**Azure 虚拟桌面\|缩放计划**”页上，选择“**+ 创建**”。
1. 在“**创建缩放计划**”页的“**基本信息**”选项卡中，指定以下设置并选择“**下一步: 计划**”：

    |设置|值|
    |---|---|
    |订阅|你在此实验室中使用的 Azure 订阅的名称|
    |资源组|新资源组名称 **az140-412e-RG**|
    |缩放计划名称|**az140-scalingplan412e**|
    |区域|部署 Azure 虚拟桌面环境的 Azure 区域的名称|
    |友好名称|**az140-scalingplan412e**|
    |时区|在其中部署 Azure 虚拟桌面环境的 Azure 区域的本地时区|
    |主机池类型|**池**|
    |缩放方法|**电源管理自动缩放**|

    > **备注**：不设置“**排除标记**”属性。 一般情况下，可以使用此功能从自动缩放中排除具有任意设置标记的 Azure VM。

1. 在“**计划**”选项卡中，选择“**+添加计划**”。

    > **备注**：通过计划，可以定义工作日的上升时段、高峰时段、下降时段和非高峰时段，并指定自动缩放触发器。 缩放计划必须包含一周中至少一天的关联计划。 

1. 在“**添加计划**”窗格的“**常规**”选项卡上，调整默认配置以匹配以下设置，然后选择“**下一步**”：

    |设置|“值”|
    |---|---|
    |时区|Azure 虚拟桌面环境的本地时区（基于之前在此任务中选择的区域）|
    |计划名称|**week_schedule**|
    |重复|已选择 7****（选择一周的每一天）|

    > **备注**：该计划有效地涵盖一周中的每一天，这将促进评估自动缩放的结果。

1. 在“**添加计划**”窗格的“**提升**”选项卡上，调整默认配置以匹配以下设置，然后选择“**下一步**”：

    |设置|“值”|
    |---|---|
    |开始时间（12 小时制）|当前时间减去 1 小时|
    |负载均衡算法|**广度优先**|
    |主机的最小百分比 (%)|**30**|
    |容量阈值 (%)|**60**|

    > **备注**：对于共用主机池，自动缩放将忽略主机池设置中的现有负载均衡算法，并转而根据日程安排配置应用负载均衡。

    > **备注**：“**主机的最小百分比**”设置用于指定在上升和高峰时段启动的会话主机虚拟机的最小百分比。 例如，如果将“**主机的最小百分比**”指定为 30%，并且主机池中的会话主机总数为 3，则自动缩放将确保至少 1 个会话主机可用于接受用户连接。

    > **备注**：自动缩放向上舍入到最接近的整数。

    > **备注**：“**容量阈值**”设置是已用主机池容量的百分比，评估在上升和高峰时段是否打开/关闭虚拟机时将考虑此百分比。 例如，如果将容量阈值指定为 60%，并且主机池容量为 1 个会话（1 个主机运行），则在主机池的负载超过 60%（此案例中为 100%）后，自动缩放就会打开其他会话主机。

1. 在在“**添加计划**”窗格的“**高峰时段**”选项卡上，调整默认配置以匹配以下设置，然后选择“**下一步**”：

    |设置|“值”|
    |---|---|
    |开始时间（12 小时制）|当前时间减去 1 小时|
    |负载均衡算法|**深度优先**|
    |容量阈值 (%)|**60**|

    > **备注**：“**容量阈值 (%)**”设置在“**上升**”和“**高峰时段**”设置之间共享。

1. 在“**添加计划**”窗格的“**下降**”选项卡上，调整默认配置以匹配以下设置，然后选择“**下一步**”：

    |设置|“值”|
    |---|---|
    |开始时间（12 小时制）|当前时间减去 2 小时|
    |负载均衡算法|**深度优先**|
    |活动主机的最小百分比(%)|**10**|
    |容量阈值 (%)|**80**|
    |强制注销用户|**否**|
    |在以下情况下停止 VM|**VM 没有活动会话或断开连接的会话**|

    > **备注**：“**活动主机的最小百分比 (%)**”设置指定在下降和非高峰时段内要达到的会话主机虚拟机的最小百分比。 例如，如果“**活动主机的最小百分比 (%)**”设置为 10%，并且主机池中的会话主机总数为 3，则自动缩放将确保至少有 1 个会话主机可用于建立用户连接。

    > **备注**：“**容量阈值 (%)**”设置指定已用主机池容量的百分比，评估在下降和非高峰时段是否关闭虚拟机时会考虑此百分比。 例如，在 1 个用户连接和 3 台主机运行的情况下，如果容量阈值指定为 80%，自动缩放将关闭 1 台主机（导致主机池容量使用 50%）。

    > **备注**：一般情况下，自动缩放会根据以下规则停止和解除分配会话主机：

    - 已用主机池容量低于容量阈值。
    - 关闭会话主机不会导致超出容量阈值。
    - 自动缩放只会关闭没有用户会话的会话主机，除非缩放计划已进入缩减阶段，并且你已启用强制用户注销设置。 
    - 共用自动缩放不会在增加阶段关闭会话主机，以确保用户体验不受影响。

1. 在“**添加计划**”窗格的“**非高峰时段**”选项卡上，调整默认配置以匹配以下设置，然后选择“**添加**”：

    |设置|“值”|
    |---|---|
    |开始时间（12 小时制）|当前时间加上 3 小时|
    |负载均衡算法|**深度优先**|
    |容量阈值 (%)|**80**|

    > **备注**：**容量阈值**设置在“**下降时段**”和“**非高峰时段**”设置之间共享。

1. 返回“**创建缩放计划**”页的“**计划**”选项卡，选择“**下一步: 主机池分配**”。
1. 在“**主机池分配**”选项卡上的“**选择主机池**”下拉列表中，选择“**az140-21-hp1**”，确保选中“**启用自动缩放**”复选框，然后选择“**查看 + 创建**”。
1. 在“查看 + 创建”页上，选择“创建”。

    > **备注**：等待自动缩放配置完成。 这通常只需要几秒钟时间。

#### 任务 5：评估自动缩放功能

> **备注**：首先评估“**上升**”设置。

1. 在实验室计算机中，在显示 Azure 门户的 Web 浏览器中搜索并选择“**Azure 虚拟桌面**”，然后在“**Azure 虚拟桌面**”页的垂直菜单栏的“**管理**”部分中，选择“**主机池**”。
1. 在“**Azure 虚拟桌面 \| 主机池**”页的主机池列表中，选择“**az140-21-hp1**”。
1. 在“**az140-21-hp1**”页的垂直菜单栏的“**管理**”部分，选择“**会话主机**”。
1. 在“**az140-21-hp1 \| 会话主机**”页上，查看会话主机的“**电源状态**”设置的值，并验证其中一个主机是否列为“**正在运行**”。

    > **备注**：可能需要等待几分钟，然后第一个会话主机才达到“**正在运行**”状态。

    > **备注**：这符合预期，因为根据新创建的缩放计划的“**上升**”设置，至少一个会话主机应始终处于联机状态。 此时，主机池容量为 1（因为只有 1 台主机正在运行），但使用的主机池容量为 0%，因为没有用户连接。

    > **备注**：接下来，将通过启动单个用户会话来评估“**上升时段**”和“**高峰时段**”容量阈值设置。 即使在“**高峰时段**”窗口之外，我们也能评估这一点，因为两个阶段共享相同的容量阈值。

1. 在实验室计算机中，启动 Microsoft 远程桌面客户端。
1. 在实验室计算机上，在“**远程桌面**”客户端窗口中选择“**订阅**”，并在出现提示时，使用可在实验室界面窗口右窗格中的“**资源**”选项卡上找到的 `User2`Entra ID 用户帐户的凭据登录。
1. 确保“**远程桌面**”页显示四个图标，包括 Microsoft Word、Microsoft Excel、Microsoft PowerPoint 和命令提示符。 
1. 双击命令提示符图标。 
1. 当系统提示登录时，在“**Windows 安全**”对话框中，输入用于连接到目标 Azure 虚拟桌面环境的 Microsoft Entra 用户帐户的密码。
1. 验证不久后是否会显示**命令提示符**窗口。 

    > **备注**：此时，已用主机池容量为 100%，大于容量阈值 (60%)。 这将导致自动缩放打开另一台主机，将使已用主机池容量达到 50%。 由于低于容量阈值，第三台主机将保持停止/解除分配。 接下来将对此进行验证。

1. 在实验室计算机上，切换到显示 Azure 门户的 Web 浏览器。 
1. 在“**az140-21-hp1 \| 会话主机**”页上，选择“**刷新**”，查看会话主机的“**电源状态**”设置的值，并验证其中两个主机现在是否列为“**正在运行**”。

    > **备注**：接下来，将通过调整其时间窗口来评估“**下降**”容量阈值设置。 

1. 在 **az140-21-hp1 \| 会话主机**页的垂直导航菜单中的“**设置**”部分，选择“**缩放计划**”，然后在“**缩放计划**”页上，选择 **az140-scalingplan412e**。
1. 在 **az140-scalingplan412e** 页的垂直导航菜单中的“**设置**”部分，选择“**计划**”，然后选择 **week_schedule**。
1. 在“**week_schedule**”窗格中，导航到“**缩减**”选项卡，将“**开始时间(12 小时制)**”设置的值调整为“**高峰时段**”阶段的“**开始时间(12 小时制)”** 和当前时间之间的任意时间。

    > **备注**：可能需要调整“**高峰时段**”阶段的“**开始时间(12 小时制)**”的值。

1. 在“**week_schedule**”窗格中，导航到“**非高峰时段**”选项卡，然后选择“**保存**”。
1. 切换到表示主机池唯一 RDP 会话的**命令提示符**窗口，并在命令提示符处输入以下命令，然后按 **Enter** 键：

    ```cmd
    logoff
    ```

1. 在实验室计算机中，在显示 Azure 门户的 Web 浏览器中搜索并选择“**Azure 虚拟桌面**”，然后在“**Azure 虚拟桌面**”页的垂直菜单栏的“**管理**”部分中，选择“**主机池**”。
1. 在“**Azure 虚拟桌面 \| 主机池**”页的主机池列表中，选择“**az140-21-hp1**”。
1. 在“**az140-21-hp1**”页的垂直菜单栏的“**管理**”部分，选择“**会话主机**”。
1. 在“**az140-21-hp1 \| 会话主机**”页上，查看会话主机“**电源状态**”设置的值，并验证现在是否只有其中一个主机列为“**正在运行**”。

    > **备注**：在会话主机关闭前可能需要 1-2 分钟。

#### 任务 6：禁用主机池自动缩放

> **备注**：为了确保自动缩放配置不会影响其他实验室，你将删除在此实验室中实现的缩放计划的主机池分配。

1. 在实验室计算机显示 Azure 门户的 Web 浏览器中，搜索并选择 **Azure 虚拟桌面**，在 **Azure 虚拟桌面**页上，在垂直导航菜单的“**管理**”部分中，选择“**缩放计划**”，然后在“**缩放计划**”页上选择 **az140-scalingplan412e**。
1. 在 **az140-scalingplan412e** 页上的“**管理**”部分中，选择“**主机池分配**”。
1. 在“**az140-scalingplan412e \| 主机池分配**”页上，选择 **az140-21-hp1**，然后选择“**取消分配**”，并在系统提示确认时，在“**取消分配主机池**”对话框中，选择“**取消分配**”。