---
lab:
  title: 实验室：连接到会话主机 (Entra ID)
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# 实验室 - 连接到会话主机 (Entra ID)
# 学生实验室手册

## 实验室依赖项

- 本实验室将使用的 Azure 订阅。
- Microsoft Entra 用户帐户，该帐户在本实验室将会使用的 Azure 订阅中具有所有者或参与者角色，在与该 Azure 订阅关联的 Entra 租户中具有足够加入设备的权限。
- 已完成实验室*使用 Azure 门户部署主机池和会话主机 (Entra ID)*
- 已完成实验室“*使用 Azure 门户 (Entra ID) 管理主机池和会话主机*”

## 估计时间

20 分钟

## 实验室方案

现有一个 Azure 虚拟桌面环境，包含已加入 Entra 的会话主机。 需要通过从未加入或未注册 Microsoft Entra 的 Windows 11 客户端连接到它们来验证其功能。

## 目标
  
完成本实验室后，你将能够：

- 通过从未加入或未注册 Microsoft Entra 的 Windows 客户端连接到已加入 Microsoft Entra 的 Azure 虚拟桌面会话主机来验证它们的功能。

## 实验室文件

- 无

## 说明

### 练习 1：通过从 Windows 11 客户端连接到已加入 Microsoft Entra 的 Azure 虚拟桌面会话主机来验证其功能
  
此练习的主要任务如下：

1. 调整 Azure 虚拟桌面主机池 RDP 属性
1. 在 Windows 11 计算机上安装 Microsoft 远程桌面客户端
1. 订阅 Azure 虚拟桌面工作区
1. 测试 Azure 虚拟桌面应用


#### 任务 1：调整 Azure 虚拟桌面主机池的 RDP 属性

> **备注**：在上一实验室中实现的 RDP 设置可提供最佳用户体验（通过对单一登录的支持），但是，这需要进行[使用 Microsoft Entra ID 身份验证为 Azure 虚拟桌面配置单一登录](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on)中所述的其他更改。 如果没有这些更改，默认情况下，只要客户端计算机满足以下条件之一，则支持身份验证：

- 它是与会话主机加入到相同 Microsoft Entra 租户的 Microsoft Entra
- 它是与会话主机混合加入到相同 Microsoft Entra 租户的 Microsoft Entra
- 它是与会话主机注册到相同 Microsoft Entra 租户的 Microsoft Entra

由于这些条件都不适用于实验室计算机，因此必须将 `targetisaadjoined:i:1` 作为自定义 RDP 属性添加到主机池。

1. 如果需要，在实验室计算机上，启动 Web 浏览器，导航到 Azure 门户，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。

    > **备注**：使用实验室会话窗口右侧“资源”选项卡上列出的 `User1-` 帐户的凭据。

1. 在显示 Azure 门户的 Web 浏览器中，在 **az140-21-hp1** 页上的垂直菜单栏中的“**设置**”部分，选择“**RDP 属性**”条目。
1. 在 **az140-21-hp1 \| RDP 属性**页上，选择“**高级**”选项卡。 
1. 在 **az140-21-hp1 \| RDP 属性**页的“**高级**”选项卡上的“**RDP 属性**”文本框中，将以下字符串追加到现有内容中（如有需要，请确保添加前导分号字符 (`;`) 以将此字符串与它前面的字符串分开）：

    ```txt
    targetisaadjoined:i:1
    ```

1. 在“**RDP 属性**”文本框中，从现有内容中删除以下字符串（如有）（及其尾部的分号字符）：

    ```txt
    enablerdsaadauth:i:value
    ```

1. 在 **az140-21-hp1 \| RDP 属性**页上，选择“**保存**”。

#### 任务 2：在 Windows 11 计算机上安装 Microsoft 远程桌面客户端

1. 在实验室计算机中，启动 Web 浏览器，导航到“[使用适用于 Windows 的远程桌面客户端连接到 Azure 虚拟桌面](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows)”页，向下滚动到“**下载并安装远程桌面客户端 (MSI)**”部分，然后选择 [Windows 64 位](https://go.microsoft.com/fwlink/?linkid=2139369)链接。 
1. 打开文件资源管理器，导航到“**下载**”文件夹，并启动新下载的 MSI 文件的安装。 
1. 在“**远程桌面安装**”窗口中，当出现提示时，接受许可协议的条款并选择“**为本机的所有用户安装**”选项。 如果出现提示，请接受用户帐户控制提示以继续安装。
1. 安装完成后，请确保选中“**安装完成时启动远程桌面**”复选框，然后选择“**完成**”以启动远程桌面客户端。

   > **备注**：适用于 Windows 的[远程桌面存储应用](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store)不支持连接到已加入 Microsoft Entra 的会话主机。

#### 任务 3：订阅 Azure 虚拟桌面工作区

1. 在实验室计算机上，切换到**远程桌面**客户端窗口，选择“**订阅**”，并在出现提示时，使用 `User1` Entra ID 用户帐户的凭据登录，该帐户可在实验室界面窗口右侧窗格中的“**资源**”选项卡上找到。

   > **备注**：选择 Entra 组中带有 **AVD-DAG** 前缀的用户帐户。

   > **备注**：或者，在**远程桌面**客户端窗口中，选择“**通过 URL 订阅**”，在“**订阅工作区**”窗格的“**电子邮件或工作区 URL**”中键入 **https://client.wvd.microsoft.com/api/arm/feeddiscovery**，选择“**下一步**”，并在出现提示后，使用 Microsoft Entra 凭据登录。

1. 确保“**远程桌面**”页仅显示 **SessionDesktop** 图标。

   > **备注**：这是预期行为，因为所选 Microsoft Entra 用户帐户是在第一个实验室*使用 Azure 门户部署主机池和会话主机 (Entra ID)* 中分配给自动生成的 **az140-21-hp1-DAG** 桌面应用程序组的。

1. 在“**远程桌面**”页上，右键单击“**SessionDesktop**”图标，然后在弹出菜单中选择“**设置**”。
1. 在“**SessionDesktop**”窗格中，关闭“**使用默认设置**”开关。
1. 在“**显示设置**”部分的下拉菜单中，选择“**选择显示器**”项，然后选择要用于会话的显示器。
1. 在“**SessionDesktop**”窗格中，查看其余选项，包括“**最大化到当前显示器**”、“**在窗口模式下使用单个显示器**”和“**使会话适应窗口**”，而无需进行任何更改。 
1. 关闭“**SessionDesktop**”窗格。 
1. 在“**远程桌面**”页上，双击 **SessionDesktop** 图标。
1. 当系统提示登录时，在“**Windows 安全**”对话框中，输入第一个 Microsoft Entra 用户帐户的密码，该帐户在此任务中用于连接到目标 Azure 虚拟桌面环境。

   > **备注**：Azure 虚拟桌面不支持使用一个用户帐户登录到 Microsoft Entra ID，之后又使用另一个用户帐户登录到 Windows。 同时使用两个不同的帐户登录可能会导致用户重新连接到错误的会话主机、Azure 门户中的信息不正确或缺失，以及在使用应用附加或 MSIX 应用附加时出现错误消息。

   > **备注**：“**SessionDesktop**”窗口会自动显示。

1. 在“远程桌面”会话窗口中，验证你在会话中是否具有完全管理访问权限（例如，选择任务栏中的 **Windows** 徽标图标，然后从弹出菜单中选择“**Windows PowerShell（管理员）**”项。
1. 在“远程桌面”会话窗口中，选择任务栏中的 Windows 徽标图标，选择代表登录所用的 Microsoft Entra 用户帐户的头象图标，然后在弹出菜单中选择“**退出登录**”。

   > **备注**：这会自动终止远程桌面会话。 

1. 返回“**远程桌面**”窗口，选择 **az140-21-ws1** 工作区条目右侧的省略号 (`...`) 图标，选择“**取消订阅**”，并在系统提示确认时选择“**继续**”。
1. 在“**远程桌面**”客户端窗口中，选择“**订阅**”，然后在出现提示时，使用可在实验室界面窗口右侧窗格中的“**资源**”选项卡上找到的第二个 Entra ID 用户帐户的凭据进行登录。

   > **备注**：选择用户帐户，该用户帐户是具有 **AVD-RemoteApp** 前缀的 Entra 组的成员。

1. 确保“**远程桌面**”页显示四个图标，包括命令提示符、Microsoft Word、Microsoft Excel 和 Microsoft PowerPoint。 

   > **备注**：这是预期行为，因为所选的 Microsoft Entra 用户帐户是在第一个实验室*使用 Azure 门户部署主机池和会话主机 (Entra ID)* 中分配给 **az140-21-hp1-Office365-RAG** 和 **az140-21-hp1-Utilities-RAG** 应用程序组的。

1. 双击命令提示符图标。 
1. 当系统提示登录时，在“**Windows 安全**”对话框中，输入用于连接到目标 Azure 虚拟桌面环境的第二个 Microsoft Entra 用户帐户的密码。
1. 验证不久后是否会显示**命令提示符**窗口。 
1. 在命令提示符窗口中，键入**主机名**并按 **Enter** 键以显示运行命令提示符的计算机的名称。

   > **备注**：验证显示的名称是否以 **sh-** 前缀开头。

1. 在命令提示符下，键入“logoff”并按 Enter 键以从当前远程应用会话中注销 。
1. 双击“**远程桌面**”页上的剩余图标以启动 Microsoft Word、Microsoft Excel 和 Microsoft PowerPoint。
1. 关闭每个会话窗口。
