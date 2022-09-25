---
lab:
  title: 实验室：在 Windows Server 中实现和配置虚拟化
  type: Answer Key
  module: 'Module 5: Hyper-V virtualization in Windows Server'
---

# <a name="lab-answer-key-implementing-and-configuring-virtualization-in-windows-server"></a>实验室解答：在 Windows Server 中实现和配置虚拟化

### <a name="exercise-1-creating-and-configuring-vms"></a>练习 1：创建和配置 VM

#### <a name="task-1-create-a-hyper-v-virtual-switch"></a>任务 1：创建 Hyper-V 虚拟交换机

1. 连接到 SEA-ADM1，然后根据需要，以 Contoso\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“服务器管理器”  。
1. 在服务器管理器中，选择“所有服务器”。
1. 在服务器列表中，选择“SEA-SVR1”条目以显示其上下文菜单，然后选择“Hyper-V 管理器” 。
1. 在 Hyper-V 管理器中，确保已选中“SEA-SVR1.CONTOSO.COM”。
1. 在“操作”窗格中，选择“虚拟交换机管理器”。
1. 在“虚拟交换机管理器”中，在“创建虚拟交换机”窗格中选择“专用”，然后选择“创建虚拟交换机”   。
1. 在“虚拟交换机属性”框中，指定以下设置，然后选择“确定” ：

   - 名称：Contoso 专用交换机
   - 连接类型：专用网络

#### <a name="task-2-create-a-virtual-hard-disk"></a>任务 2：创建虚拟硬盘

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, in Hyper-V Manager connected to <bpt id="p2">**</bpt>SEA-SVR1<ept id="p2">**</ept>, select <bpt id="p3">**</bpt>New<ept id="p3">**</ept>, and then select <bpt id="p4">**</bpt>Hard Disk<ept id="p4">**</ept>. The <bpt id="p1">**</bpt>New Virtual Hard Disk Wizard<ept id="p1">**</ept> starts.
1. 在“开始之前”**** 页面上，选择“下一步”****。
1. 在“选择磁盘格式”页上，选择“VHD”，然后选择“下一步”  。
1. 在“选择磁盘类型”页上，选择“差异”，然后选择“下一步”  。
1. 在“指定名称和位置”页上，指定以下设置，然后选择“下一步” ：

   - 名称：SEA-VM1
   - 位置：C:\Base

1. 在“配置磁盘”页上的“位置”框中，输入 C:\Base\BaseImage.vhd，然后选择“下一步”   。
1. 在“摘要”页中，选择“完成” 。

#### <a name="task-3-create-a-virtual-machine"></a>任务 3：创建虚拟机

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, in Hyper-V Manager, select <bpt id="p2">**</bpt>New<ept id="p2">**</ept>, and then select <bpt id="p3">**</bpt>Virtual Machine<ept id="p3">**</ept>. The <bpt id="p1">**</bpt>New Virtual Machine Wizard<ept id="p1">**</ept> starts.
1. 在“开始之前”**** 页面上，选择“下一步”****。
1. 在“指定名称和位置”页上，输入“SEA-VM1”，然后选中“将虚拟机存储在其他位置”旁边的复选框  。
1. 在“位置”框中，输入 C:\Base，然后选择“下一步”  。
1. 在“指定代数”页面上，选择“第一代”，然后选择“下一步”  。
1. 在“分配内存”页上，输入 4096，然后选择“下一步”  。
1. 在“配置网络”页上，选择“连接”下拉列表，然后依次选择“Contoso 专用交换机”和“下一步”  。
1. 在“连接虚拟硬盘”页上，选择“使用现有虚拟硬盘”，然后选择“浏览”  。
1. 浏览到 C:\Base，选择“SEA-VM1.vhd”，选择“打开”，然后选择“下一步”   。
1. On the <bpt id="p1">**</bpt>Summary<ept id="p1">**</ept> page, select <bpt id="p2">**</bpt>Finish<ept id="p2">**</ept>. Notice that <bpt id="p1">**</bpt>SEA-VM1<ept id="p1">**</ept> displays in the Virtual Machines list.
1. 选择“SEA-VM1”，然后在“操作”窗格中的“SEA-VM1”下，选择“设置”  。
1. 在“硬件”列表中，选择“内存” 。
1. 在“动态内存”部分中，选中“启用动态内存”旁边的复选框 。
1. 在“最大 RAM”旁边输入 4096，然后选择“确定”  。
1. 关闭 HYPER-V 管理器。

#### <a name="task-4-manage-virtual-machines-using-windows-admin-center"></a>任务 4：使用 Windows Admin Center 管理虚拟机

1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. In the <bpt id="p1">**</bpt>Windows PowerShell<ept id="p1">**</ept> console, enter the following command. and then press Enter to download the latest version of Windows Admin Center:
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait until the installation completes. This should take about 2 minutes. If the web page does not respond, open <bpt id="p1">**</bpt>services.msc<ept id="p1">**</ept> and verify that the Windows Admin Center server is <bpt id="p2">**</bpt>Started<ept id="p2">**</ept>.

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 `https://SEA-ADM1.contoso.com`。 
1. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在“所有连接”窗格中，选择“+ 添加” 。
1. 在“添加或创建资源”窗格上的“服务器”磁贴中，选择“添加”  。
1. 在“服务器名称”文本框中，输入“sea-svr1.contoso.com” 。
1. 确保已选中“为此连接使用另一个帐户”选项，输入以下凭据，然后选择“使用凭据添加” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: After performing step 8, if an error message that says <bpt id="p2">**</bpt>You can add this server to your list of connections, but we can't confirm it's available.<ept id="p2">**</ept> appears, select <bpt id="p1">**</bpt>Add<ept id="p1">**</ept>. In the All Connections pane,  select <bpt id="p1">**</bpt>sea-svr1.contoso.com<ept id="p1">**</ept>, and then select <bpt id="p2">**</bpt>Manage as<ept id="p2">**</ept>. In the <bpt id="p1">**</bpt>Specify your credentials<ept id="p1">**</ept> dialog box, ensure that the <bpt id="p2">**</bpt>Use another account for this connection<ept id="p2">**</ept> option is selected, enter the Administrator credentials, and then select <bpt id="p3">**</bpt>Continue<ept id="p3">**</ept>.

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派。

1. 在“sea-svr1.contoso.com ”页上的“工具”列表中，依次选择“虚拟机”和“摘要”选项卡，然后查看其内容   。
1. 选择“清单”选项卡，并验证它是否包含两个虚拟机的列表。
1. 选择“SEA-VM1”，并查看其“属性”窗格。
1. 选择“设置”，然后选择“磁盘” 。
1. 滚动到窗格底部，并选择“+ 添加磁盘”。
1. 选择“新建虚拟硬盘”。
1. 在“新建虚拟硬盘”窗格的“大小(GB)”文本框中，键入 5，保留其他设置的默认值，然后选择“创建”   。
1. 选择“保存磁盘设置”，然后选择“关闭” 。
1. 返回到 SEA-VM1 的“属性”窗格，选择“电源”，然后选择“启动”以启动 SEA-VM1    。
1. 向下滚动并显示正在运行的 VM 的统计信息。
1. 刷新页面，依次选择“电源”和“关机”，然后选择“是”进行确认  。
1. 在“工具”列表中，选择“虚拟交换机”，并验证窗格中是否显示两个交换机 。

### <a name="exercise-1-results"></a>练习 1 结果

完成本练习后，你应该已使用 Hyper-V 管理器和 Windows Admin Center 来创建虚拟交换机、虚拟硬盘和虚拟机，然后管理虚拟机。

### <a name="exercise-2-installing-and-configuring-containers"></a>练习 2：安装和配置容器

#### <a name="task-1-install-docker-on-windows-server"></a>任务 1：在 Windows Server 上安装 Docker

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, in the <bpt id="p2">**</bpt>Tools<ept id="p2">**</ept> listing for <bpt id="p3">**</bpt>SEA-SVR1<ept id="p3">**</ept>, select <bpt id="p4">**</bpt>PowerShell<ept id="p4">**</ept>. When prompted, type <bpt id="p1">**</bpt>Pa55w.rd<ept id="p1">**</ept> to authenticate using the <bpt id="p2">**</bpt>CONTOSO<ph id="ph1">\\</ph>Administrator<ept id="p2">**</ept> user account, and then press Enter. 

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

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Identify the IPv4 address of the Ethernet adapter named vEthernet (nat). This is the address of the new container. Next, identify the IPv4 address of the Ethernet adapter named <bpt id="p1">**</bpt>Ethernet<ept id="p1">**</ept>. This is the IP address of the host (<bpt id="p1">**</bpt>SEA-SVR1<ept id="p1">**</ept>) and is set to <bpt id="p2">**</bpt>172.16.10.12<ept id="p2">**</ept>.

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, switch to the Microsoft Edge window, open another tab and go to <bpt id="p2">**</bpt><ph id="ph1">http://172.16.10.12</ph><ept id="p2">**</ept>. Verify that the browser displays the default IIS page.
1. 在 SEA-ADM1 上，切换回与 SEA-SVR1 的 PowerShell 远程处理会话，然后在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，列出正在运行的容器  ：

   ```powershell
   docker ps
   ```
   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: This command provides information on the container that is currently running on <bpt id="p2">**</bpt>SEA-SVR1<ept id="p2">**</ept>. Record the container ID because you will use it to stop the container. 

1. 输入以下命令然后按 Enter，停止正在运行的容器（将 `<ContainerID>` 占位符替换为你在上一步中标识的容器 ID）： 

   ```powershell
   docker stop <ContainerID>
   ```
1. 输入以下命令然后按 Enter，验证容器是否已停止：

   ```powershell
   docker ps
   ```

#### <a name="task-3-use-windows-admin-center-to-manage-containers"></a>任务 3：使用 Windows Admin Center 管理容器

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, in the Windows Admin Center, in the Tools menu of <bpt id="p2">**</bpt>sea-svr1.contoso.com<ept id="p2">**</ept>, select <bpt id="p3">**</bpt>Containers<ept id="p3">**</ept>. When prompted to close the <bpt id="p1">**</bpt>PowerShell<ept id="p1">**</ept> session, select <bpt id="p2">**</bpt>Continue<ept id="p2">**</ept>.
1. 在“容器”窗格中，浏览“概述”、“容器”、“映像”、“网络”和“卷”选项卡    。

### <a name="exercise-2-results"></a>练习 2 结果

完成本练习后，你应该已在 Windows Server 上安装了 Docker，下载了包含 Web 服务的 Windows 容器映像，并验证了其功能。
