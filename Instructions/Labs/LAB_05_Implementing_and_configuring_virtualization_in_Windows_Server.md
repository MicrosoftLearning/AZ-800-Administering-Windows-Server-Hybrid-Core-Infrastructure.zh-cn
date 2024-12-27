---
lab:
  title: 实验室：在 Windows Server 中实现和配置虚拟化
  module: 'Module 5: Hyper-V virtualization in Windows Server'
---

# 实验室：在 Windows Server 中实现和配置虚拟化

## 场景

Contoso 是一家全球工程和制造公司，其总部位于美国西雅图市。 IT 办公室和数据中心位于西雅图，以支持西雅图位置和其他位置。 Contoso 最近部署了一个 Windows Server 服务器和客户端基础结构。 

由于目前许多物理服务器未得到充分利用，公司计划扩展虚拟化以优化环境。 因此，你决定执行概念证明来验证如何使用 Hyper-V 管理虚拟机环境。 另外，Contoso DevOps 团队想要探索容器技术，确定它是否有助于减少新应用程序的部署时间，并简化将应用程序移动到云的过程。 你计划与该团队合作，评估 Windows 服务器容器并考虑在容器中提供 Internet Information Services（Web 服务）。

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Implementing%20and%20configuring%20virtualization%20in%20Windows%20Server)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## 目标

完成本实验室后，你将能够：

- 创建和配置 VM。
- 安装并配置容器。

## 预计时间：60 分钟

## 实验室设置

虚拟机：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1 和 AZ-800T00A-ADM1 必须正在运行  。 其他 VM 可以运行，但此实验室不需要这些 VM。

> 注意：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1 和 AZ-800T00A-SEA-ADM1 虚拟机承载 SEA-DC1、SEA-SVR1 和 SEA-ADM1 的安装

1. 选择“SEA-ADM1”。
1. 使用讲师提供的凭据进行登录。

对于本实验室，你将使用可用的 VM 环境。

## 练习 1：创建和配置 VM

### 场景

在此练习中，使用 Hyper-V 管理器和 Windows Admin Center 来创建和配置虚拟机。 首先，你将创建一个专用虚拟网络交换机。 接下来，你决定创建一个基础映像的差分驱动器，该驱动器已准备好要安装到 VM 上的操作系统。 最后，你将创建第 1 代 VM，该 VM 使用已为概念证明准备的差分驱动器和专用交换机。

此练习的主要任务如下：

1. 创建 Hyper-V 虚拟交换机
1. 创建虚拟硬盘
1. 创建虚拟机
1. 使用 Windows Admin Center 管理虚拟机

#### 任务 1：创建 Hyper-V 虚拟交换机

1. 在 SEA-ADM1 上，打开服务器管理器 。 
1. 在服务器管理器中，选择“所有服务器”。 
1. 在“服务器”列表中，选择 SEA-SVR1，并使用其上下文相关菜单来启动针对该服务器的 Hyper-V 管理器 。 
1. 在 Hyper-V 管理器中，使用“虚拟交换机管理器”在 SEA-SVR1 上创建一个具有以下设置的虚拟交换机  ：

   - 名称：Contoso 专用交换机
   - 连接类型：专用网络

#### 任务 2：创建虚拟硬盘

1. 在 SEA-ADM1 上的 Hyper-V 管理器中，使用“新建虚拟硬盘向导”在 SEA-SVR1 上创建一个具有以下设置的新虚拟硬盘  ： 

   - 磁盘格式：VHD
   - 磁盘类型：差分
   - 名称：SEA-VM1
   - 位置：C:\Base
   - 父磁盘：C:\Base\BaseImage.vhd

#### 任务 3：创建虚拟机

1. 在 SEA-ADM1 上的 Hyper-V 管理器中，在 SEA-SVR1 上创建一个具有以下设置的新虚拟机 ： 

   - 名称：SEA-VM1
   - 位置：C:\Base
   - 代系：第 1 代
   - 内存：4096
   - 网络：Contoso 专用交换机
   - 硬盘：C:\Base\SEA-VM1.vhd

1. 打开 SEA-VM1 的“设置”窗口并启用“动态内存”，最大 RAM 值为 4096   。
1. 关闭 HYPER-V 管理器。

#### 任务 4：使用 Windows Admin Center 管理虚拟机

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，运行以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 备注：请等待安装完成。 这大约需要 2 分钟。

1. 在 SEA-ADM1 上，启动 Microsoft Edge 并连接到 Windows Admin Center 的本地实例 (`https://SEA-ADM1.contoso.com`)。 
1. 如果出现提示，请在“**Windows 安全**”对话框中输入讲师提供的凭据，然后选择“**确定**”。

1. 在 Windows Admin Center 中，添加与 **sea-svr1.contoso.com** 的连接，并使用讲师提供的凭据与之连接。
1. 在“工具”列表中，选择“虚拟机”，并查看“摘要”窗格  。
1. 在“库存”窗格中，打开“SEA-VM1”并查看“设置”  。
1. 使用 Windows Admin Center 创建新磁盘，大小为 5 GB。
1. 使用 Windows Admin Center 启动 SEA-VM1，然后显示正在运行的 VM 的统计信息。
1. 使用 Windows Admin Center 关闭 SEA-VM1。
1. 在“工具”列表中，选择“虚拟交换机”并标识现有交换机 。

### 练习 1 结果

完成本练习后，你应该已使用 Hyper-V 管理器和 Windows Admin Center 来创建虚拟交换机、虚拟硬盘和虚拟机，然后管理虚拟机。

## 练习 2：安装和配置容器

### 场景

在本练习中，你将使用 Docker 来安装和运行 Windows 容器。 你还将使用 Windows Admin Center 来管理容器。

此练习的主要任务如下：

1. 在 Windows Server 上安装 Docker
1. 安装并运行 Windows 容器
1. 使用 Windows Admin Center 管理容器

#### 任务 1：在 Windows Server 上安装 Docker

1. 在 SEA-ADM1 上，在 Windows Admin Center 中，当连接到 sea-svr1.contoso.com 时，使用“工具”菜单与该服务器建立 PowerShell 远程处理会话  。 

   > 注意：由于实验室中使用了嵌套虚拟化，Windows Admin Center 中的 Powershell 连接可能相对较慢，因此另一种方法是从 SEA-ADM1 上的 Windows Powershell 控制台运行 `Enter-PSSession -ComputerName SEA-SVR1` 。

1. 在 PowerShell 控制台中，运行以下命令，以强制使用 TLS 1.2 并安装 PowerShellGet 模块： 

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
   Install-PackageProvider -Name NuGet -Force
   Install-Module PowerShellGet -RequiredVersion 2.2.4 -SkipPublisherCheck
   ```
1. 安装完成后，运行以下命令以重启 SEA-SVR1：

   ```powershell
   Restart-Computer -Force
   ```
1. 在 SEA-SVR1 重启后，再次使用 PowerShell 工具与 SEA-SVR1 建立新的 PowerShell 远程处理会话  。

1. 在 Windows PowerShell 控制台中，运行以下命令，在 SEA-SVR1 上安装 Docker Microsoft PackageManagement 提供程序 ：

   ```powershell
   Install-Module -Name DockerProvider -Repository PSGallery -Force
   ```
1. 在 Windows PowerShell 控制台中，运行以下命令，在 SEA-SVR1 上安装 Docker 运行时 ：

   ```powershell
   Install-Package -Name docker -ProviderName DockerProvider
   ```
1. 安装完成后，运行以下命令以重启 SEA-SVR1：

   ```powershell
   Restart-Computer -Force
   ```

#### 任务 2：安装并运行 Windows 容器

1. 在 SEA-SVR1 重启后，再次使用 PowerShell 工具与 SEA-SVR1 建立新的 PowerShell 远程处理会话  。
1. 在 Windows PowerShell 控制台中，运行以下命令以验证安装的 Docker 版本：

   ```powershell
   Get-Package -Name Docker -ProviderName DockerProvider
   ```
1. 运行以下命令，标识 SEA-SVR1 上当前存在的 Docker 映像： 

   ```powershell
   docker images
   ```

   > 注意：确认本地存储库存储中没有映像。

1. 运行以下命令，下载包含 Internet Information Services (IIS) 安装的 Nano Server 映像：

   ```powershell
   docker pull nanoserver/iis
   ```

   > 注意：完成下载所用的时间取决于从实验室 VM 到 Microsoft 容器注册表的网络连接的可用带宽。

1. 运行以下命令，确认已成功下载 Docker 映像：

   ```powershell
   docker images
   ```
1. 运行以下命令，根据下载的映像启动容器：

   ```powershell
   docker run --isolation=hyperv -d -t --name nano -p 80:80 nanoserver/iis 
   ```

   > 注意：docker 命令以 Hyper-V 隔离模式（可解决任何主机操作系统不兼容问题）作为后台服务 (`-d`) 启动容器，并配置网络以使容器主机的端口 80 映射到容器的端口 80。 

1. 运行以下命令，检索容器主机的 IP 地址信息：

   ```powershell
   ipconfig
   ```

   > 注意：标识名为 vEthernet (nat) 的以太网适配器的 IPv4 地址。 这是新容器的地址。 接下来，标识名为 Ethernet 的以太网适配器的 IPv4 地址。 这是主机 (SEA-SVR1) 的 IP 地址，设置为 172.16.10.12 。

1. 在 SEA-ADM1 上，切换到 Microsoft Edge 窗口，打开另一个选项卡并转到 http://172.16.10.12 。 确认浏览器显示默认 IIS 页。
1. 在 SEA-ADM1 上，切换回与 SEA-SVR1 的 PowerShell 远程处理会话，然后在 Windows PowerShell 控制台中，运行以下命令以列出正在运行的容器  ：

   ```powershell
   docker ps
   ```
   > 注意：此命令提供有关当前正在 SEA-SVR1 上运行的容器的信息 。 记录容器 ID，因为你将使用它来停止容器。 

1. 运行以下命令以停止正在运行的容器（将 `<ContainerID>` 占位符替换为你在上一步中标识的容器 ID）：

   ```powershell
   docker stop <ContainerID>
   ```
1. 运行以下命令，确认容器已停止：

   ```powershell
   docker ps
   ```

#### 任务 3：使用 Windows Admin Center 管理容器

1. 在 SEA-ADM1 上，在 Windows Admin Center 中 sea-svr1.contoso.com 的“工具”菜单中，选择“容器”   。 系统提示关闭 PowerShell 会话时，选择“继续” 。
1. 在“容器”窗格中，浏览“概述”、“容器”、“映像”、“网络”和“卷”选项卡    。

### 练习 2 结果

完成本练习后，你应该已在 Windows Server 上安装了 Docker，下载了包含 Web 服务的 Windows 容器映像，并验证了其功能。
