---
lab:
  title: 实验室：在 Windows Server 中实现和配置虚拟化
  type: Answer Key
  module: 'Module 5: Hyper-V virtualization in Windows Server'
ms.openlocfilehash: 9202bf8b30582e0c3d8aebfddf7ca0762f53024a
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906920"
---
# <a name="lab-answer-key-implementing-and-configuring-virtualization-in-windows-server"></a>实验室解答：在 Windows Server 中实现和配置虚拟化

### <a name="exercise-1-creating-and-configuring-vms"></a>练习 1：创建和配置 VM

#### <a name="task-1-create-a-hyper-v-virtual-switch"></a>任务 1：创建 Hyper-V 虚拟交换机

1. 连接到 SEA-ADM1，然后根据需要，以 Contoso\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“服务器管理器”  。
1. 在服务器管理器中，选择“所有服务器”。
1. 在服务器列表中，选择“SEA-SVR1”条目以显示其上下文菜单，然后选择“Hyper-V 管理器” 。
1. 在 Hyper-V 管理器中，确保已选中“SEA-SVR1.Contoso.com”框。
1. 在“操作”窗格中，选择“虚拟交换机管理器”。
1. 在“虚拟交换机管理器”中，在“创建虚拟交换机”窗格中选择“专用”，然后选择“创建虚拟交换机”   。
1. 在“虚拟交换机属性”框中，指定以下设置，然后选择“确定” ：

   - 名称：Contoso 专用交换机
   - 连接类型：专用网络

#### <a name="task-2-create-a-virtual-hard-disk"></a>任务 2：创建虚拟硬盘

1. 在 SEA-ADM1 上，在连接到“SEA-SVR1”的 Hyper-V 管理器中，选择“新建”，然后选择“硬盘”   。 此时会启动“新建虚拟硬盘向导”。
1. 在“准备工作”页中，选择“下一步” 。
1. 在“选择磁盘格式”页上，选择“VHD”，然后选择“下一步”  。
1. 在“选择磁盘类型”页上，选择“差异”，然后选择“下一步”  。
1. 在“指定名称和位置”页上，指定以下设置，然后选择“下一步” ：

   - 名称：SEA-VM1
   - 位置：C:\Base

1. 在“配置磁盘”页上的“位置”框中，输入 C:\Base\BaseImage.vhd，然后选择“下一步”   。
1. 在“摘要”页中，选择“完成” 。

#### <a name="task-3-create-a-virtual-machine"></a>任务 3：创建虚拟机

1. 在 SEA-ADM1 上，在 Hyper-V 管理器中，选择“新建”，然后选择“虚拟机”  。 “新建虚拟机向导”随即启动。
1. 在“准备工作”页中，选择“下一步” 。
1. 在“指定名称和位置”页上，输入“SEA-VM1”，然后选中“将虚拟机存储在其他位置”旁边的复选框  。
1. 在“位置”框中，输入 C:\Base，然后选择“下一步”  。
1. 在“指定代数”页面上，选择“第一代”，然后选择“下一步”  。
1. 在“分配内存”页上，输入 4096，然后选择“下一步”  ：
1. 在“配置网络”页上，选择“连接”下拉列表，然后依次选择“Contoso 专用交换机”和“下一步”  。
1. 在“连接虚拟硬盘”页上，选择“使用现有虚拟硬盘”，然后选择“浏览”  。
1. 浏览到 C:\Base，选择“SEA-VM1.vhd”，选择“打开”，然后选择“下一步”   。
1. 在“摘要”页中，选择“完成” 。 请注意，“SEA-VM1”显示在虚拟机列表中。
1. 选择“SEA-VM1”，然后在“操作”窗格中的“SEA-VM1”下，选择“设置”  。
1. 在“硬件”列表中，选择“内存” 。
1. 在“动态内存”部分中，选中“启用动态内存”旁边的复选框 。
1. 在“最大 RAM”旁边输入 4096，然后选择“确定”  。
1. 关闭 HYPER-V 管理器。

#### <a name="task-4-manage-virtual-machines-using-windows-admin-center"></a>任务 4：使用 Windows Admin Center 管理虚拟机

1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，输入以下命令。 然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 注意：请等待安装完成。 这大约需要 2 分钟。

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 `https://SEA-ADM1.contoso.com`。 
1. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在“所有连接”窗格中，选择“+ 添加” 。
1. 在“添加或创建资源”窗格上的“服务器”磁贴中，选择“添加”  
1. 在“服务器名称”文本框中，输入“sea-svr1.contoso.com” 。
1. 确保已选中“为此连接使用另一个帐户”选项，输入以下凭据，然后选择“使用凭据添加” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

   > 注意：执行步骤 8 后，如果出现“你可将此服务器添加到你的连接列表，但我们无法确认它是否可用。”  错误消息，请选择“添加”。 在“所有连接”窗格中，选择“sea-svr1.contoso.com”，然后选择“管理形式” 。 在“指定凭据”对话框中，确保已选中“对此连接使用另一个帐户”选项，输入管理员凭据，然后选择“继续”  。

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派。

1. 在“sea-svr1.contoso.com ”页上的“工具”列表中，依次选择“虚拟机”和“摘要”选项卡，然后查看其内容   。
1. 选择“清单”选项卡，并验证它是否包含两个虚拟机的列表。
1. 选择“SEA-VM1”，并查看其“属性”窗格。
1. 选择“设置”，然后选择“磁盘” 。
1. 滚动到窗格底部，并选择“+ 添加磁盘”。
1. 选择“新建虚拟硬盘”。
1. 在“新建虚拟硬盘”窗格的“大小(GB)”文本框中，键入 5，保留其他设置的默认值，然后选择“创建”  。
1. 选择“保存磁盘设置”，然后选择“关闭” 。
1. 返回到 SEA-VM1 的“属性”窗格，选择“电源”，然后选择“启动”以启动 SEA-VM1    。
1. 向下滚动并显示正在运行的 VM 的统计信息。
1. 刷新页面，依次选择“电源”和“关机”，然后选择“是”进行确认  。
1. 在“工具”列表中，选择“虚拟交换机”，并验证窗格中是否显示两个交换机 。

### <a name="exercise-1-results"></a>练习 1 结果

完成本练习后，你应该已使用 Hyper-V 管理器和 Windows Admin Center 来创建虚拟交换机、虚拟硬盘和虚拟机，然后管理虚拟机。

### <a name="exercise-2-installing-and-configuring-containers"></a>练习 2：安装和配置容器

#### <a name="task-1-install-docker-on-windows-server"></a>任务 1：在 Windows Server 上安装 Docker

1. 在 SEA-ADM1 上，在 SEA-SVR1 的“工具”列表中，选择“PowerShell”   。 出现提示时，键入 Pa55w.rd，使用 CONTOSO\\Administrator 用户帐户进行身份验证，然后按 Enter 。 

   > 注意：这将建立与 SEA-SVR1 的 PowerShell 远程处理连接。

   > 注意：由于实验室中使用了嵌套虚拟化，Windows Admin Center 中的 Powershell 连接可能相对较慢，因此另一种方法是从 SEA-ADM1 上的 Windows Powershell 控制台运行 `Enter-PSSession -ComputerName SEA-SVR1` 。

1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，以强制使用 TLS 1.2 并安装 PowerShellGet 模块：

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
   Install-PackageProvider -Name NuGet -Force
   Install-Module PowerShellGet -RequiredVersion 2.2.4 -SkipPublisherCheck
   ```
1. 当系统提示确认从不受信任的存储库安装模块时，请按 A 键，然后按 Enter。
1. 安装完成后，输入以下命令然后按 Enter，重启 SEA-SVR1：

   ```powershell
   Restart-Computer -Force
   ```
1. 在 SEA-SVR1 重启后，再次使用 PowerShell 工具与 SEA-SVR1 建立新的 PowerShell 远程处理会话  。
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，在 SEA-SVR1 上安装 Docker Microsoft PackageManagement 提供程序 ：

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```
1. 在 NuGet 提供程序提示符处，按 Y 键，然后按 Enter。
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，在 SEA-SVR1 上安装 Docker 运行时 ：

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```
1. 出现确认提示时，按 A 键，然后按 Enter。
1. 安装完成后，输入以下命令然后按 Enter，重启 SEA-SVR1：

   ```powershell
   Restart-Computer -Force
   ```

#### <a name="task-2-install-and-run-a-windows-container"></a>任务 2：安装并运行 Windows 容器

1. 在 SEA-SVR1 重启后，再次使用 PowerShell 工具与 SEA-SVR1 建立新的 PowerShell 远程处理会话 。
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，验证已安装 Docker 版本：

   ```powershell
   Get-Package -Name Docker -ProviderName DockerMsftProvider
   ```
1. 输入以下命令然后按 Enter，识别 SEA-SVR1 上当前存在的 Docker 映像： 

   ```powershell
   docker images
   ```

   > 注意：确认本地存储库存储中没有映像。

1. 输入以下命令然后按 Enter，列出联机 Microsoft 存储库中的 Docker 基础映像：

   ```powershell
   docker search Microsoft
   ```
1. 输入以下命令然后按 Enter，下载包含 Internet Information Services (IIS) 安装的 Nano Server 映像：

   ```powershell
   docker pull nanoserver/iis
   ```

   > 注意：完成下载所用的时间取决于从实验室 VM 到 Microsoft 容器注册表的网络连接的可用带宽。

1. 输入以下命令然后按 Enter，确认已成功下载 Docker 映像：

   ```powershell
   docker images
   ```
1. 输入以下命令然后按 Enter，启动基于已下载映像的容器：

   ```powershell
   docker run --isolation=hyperv -d -t --name nano -p 80:80 nanoserver/iis 
   ```

   > 注意：docker 命令以 Hyper-V 隔离模式（可解决任何主机操作系统不兼容问题）作为后台服务 (`-d`) 启动容器，并配置网络以使容器主机的端口 80 映射到容器的端口 80。 

1. 输入以下命令然后按 Enter，检索容器主机的 IP 地址信息：

   ```powershell
   ipconfig
   ```

   > 注意：标识名为 vEthernet (nat) 的以太网适配器的 IPv4 地址。 这是新容器的地址。 接下来，标识名为 Ethernet 的以太网适配器的 IPv4 地址。 这是主机 (SEA-SVR1) 的 IP 地址，设置为 172.16.10.12 。

1. 在 SEA-ADM1 上，切换到 Microsoft Edge 窗口，打开另一个选项卡并转到 http://172.16.10.12 。 确认浏览器显示默认 IIS 页。
1. 在 SEA-ADM1 上，切换回与 SEA-SVR1 的 PowerShell 远程处理会话，然后在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，列出正在运行的容器  ：

   ```powershell
   docker ps
   ```
   > 注意：此命令提供有关当前正在 SEA-SVR1 上运行的容器的信息 。 记录容器 ID，因为你将使用它来停止容器。 

1. 输入以下命令然后按 Enter，停止正在运行的容器（将 `<ContainerID>` 占位符替换为你在上一步中标识的容器 ID）： 

   ```powershell
   docker stop <ContainerID>
   ```
1. 输入以下命令然后按 Enter，验证容器是否已停止：

   ```powershell
   docker ps
   ```

#### <a name="task-3-use-windows-admin-center-to-manage-containers"></a>任务 3：使用 Windows Admin Center 管理容器

1. 在 SEA-ADM1 上，在 Windows Admin Center 中 sea-svr1.contoso.com 的“工具”菜单中，选择“容器”  。 系统提示关闭 PowerShell 会话时，选择“继续” 。
1. 在“容器”窗格中，浏览“摘要”、“容器”、“映像”、“网络”和“卷”选项卡    。

### <a name="exercise-2-results"></a>练习 2 结果

完成本练习后，你应该已在 Windows Server 上安装了 Docker，下载了包含 Web 服务的 Windows 容器映像，并验证了其功能。