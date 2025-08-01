---
lab:
  title: 实验室：在 Windows Server 中实现和配置虚拟化
  type: Answer Key
  module: 'Module 5: Hyper-V virtualization in Windows Server'
---

# 实验室解答：在 Windows Server 中实现和配置虚拟化

### 练习 1：创建和配置 VM

#### 任务 1：创建 Hyper-V 虚拟交换机

1. 连接到 **SEA-ADM1**，如果需要，请使用讲师提供的凭据登录。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“服务器管理器”  。
1. 在服务器管理器中，选择“所有服务器”。
1. 在服务器列表中，选择“SEA-SVR1”条目以显示其上下文菜单，然后选择“Hyper-V 管理器” 。
1. 在 Hyper-V 管理器中，确保已选中“SEA-SVR1.CONTOSO.COM”。
1. 在“操作”窗格中，选择“虚拟交换机管理器”。
1. 在“虚拟交换机管理器”中，在“创建虚拟交换机”窗格中选择“专用”，然后选择“创建虚拟交换机”   。
1. 在“虚拟交换机属性”框中，指定以下设置，然后选择“确定” ：

   - 名称：Contoso 专用交换机
   - 连接类型：专用网络

#### 任务 2：创建虚拟硬盘

1. 在 SEA-ADM1 上，在连接到“SEA-SVR1”的 Hyper-V 管理器中，选择“新建”，然后选择“硬盘”   。 此时会启动“新建虚拟硬盘向导”。
1. 在“开始之前”**** 页面上，选择“下一步”****。
1. 在“选择磁盘格式”页上，选择“VHD”，然后选择“下一步”  。
1. 在“选择磁盘类型”页上，选择“差异”，然后选择“下一步”  。
1. 在“指定名称和位置”页上，指定以下设置，然后选择“下一步” ：

   - 名称：SEA-VM1
   - 位置：C:\Base

1. 在“配置磁盘”页上的“位置”框中，输入 C:\Base\BaseImage.vhd，然后选择“下一步”   。
1. 在“摘要”页中，选择“完成” 。

#### 任务 3：创建虚拟机

1. 在 SEA-ADM1 上，在 Hyper-V 管理器中，选择“新建”，然后选择“虚拟机”  。 “新建虚拟机向导”随即启动。
1. 在“开始之前”**** 页面上，选择“下一步”****。
1. 在“指定名称和位置”页上，输入“SEA-VM1”，然后选中“将虚拟机存储在其他位置”旁边的复选框  。
1. 在“位置”框中，输入 C:\Base，然后选择“下一步”  。
1. 在“指定代数”页面上，选择“第一代”，然后选择“下一步”  。
1. 在“分配内存”页上，输入 4096，然后选择“下一步”  。
1. 在“配置网络”页上，选择“连接”下拉列表，然后依次选择“Contoso 专用交换机”和“下一步”  。
1. 在“连接虚拟硬盘”页上，选择“使用现有虚拟硬盘”，然后选择“浏览”  。
1. 浏览到 C:\Base，选择“SEA-VM1.vhd”，选择“打开”，然后选择“下一步”   。
1. 在“摘要”页中，选择“完成” 。 请注意，“SEA-VM1”显示在虚拟机列表中。
1. 选择“SEA-VM1”，然后在“操作”窗格中的“SEA-VM1”下，选择“设置”  。
1. 在“硬件”列表中，选择“内存” 。
1. 在“动态内存”部分中，选中“启用动态内存”旁边的复选框 。
1. 在“最大 RAM”旁边输入 4096，然后选择“确定”  。
1. 关闭 HYPER-V 管理器。

#### 任务 4：使用 Windows Admin Center 管理虚拟机

1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，输入以下命令。 然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   $parameters = @{
     Source = "https://aka.ms/WACdownload"
     Destination = ".\WindowsAdminCenter.exe"
     }
   Start-BitsTransfer @parameters
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process -FilePath '.\WindowsAdminCenter.exe' -ArgumentList '/VERYSILENT' -Wait
   ```

   > 备注：请等待安装完成。 这大约需要 2 分钟。 如果网页没有响应，请打开 services.msc 并验证 Windows Admin Center 服务器是否已启动 。

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 `https://SEA-ADM1.contoso.com`。 
   
   >注意：如果链接不起作用，请在 SEA-ADM1 上打开文件资源管理器，选择“下载”文件夹，在“下载”文件夹中选择 WindowsAdminCenter.msi 文件并手动安装。 安装完成后，刷新 Microsoft Edge。

   >注意：如果收到 NET::ERR_CERT_DATE_INVALID 错误，请在 Microsoft Edge 浏览器页上选择“高级”，在页面底部选择“继续访问 sea-adm1-contoso.com (不安全)” 。
   
1. 如果出现提示，请在“**Windows 安全**”对话框中输入讲师提供的凭据，然后选择“**确定**”。

1. 查看“配置 Windows Admin Center 设置和环境”弹出窗口上的所有选项卡，包括“扩展”选项卡，然后选择“完成”以关闭窗口。************
1. 查看“此版本中的新增功能”弹出窗口，然后在其右上角选择“关闭” 。

1. 在“所有连接”窗格中，选择“+ 添加” 。
1. 在“添加或创建资源”窗格上的“服务器”磁贴中，选择“添加”  。
1. 在“服务器名称”文本框中，输入“sea-svr1.contoso.com” 。
1. 确保已选中“**为此连接使用另一个帐户**”选项，输入讲师提供的凭据，然后选择“**使用凭据添加**”：

   > 注意：执行步骤 8 后，如果出现“你可将此服务器添加到你的连接列表，但我们无法确认它是否可用。” 错误消息，请选择“添加”。 在“所有连接”窗格中，选择“sea-svr1.contoso.com”，然后选择“管理形式” 。 在“指定凭据”对话框中，确保已选中“对此连接使用另一个帐户”选项，输入管理员凭据，然后选择“继续”  。

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派。

1. 在“sea-svr1.contoso.com ”页上的“工具”列表中，依次选择“虚拟机”和“摘要”选项卡，然后查看其内容   。
1. 选择“清单”选项卡并验证它包含 SEA-VM1。
1.  选择“SEA-VM1”，并查看其“属性”窗格。
1. 选择“设置”，然后选择“磁盘” 。
1. 滚动到窗格底部，并选择“+ 添加磁盘”。
1. 选择“新建虚拟硬盘”。
1. 在“新建虚拟硬盘”窗格的“大小(GB)”文本框中，键入 5，保留其他设置的默认值，然后选择“创建”   。
1. 选择“保存磁盘设置”，然后选择“关闭” 。
1. 返回到 SEA-VM1 的“属性”窗格，选择“电源”，然后选择“启动”以启动 SEA-VM1    。
1. 向下滚动并显示正在运行的 VM 的统计信息。
1. 刷新页面，依次选择“电源”和“关机”，然后选择“是”进行确认  。
1. 在“工具”列表中，选择“虚拟交换机”并标识现有交换机 。

### 练习 1 结果

完成本练习后，你应该已使用 Hyper-V 管理器和 Windows Admin Center 来创建虚拟交换机、虚拟硬盘和虚拟机，然后管理虚拟机。

### 练习 2：安装和配置容器

#### 任务 1：在 Windows Server 上安装 Docker

1. 在 SEA-ADM1 上，在 SEA-SVR1 的“工具”列表中，选择“PowerShell”   。 出现提示时，使用讲师提供的凭据进行验证，然后按 Enter。 

   > 注意：这将建立与 SEA-SVR1 的 PowerShell 远程处理连接。

   > 注意：由于实验室中使用了嵌套虚拟化，Windows Admin Center 中的 Powershell 连接可能相对较慢，因此另一种方法是从 SEA-ADM1 上的 Windows Powershell 控制台运行 `Enter-PSSession -ComputerName SEA-SVR1` 。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter，在 SEA-SVR1 上安装 Docker CE（社区版）********：

   ```powershell
   Invoke-WebRequest -UseBasicParsing "https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1" -o install-docker-ce.ps1

   .\install-docker-ce.ps1
   ```
 
1. 安装完成后，输入以下命令然后按 Enter，重启 SEA-SVR1：

   ```powershell
   Restart-Computer -Force
   ```

#### 任务 2：安装并运行 Windows 容器

1. 在 SEA-SVR1 重启后，再次使用 PowerShell 工具与 SEA-SVR1 建立新的 PowerShell 远程处理会话 。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter，识别 SEA-SVR1 上当前存在的 Docker 映像********： 

   ```powershell
   docker images
   ```

   > 注意：确认本地存储库存储中没有映像。

1. 输入以下命令，然后按 Enter，下载 Nano Server 映像：

   ```powershell
   docker pull mcr.microsoft.com/windows/nanoserver:ltsc2022
   ```

   > 注意：完成下载所用的时间取决于从实验室 VM 到 Microsoft 容器注册表的网络连接的可用带宽。

1. 输入以下命令然后按 Enter，确认已成功下载 Docker 映像：

   ```powershell
   docker images
   ```

1. 输入以下命令然后按 Enter，启动基于已下载映像的容器：

   ```powershell
   docker run -it mcr.microsoft.com/windows/nanoserver:ltsc2022 cmd.exe 
   ```

   > **注意**：该 docker 命令会启动容器并将你连接到容器的命令行接口。 

1. 输入以下命令然后按 Enter，检索容器主机的 IP 地址信息：

   ```powershell
   hostname
   ```
    > **注意**：确认这是容器实例的主机名，而不是 SEA-SVR1****。

1. 输入以下命令，然后按 Enter 为容器创建文本文件：

   ```powershell
   echo "Hello World!" > C:\Users\Public\Hello.txt
   ```

1. 输入以下命令，然后按 Enter 退出容器的命令行接口，并返回到 SEA-SVR1 上的 PowerShell 提示符****：

   ```powershell
   exit
   ```

1. 输入以下命令，然后按 Enter，通过运行 docker ps 命令获取刚从其退出的容器的容器 ID：

   ```powershell
   docker ps -a
   ```
   > **注意**：该 `-a` 开关会列出所有容器，包括当前未运行的容器。

1. 创建新的 helloworld 映像，其中包含已运行的第一个容器中的更改。 为此，请运行 docker commit 命令，将 \<containerID\> 替换为容器的 ID：

   ```powershell
   docker commit <containerID> helloworld
   ```

1. 现在，你有一个包含 Hello.txt 文件的自定义映像。 可以使用 docker images 命令来查看新映像。

   ```powershell
   docker images
   ```

1. 使用带有 --rm 选项的 docker run 命令运行新容器。 使用此选项时，Docker 会在命令（在本例中为 cmd.exe）停止时自动删除容器。

   ```powershell
   docker run --rm helloworld cmd.exe /s /c type C:\Users\Public\Hello.txt
   ```
   > **注意**：此命令行输出之前创建的文件的内容，并再次停止容器。

1. 输入以下命令，然后按 Enter 启动原始映像的新容器实例，并检查创建的文件是否存在：

   ```powershell
   docker run --rm mcr.microsoft.com/windows/nanoserver:ltsc2022 cmd.exe /s /c type C:\Users\Public\Hello.txt
   ```
   > **注意**：原始映像并未因添加文件而修改，并且在停止后恢复到了其原始状态。


#### 任务 3：使用 Windows Admin Center 管理容器

1. 在 SEA-ADM1 上的 Windows Admin Center 中，前往左上角的设置图标，然后选择“扩展”********。

1. 在“扩展”窗格中，在“已安装扩展”下确认“容器”扩展已安装并更新************。 如果该扩展未安装，请从“可用扩展”窗格添加****。

1. 在 SEA-ADM1 上，在 Windows Admin Center 中 sea-svr1.contoso.com 的“工具”菜单中，选择“容器”  。 系统提示关闭 PowerShell 会话时，选择“继续” 。

1. 在“容器”窗格中，浏览“概述”、“容器”、“映像”、“网络”和“卷”选项卡    。

### 练习 2 结果

完成本练习后，你应该已在 Windows Server 上安装了 Docker，下载了包含 Web 服务的 Windows 容器映像，并验证了其功能。
