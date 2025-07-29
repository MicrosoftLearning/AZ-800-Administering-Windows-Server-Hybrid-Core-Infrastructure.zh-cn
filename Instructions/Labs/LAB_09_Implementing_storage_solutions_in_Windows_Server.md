---
lab:
  title: 实验室：在 Windows Server 中实现存储解决方案
  module: 'Module 9: File servers and storage management in Windows Server'
---

# 实验室：在 Windows Server 中实现存储解决方案

## 场景

在 Contoso, Ltd，需要在 Windows Server 服务器上实现存储空间功能，以简化存储访问并提供存储级别的冗余。 管理方希望你能测试重复数据删除以节省存储。 他们还希望你能实现 Internet 小型计算机系统接口 (iSCSI) 存储，为在组织中部署存储提供更简单的解决方案。 此外，该组织正在探索能使存储高度可用的选择，且在研究为了实现高可用性而必须满足的要求。 你需要测试使用高度可用存储（尤其是存储空间直通）的可行性。

## 目标

完成本实验室后，你将能够：

- 实现重复数据删除。
- 配置 iSCSI 存储。
- 配置存储空间。
- 实现存储空间直通。

## 估计时间：90 分钟

## 实验室设置

虚拟机：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1、AZ-800T00A-SEA-SVR2、AZ-800T00A-SEA-SVR3 和 AZ-800T00A-ADM1 必须处于运行状态    。 

- 用于练习 1-3：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR3 和 AZ-800T00A-SEA-ADM1
- 用于练习 4：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1、AZ-800T00A-SEA-SVR2、AZ-800T00A-SEA-SVR3 和 AZ-800T00A-SEA-ADM1

> 注意：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1、AZ-800T00A-SEA-SVR2、AZ-800T00A-SEA-SVR3 和 AZ-800T00A-SEA-ADM1 分别托管 SEA-DC1、SEA-SVR1、SEA-SVR2、SEA-SVR3 和 SEA-ADM1 的安装           。

1. 选择“SEA-ADM1”。
1. 使用讲师提供的凭据进行登录。

对于本实验室，你将使用可用的 VM 环境。

## 实验室练习 1：实现重复数据删除

### 场景

你决定使用服务器管理器安装重复数据删除角色服务。 你判定驱动器 M 被过度使用，且怀疑其中的某些文件夹包含重复的文件。 你决定启用并配置重复数据删除角色，以减少此卷上的已占用空间。

此练习的主要任务如下：

1. 在 SEA-SVR3 上安装重复数据删除功能。
1. 在 SEA-SVR3 上的驱动器 M 上启用和配置重复数据删除 。
1. 通过添加文件和观察重复数据删除情况来测试重复数据删除。

#### 任务 1：安装重复数据删除角色服务

1. 在 SEA-ADM1 上，使用服务器管理器在 SEA-SVR3 上安装重复数据删除角色服务  。
1. 在 SEA-ADM1 上，使用授予“用户”组的读取权限共享 C:\Labfiles 文件夹  。
1. 切换到 **SEA-SVR3** 控制台会话，然后根据需要使用讲师提供的凭据登录。
1. 在 SEA-SVR3 上，启动 Windows PowerShell 会话，然后在 Windows PowerShell 控制台中运行以下命令，以创建通过 ReFS 格式化的卷，并向其分配驱动器号“M”   ：

   ```powershell
   Get-Disk
   Initialize-Disk -Number 1
   New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter M
   Format-Volume -DriveLetter M -FileSystem ReFS
   ```
1. 在 SEA-SVR3 上，在 Windows PowerShell 控制台中运行以下命令，然后从 SEA-ADM1 复制一个可创建要删除重复数据的示例文件的脚本，执行该脚本，然后识别结果  ：

   ```powershell
   New-PSDrive –Name 'X' –PSProvider FileSystem –Root '\\SEA-ADM1\Labfiles'
   New-Item -Type Directory -Path 'M:\Data' -Force
   Copy-Item -Path X:\Lab09\CreateLabFiles.cmd -Destination M:\Data\ -PassThru
   Start-Process -FilePath M:\Data\CreateLabFiles.cmd -PassThru
   Set-Location -Path M:\Data
   Get-ChildItem -Path .
   Get-PSDrive -Name M
   ```

   > 注意：记录驱动器 M 上的可用空间 。

#### 任务 2：启用和配置重复数据删除

1. 将控制台会话切换回 SEA-ADM1。
1. 使用“服务器管理器”中的“文件和存储服务”接口显示 SEA-SVR3 上的磁盘  。
1. 在 SEA-SVR3 上磁盘号为 1 的 M 卷上启用重复数据删除，设置如下  ：

   - 重复数据删除选项：常规用途文件服务器
   - 要删除的重复文件的存在时间高于（天）：0
   - 启用吞吐量优化：已选中

#### 任务 3：测试重复数据删除

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，运行以下命令，下载最新版本的 Windows Admin Center：
    
   ```powershell
   $parameters = @{
     Source = "https://aka.ms/WACdownload"
     Destination = ".\WindowsAdminCenter.exe"
     }
   Start-BitsTransfer @parameters
   ```
1. 运行以下命令以安装 Windows Admin Center：
    
   ```powershell
   Start-Process -FilePath '.\WindowsAdminCenter.exe' -ArgumentList '/VERYSILENT' -Wait
   ```

   > 注意：请等待安装完成。 这大约需要 2 分钟。

1. 在 SEA-ADM1 上，启动 Microsoft Edge 并连接到 Windows Admin Center 的本地实例 (`https://SEA-ADM1.contoso.com`)。 
1. 如果出现提示，请在“**Windows 安全**”对话框中输入讲师提供的凭据，然后选择“**确定**”。
1. 查看“配置 Windows Admin Center 设置和环境”弹出窗口上的所有选项卡，包括“扩展”选项卡，然后选择“完成”以关闭窗口。************
1. 在 Windows Admin Center 中，添加与 **sea-svr3.contoso.com** 的连接，并使用讲师提供的凭据与之连接。
1. 连接到 sea-svr3.contoso.com 时，使用“工具”列表中的“PowerShell”工具运行以下命令来触发重复数据删除  ：

   ```powershell
   Start-DedupJob -Volume M: -Type Optimization –Memory 50
   ```
1. 将控制台会话切换回 SEA-SVR3。
1. 在 SEA-SVR3 上的 Windows PowerShell 提示符下，运行以下命令，确定要删除重复数据的卷上的可用空间 ：

   ```powershell
   Get-PSDrive -Name M
   ```

   > 注意：将以前显示的值与当前值进行比较。 

1. 等待五到十分钟让删除重复作业完成并重复上一步。
1. 将控制台会话切换回 SEA-ADM1。
1. 在 SEA-ADM1 上，在已连接 sea-svr3.contoso.com 的 Windows Admin Center 中使用 PowerShell 工具运行以下命令，确定重复数据删除作业的状态  ：

   ```powershell
   Get-DedupStatus –Volume M: | fl
   Get-DedupVolume –Volume M: |fl
   Get-DedupMetadata –Volume M: |fl
   ```
1. 在 SEA-ADM1 上，刷新服务器管理器中的“磁盘”窗格，并显示 M 卷的属性  。
1. 在“卷(M:\\)属性”窗口中，查看“重复数据删除率”和“重复数据删除节省”的值  。

## 实验室练习 2：配置 iSCSI 存储

### 场景

Contoso 的管理人员正在探索使用 iSCSI 降低配置集中存储的成本和复杂性的选项。 若要对此进行测试，必须安装和配置 iSCSI 目标，并配置 iSCSI 发起程序以提供对目标的访问权限。

此练习的主要任务如下：

1. 在 SEA-SVR3 上安装 iSCSI 并配置目标。
1. 从 SEA-DC1（发起程序）连接和配置 iSCSI 目标。
1. 验证 iSCSI 磁盘配置。
1. 还原磁盘配置。

#### 任务 1：安装 iSCSI 并配置目标

1. 在 SEA-ADM1 上，在 Windows PowerShell 控制台中运行以下命令，建立与 SEA-SVR3 的 PowerShell 远程会话  ：

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR3
   ```
1. 运行以下命令，在 SEA-SVR3 上安装 iSCSI 目标：

   ```powershell
   Install-WindowsFeature –Name FS-iSCSITarget-Server –IncludeManagementTools
   ```
1. 运行以下命令，在磁盘 2 上创建使用 ReFS 格式化的新卷：

   ```powershell
   Initialize-Disk -Number 2
   $partition2 = New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter $partition2.DriveLetter -FileSystem ReFS
   ```
1. 运行以下命令，在磁盘 3 上创建使用 ReFS 格式化的新卷：

   ```powershell
   Initialize-Disk -Number 3
   $partition3 = New-Partition -DiskNumber 3 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter $partition3.DriveLetter -FileSystem ReFS
   ```
1. 运行以下命令，配置具有允许 iSCSI 流量的高级安全规则 Windows Defender 防火墙：

   ```powershell
   New-NetFirewallRule -DisplayName "iSCSITargetIn" -Profile "Any" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3260
   New-NetFirewallRule -DisplayName "iSCSITargetOut" -Profile "Any" -Direction Outbound -Action Allow -Protocol TCP -LocalPort 3260
   ```
1. 运行以下命令，显示分配给新创建的卷的驱动器号：

   ```powershell
   $partition2.DriveLetter
   $partition3.DriveLetter
   ```

   > 注意：该说明假定驱动器号分别为 E 和 F  。 如果驱动器号分配不同，则请考虑在本练习中按照说明进行操作。

#### 任务 2：连接并配置 iSCSI 目标

1. 在 SEA-ADM1 上，刷新服务器管理器中的“磁盘”窗格，并显示 SEA-DC1 的磁盘配置  。 请注意，它仅包含启动卷和系统卷驱动器 C。
1. 在服务器管理器中，在“文件和存储服务”中切换到 iSCSI 窗格 。 
1. 在 iSCSI 窗格中，采用以下设置创建 iSCSI 虚拟磁盘：

   - 存储位置：E:
   - 名称：iSCSIDisk1
   - 磁盘大小：5 GB，动态扩展
   - iSCSI 目标：新建
   - 目标名称：iSCSIFarm
   - 访问服务器：SEA-DC1

1. 采用下列设置创建第二个 iSCSI 虚拟磁盘：

   - 存储位置：F:
   - 名称：**iSCSIDisk2**
   - 磁盘大小：5 GB，动态扩展
   - iSCSI 目标：iSCSIFarm

1. 切换到 **SEA-DC1** 控制台会话，然后根据需要使用讲师提供的凭据登录。
1. 在 SEA-SVR3 上启动 Windows PowerShell 会话，然后在 Windows PowerShell 控制台中运行以下命令，启动 iSCSI 发起程序服务并显示 iSCSI 发起程序配置  ：

   ```powershell
   Start-Service msiscsi
   iscsicpl
   ```

   > 注意：iscsicpl 命令将打开“iSCSI 发起程序属性”窗口  。

1. 使用 iSCSI 发起程序属性接口连接到以下 iSCSI 目标：

   - 名称：SEA-SVR3
   - 目标名称：iqn.1991-05.com.microsoft:SEA-SVR3-fileserver-target

#### 任务 3：验证 iSCSI 磁盘配置 

1. 将控制台会话切换回 SEA-ADM1。
1. 在服务器管理器中，浏览到“文件和存储服务”的“磁盘”窗格并刷新其视图 。 
1. 查看 SEA-DC1 磁盘配置，并验证它是否包含两个状态为“脱机”，总线类型为“iSCSI”的 5 GB 磁盘   。
1. 切换到 SEA-DC1 控制台会话。 
1. 在 Windows PowerShell 提示符下，运行以下命令以显示磁盘配置：

   ```powershell
   Get-Disk
   ```
   > 注意：这两个磁盘均存在且正常，但处于脱机状态。 若要使用它们，需要对其进行初始化和格式化。

1. 在 SEA-DC1 上的 Windows PowerShell 提示符处运行以下命令，创建使用 ReFS 格式化且驱动器号为 E 的卷  。

   ```powershell
   Initialize-Disk -Number 1
   New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
   Format-Volume -DriveLetter E -FileSystem ReFS
   ```
1. 重复上一步，创建使用 ReFS 格式化的新驱动器，但这次使用磁盘号 2 和驱动器号 F 。
1. 将控制台会话切换回 SEA-ADM1，并且将“服务器管理器”窗口保持活动状态 。
1. 在服务器管理器中，刷新“文件和存储服务”中的“磁盘”窗格 。
1. 查看 SEA-DC1 磁盘配置，验证这两个驱动器现在是否都为联机状态 。

#### 任务 4：还原磁盘配置 

1. 将控制台会话切换回 SEA-SVR3。
1. 在 Windows PowerShell 提示符处运行以下命令，将 SEA-SVR3 上的磁盘重置为原始状态 ：

   ```powershell
   for ($num = 1;$num -le 4; $num++) {Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue}
   for ($num = 1;$num -le 4; $num++) {Set-Disk -Number $num -IsOffline $true}
   ```

   > 注意：这是准备下一次练习的必需的操作。

## 实验室练习 3：配置冗余存储空间

### 场景

为了满足高可用性的一些要求，你决定评估存储空间中的冗余选项。 此外，你想要测试将新磁盘预配到存储池的情况。

此练习的主要任务如下：

1. 创建存储池。
1. 基于三向镜像磁盘创建卷。
1. 在文件资源管理器中管理卷。
1. 断开磁盘与存储池的连接并验证卷的可用性。
1. 将磁盘添加到存储池并验证卷可用性。
1. 还原磁盘配置。

> **注意：** 在 Windows Server 中，你无法断开存储池中的磁盘的连接。 只能删除磁盘。 如果不先添加新磁盘，那么也无法删除三向镜像中的磁盘。 

#### 任务 1：创建存储池 

1. 切换到 SEA-ADM1，并在服务器管理器中刷新“文件和存储服务”的“磁盘”窗格  。
1. 将 SEA-SVR3 的磁盘 1-4 的状态设置为“联机” 。
1. 在服务器管理器中锁定 SEA-SVR3 的存储配置，创建名为 SP1 的新存储池，该存储池由三个大小为 127 GB 的磁盘组成   。

#### 任务 2：基于三向镜像磁盘创建卷 

1. 在 SEA-ADM1 上，在服务器管理器中使用新创建的存储池 SP1 创建名为“Three-Mirror”的虚拟磁盘，该磁盘采用镜像存储布局和精简预配，大小为 25 GB    。
1. 使用新预配的虚拟磁盘创建名为 TestData 的 ReFS 卷，将其大小设置为所有可用磁盘空间，并为其分配驱动器号 T 。

#### 任务 3：在文件资源管理器中管理卷 

1. 在 SEA-ADM1 上，切换到托管 SEA-SVR3 的 PowerShell 远程会话的 Windows PowerShell  。
1. 运行以下命令，使用 PowerShell 远程会话启用具有高级安全性的 Windows Defender 防火墙的所有文件和打印机共享规则：

   ```
   Enable-NetFirewallRule -Group "@FirewallAPI.dll,-28502"
   ```

1. 在 SEA-ADM1 上启动文件资源管理器，浏览到 \\\\SEA-SVR3.contoso.com\\t$ 共享  。
1. 创建名为“TestData”的文件夹，然后在文件夹中创建文档 TestDocument.txt 。

#### 任务 4：断开磁盘与存储池的连接并验证卷的可用性 

1. 在 SEA-ADM1 上，使用服务器管理器将附加到 SEA-SVR3 的剩余可用磁盘添加到存储池 SP1   。 请确保磁盘使用自动分配。
1. 使用服务器管理器删除分配到 SP1 池的前三个磁盘之中的一个 。
1. 在 SEA-ADM1 上使用文件资源管理器验证 TestDocument.txt 是否仍然可用 。 

#### 任务 5：将磁盘添加到存储池并验证卷可用性 

1. 在 SEA-ADM1 上，在服务器管理器中重新扫描 SP1 存储池  。
1. 将上一任务中删除的磁盘添加回来，并确保它使用自动分配。
1. 在 SEA-ADM1 上使用文件资源管理器验证 TestDocument.txt 是否仍然可用 。 
1. 切换回 SEA-SVR3。

#### 任务 6：还原磁盘配置 

1. 将控制台会话切换回 SEA-SVR3。
1. 在 Windows PowerShell 提示符处运行以下命令，将 SEA-SVR3 上的磁盘重置为原始状态 ：

   ```powershell
   Get-VirtualDisk -FriendlyName 'Three-Mirror' | Remove-VirtualDisk
   Get-StoragePool -FriendlyName 'SP1' | Remove-StoragePool
   for ($num = 1;$num -le 4; $num++) {Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue}
   for ($num = 1;$num -le 4; $num++) {Set-Disk -Number $num -IsOffline $true}
   ```

   > 注意：这是准备下一次练习的必需的操作。

## 实验室练习 4：实现存储空间直通

### 场景

你想要测试将本地存储用作高可用性存储是否是适合组织的解决方案。 以前，你的组织仅使用存储区域网络 (SAN) 存储 VM。 使用 Windows Server 中的功能，可以仅使用本地存储，因此你想要将存储空间直通实现为测试实现。

此练习的主要任务如下：

1. 准备安装存储空间直通。
1. 创建并验证故障转移群集。
1. 启用存储空间直通。
1. 创建存储池、虚拟磁盘和共享。
1. 验证存储空间直通功能。

#### 任务 1：准备安装存储空间直通 

1. 在 SEA-ADM1 上，在服务器管理器中，确认 SEA-SVR1、SEA-SVR2 和 SEA-SVR3 的可管理性状态为“联机 - 性能计数器未启动”，然后再继续      。
1. 在服务器管理器中，浏览到“文件和存储服务”的“磁盘”窗格 。
1. 在“磁盘”窗格中浏览至 SEA-SVR3 节点，并验证磁盘 1 到 4 是否列为“未知” 。
1. 在服务器管理器的“磁盘”窗格中，将附加到 SEA-SVR1、SEA-SVR2 和 SEA-SVR3 的所有磁盘转为联机状态   。
1. 在 SEA-ADM1 上启动 Windows PowerShell ISE 并在其脚本窗格中打开 C:\\Labfiles\\Lab09\\Implement-StorageSpacesDirect.ps1  。

   > 注意：脚本分为多个带编号的步骤。 共有 8 个步骤，每个步骤都有多个命令。 若要执行单个行，可以将光标置于该行中的任意位置，然后按 F8，或在 Windows PowerShell ISE 窗口的工具栏中选择“运行选择” 。 若要执行多个行，请完全选择所有行，然后使用 F8 或“运行选择”工具栏图标。 此练习的说明中介绍了步骤顺序。 确保每个步骤完成后再开始下一个步骤。

1. 运行步骤 1 中的第一个命令，在 SEA-SVR1、SEA-SVR2 和 SEA-SVR3 上安装文件服务器角色和故障转移群集功能  。

   > 注意：请等待安装完成。 这大约需要 2 分钟。 验证在每个命令的输出中，Success 属性是否设置为 True 。

1. 运行步骤 1 中的第二个命令以重启 SEA-SVR1、SEA-SVR2 和 SEA-SVR3  。

   > 注意：调用第二个命令重新启动服务器后，可以运行第三个命令来安装故障转移群集管理工具，而无需等待重新启动完成。

1. 运行步骤 1 中的第三个命令，在 SEA-ADM1 上安装故障转移群集管理器工具 。

   > 注意：服务器重新启动时请等待几分钟，故障转移群集管理器工具将安装在 SEA-ADM1 上  。

#### 任务 2：创建并验证故障转移群集 

1. 在 SEA-ADM1 上，启动“故障转移群集管理器”控制台 。
1. 在 SEA-ADM1 上，在 Windows PowerShell ISE 中运行步骤 2 命令以调用群集验证测试 。

   > 注意：请等待测试完成。 这大约需要 2 分钟。 验证所有测试均未失败。 忽略所有警告，因为这是意料之中的。

1. 在 Windows PowerShell ISE 中运行步骤 3 命令以创建群集。

   > 注意：请等待步骤完成。 这大约需要 2 分钟。 

1. 命令完成后，切换到故障转移群集管理器，并添加新创建的群集（名为 S2DCluster.Contoso.com） 。

#### 任务 3：启用存储空间直通

1. 在 SEA-ADM1 上，在 Windows PowerShell ISE 中运行步骤 4 命令以在新安装的群集上启用存储空间直通 。

   > 注意：请等待步骤完成。 这大约需要 1 分钟。

1. 在 Windows PowerShell ISE 中，运行步骤 5 命令以创建名为 S2DStoragePool 的存储池 。

   > 注意：请等待步骤完成。 此过程应该会在 1 分钟内完成。 在命令输出中，验证 FriendlyName 属性的值是否为 S2DStoragePool 。

1. 切换到故障转移群集管理器，验证群集是否包含名为“群集池 1”的存储池 。
1. 切换到 ，然后运行步骤 6 命令以创建虚拟磁盘。

   > 注意：请等待步骤完成。 此过程应该会在 1 分钟内完成。 

1. 切换到故障转移群集管理器，并验证群集虚拟磁盘 (CSV) 对象是否显示在“磁盘”窗格中 。

#### 任务 4：创建存储池、虚拟磁盘和共享

1. 在 SEA-ADM1 上，在 Windows PowerShell ISE 中运行步骤 7 命令，创建 S2D-SOFS 角色 。

   > 注意：请等待步骤完成。 此过程应该会在 1 分钟内完成。 

1. 切换到故障转移群集管理器，并验证 S2D-SOFS 对象是否显示“角色”窗格中 。
1. 切换到 Windows PowerShell ISE 并运行步骤 8 中的所有三个步骤，创建 VM01 共享 。 
1. 切换到故障转移群集管理器并验证 VM01 共享是否显示在“共享”窗格中 。

#### 任务 5：验证存储空间直通功能

1. 在 SEA-ADM1 上，在文件资源管理器中打开 \\\\s2d-sofs\\VM01 共享并创建名为 VMFolder 的文件夹  。
1. 在 SEA-ADM1 上，在 Windows PowerShell ISE 的控制台窗格中运行以下命令以关闭 SEA-SVR3  ：

   ```powershell
   Stop-Computer -ComputerName SEA-SVR3 -Force
   ```
1. 切换到服务器管理器，刷新“所有服务器”视图，验证是否已不可再访问 SEA-SVR3  。
1. 切换到故障转移群集管理器，并在“磁盘”节点中查看群集虚拟磁盘 (CSV) 信息  。 
1. 确认群集虚拟磁盘 (CSV) 的运行状况状态是否设置为“警告”以及“操作状态”是否设置为“降级”（“操作状态”也可能被列为“未完成”）      。
1. 在 SEA-ADM1 上，切换到显示 Windows Admin Center 的 Microsoft Edge 窗口。 
1. 在 Windows Admin Center 中，添加对 S2DCluster.Contoso.com 的连接。 

   > **注意**：请勿添加群集节点，因为 Windows Admin Center 中已有群集节点。 

1. 在 Windows Admin Center 中，在群集的仪表板窗格上，找到表示 SEA-SVR3 已不可访问的警报。
   > **注意**：如果概述页未显示警报，请滚动到左侧菜单上的“群集资源”，然后选择“服务器”来查看群集成员服务器的状态。********
1. 将控制台会话切换到 SEA-SVR3 并启动它。 
1. 验证该警报是否在几分钟后自动消失。
1. 刷新显示 Windows Admin Center 的浏览器页面，并验证所有服务器是否正常运行。

### 结果

完成此实验室后，你已经：

- 测试重复数据删除的实现。
- 安装并配置 iSCSI 存储。
- 配置冗余存储空间。
- 测试存储空间直通的实现。
