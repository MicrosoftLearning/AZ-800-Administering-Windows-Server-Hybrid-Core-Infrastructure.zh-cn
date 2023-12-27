---
lab:
  title: 实验室：在 Windows Server 中实现存储解决方案
  type: Answer Key
  module: 'Module 9: File servers and storage management in Windows Server'
---

# 实验室解答：在 Windows Server 中实现存储解决方案

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Implementing%20storage%20solutions%20in%20Windows%20Server)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

> 注意：请确保在每次练习后复原虚拟机 (VM)。 由于大多数 VM 都是 Windows Server 2019 Server Core，因此在练习中复原和重启所需的时间比尝试撤销对存储环境所做更改所需的时间更快。

## 练习 1：实施重复数据删除

#### 任务 1：安装重复数据删除角色服务

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\Administrator 的身份使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“服务器管理器”  。
1. 在“服务器管理器”中选择“管理”，然后选择“添加角色和功能” 。
1. 在“添加角色和功能向导”中，选择两次“下一步” 。
1. 在“选择目标服务器”页面的“服务器池”窗格中，选择“SEA-SVR3.Contoso.com”，然后选择“下一步”  。
1. 在“选择服务器角色”页面的“角色”窗格中，展开“文件和存储服务”项目，然后展开“文件和 iSCSI 服务”项目，选择“重复数据删除”项目，然后选择“下一步”    。
1. 在“选择功能”页上，选择“下一步”，然后在“确认安装选择”页中，选择“安装”   。
1. 在安装角色服务时，请在任务栏上选择“文件资源管理器”图标。
1. 在文件资源管理器中，浏览到 C 盘 。
1. 选择“Labfiles”目录，然后显示上下文相关菜单。 在菜单中，选择“授予访问权限”，然后在层叠菜单中，选择“特定人员...” 。
1. 在“网络访问”窗口的“键入名称，然后单击‘添加’，或单击箭头查找用户”文本框中，键入“用户”，然后单击“添加”   。
1. 在“网络访问”窗口中，选择“共享”，在出现“你的文件夹已共享”窗口后，选择“完成”   。
1. 切换回“服务器管理器”窗口，然后在“添加角色和功能向导安装成功”页上，选择“关闭”  。
1. 切换到 SEA-SVR3 控制台会话，然后根据需要以 CONTOSO\Administrator 身份使用密码 Pa55w.rd 登录  。
1. 如果显示“SConfig”菜单，请在“输入编号以选择一个选项”下，输入“15”并按 Enter 键以退出到 PowerShell 控制台会话   。
   
   > 注意：若要从 PowerShell 打开记事本，请键入“记事本”，然后按 Enter。

1. 在 Windows PowerShell 提示符下，输入以下命令，然后在每个命令后面按 Enter 键以创建使用 ReFS 格式化的新驱动器：

   ```powershell
   Get-Disk
   Initialize-Disk -Number 1
   New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter M
   Format-Volume -DriveLetter M -FileSystem ReFS
   ```
1. 在 Windows PowerShell 提示符下，输入以下命令，然后在每个命令后按 Enter 键从 SEA-ADM1 复制一个可创建要删除重复数据的示例文件的脚本，执行该脚本，然后识别结果 ：

   ```powershell
   New-PSDrive –Name 'X' –PSProvider FileSystem –Root '\\SEA-ADM1\Labfiles'
   New-Item -Type Directory -Path 'M:\Data' -Force
   Copy-Item -Path X:\Lab09\CreateLabFiles.cmd -Destination M:\Data\ -PassThru
   Start-Process -FilePath M:\Data\CreateLabFiles.cmd -PassThru
   Set-Location -Path M:\Data
   Get-ChildItem -Path .
   Get-PSDrive -Name M
   ```

   > 注意：记录 M 盘上的可用空间 。 

#### 任务 2：启用和配置重复数据删除

1. 将控制台会话切换回 SEA-ADM1，然后在控制台会话中切换到服务器管理器 。
1. 在“服务器管理器”树形窗格中，选择“文件和存储服务”，然后选择“磁盘”  。
1. 在“磁盘”窗格中，浏览到 SEA-SVR3 的磁盘列表，并选择表示在上一个任务中配置的磁盘号 1 的条目 。
1. 在“卷”窗格中，显示 M: 卷的上下文相关菜单，然后在菜单中选择“配置重复数据删除” 。
1. 在“卷(M:\\)重复数据删除设置”窗口的“重复数据删除”下拉列表中，选择“常规用途文件服务器”设置  。
1. 在“删除超过以下天数的文件中的重复数据:”文本框中，将默认值“3”替换为“0”  。
1. 选择“设置重复数据删除计划”按钮。
1. 在“SEA-SVR3 重复数据删除计划”窗口中，选择“启用吞吐量优化”，然后选择“确定”  。
1. 返回“卷(M:\\)重复数据删除设置”窗口，选择“确定” 。

#### 任务 3：测试重复数据删除

1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 注意：请等待安装完成。 这大约需要 2 分钟。

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 `https://SEA-ADM1.contoso.com`。

   >注意：如果链接不起作用，请在 SEA-ADM1 上打开文件资源管理器，选择“下载”文件夹，在“下载”文件夹中选择 WindowsAdminCenter.msi 文件并手动安装。 安装完成后，刷新 Microsoft Edge。

   >注意：如果收到 NET::ERR_CERT_DATE_INVALID 错误，请在 Microsoft Edge 浏览器页上选择“高级”，在页面底部选择“继续访问 sea-adm1-contoso.com (不安全)” 。

1. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：`CONTOSO\Administrator`
   - 密码：`Pa55w.rd`

1. 在“所有连接”窗格中，选择“+ 添加”。
1. 在“添加或创建资源”窗格上的“服务器”磁贴中，选择“添加” 。
1. 在“服务器名称”文本框中，输入“sea-svr3.contoso.com” 。 
1. 如果需要，请确保已选中“为此连接使用另一个帐户”选项，输入以下凭据，然后选择“使用凭据添加” ：

   - 用户名：`CONTOSO\Administrator`
   - 密码：`Pa55w.rd`

1. 在“sea-svr3.contoso.com”页上的“工具”菜单中，选择“PowerShell”，然后在出现提示时，以 CONTOSO\Administrator 用户身份使用密码 Pa55w.rd 登录    。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 键触发重复数据删除：

   ```powershell
   Start-DedupJob -Volume M: -Type Optimization –Memory 50
   ```
1.  将控制台会话切换回 SEA-SVR3。
1. 在 SEA-SVR3 上的 Windows PowerShell 提示符下，输入以下命令并按 Enter 键，以确定要删除重复数据的卷上的可用空间 ：

   ```powershell
   Get-PSDrive -Name M
   ```

   > 注意：将以前显示的值与当前值进行比较。 

1. 等待五到十分钟让删除重复作业完成并重复上一步。
1. 将控制台会话切换回 SEA-ADM1。
1. 在 SEA-ADM1 上，在 Microsoft Edge 窗口内的 Windows PowerShell 控制台中（其中显示 Windows Admin Center 与 sea-svr3.contoso.com 的连接），输入以下命令，并在每个命令后按 Enter 键来确定重复数据删除作业的状态   ：

   ```powershell
   Get-DedupStatus –Volume M: | fl
   Get-DedupVolume –Volume M: |fl
   Get-DedupMetadata –Volume M: |fl
   ```
1. 在 SEA-ADM1 上，切换到服务器管理器中的“磁盘”窗格，然后在右上角的“任务”菜单中选择“刷新”   。
1. 在“卷”部分中选择“M:”卷，显示其上下文相关菜单，然后从菜单中选择“属性”  。 
1. 在“卷(M:\\)属性”窗口中，查看“重复数据删除率”和“重复数据删除节省”的值  。

## 练习 2：配置 iSCSI 存储

#### 任务 1：安装 iSCSI 并配置目标

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 窗口 。
1. 在 Windows PowerShell 控制台中，输入以下命令并按下 Enter 键，建立与 SEA-SVR3 的 PowerShell 远程会话 ：

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR3
   ```
1. 输入以下命令，并按 Enter 键在 SEA-SVR3 上安装 iSCSI 目标：

   ```powershell
   Install-WindowsFeature –Name FS-iSCSITarget-Server –IncludeManagementTools
   ```
1. 输入以下命令，然后在每个命令之后按 Enter 键，以在磁盘 2 上创建使用 ReFS 格式化的新卷：

   ```powershell
   Initialize-Disk -Number 2
   $partition2 = New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter $partition2.DriveLetter -FileSystem ReFS
   ```
1. 输入以下命令，然后在每个命令之后按 Enter 键，以在磁盘 3 上创建使用 ReFS 格式化的新卷：

   ```powershell
   Initialize-Disk -Number 3
   $partition3 = New-Partition -DiskNumber 3 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter $partition3.DriveLetter -FileSystem ReFS
   ```
1. 输入以下命令，然后在每个命令后按 Enter 键来配置具有允许 iSCSI 流量的高级安全规则 Windows Defender 防火墙：

   ```powershell
   New-NetFirewallRule -DisplayName "iSCSITargetIn" -Profile "Any" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3260
   New-NetFirewallRule -DisplayName "iSCSITargetOut" -Profile "Any" -Direction Outbound -Action Allow -Protocol TCP -LocalPort 3260
   ```
1. 输入以下命令，然后按 Enter 键显示分配给新创建的卷的驱动器号：

   ```powershell
   $partition2.DriveLetter
   $partition3.DriveLetter
   ```

   > 注意：该说明假定驱动器号分别为 E 和 F  。 如果驱动器号分配不同，则请考虑在本练习中按照说明进行操作。

#### 任务 2：连接并配置 iSCSI 目标

1. 在 SEA-ADM1 上，切换到服务器管理器 。
1. 在 SEA-ADM1 上，切换到服务器管理器中的“磁盘”窗格，然后在右上角的“任务”菜单中选择“刷新”   。
1. 在服务器管理器中，在树形窗格中选择文件和存储服务的磁盘后，在右上角的“任务”菜单中，选择“刷新”    。
1. 查看更新后的 SEA-SVR3 磁盘配置，其中磁盘 2 和 3 联机。
1. 浏览到 SEA-DC1 条目并查看磁盘配置。

   > 注意：SEA-DC1 仅有一个承载启动卷和系统卷的磁盘 。

1. 在服务器管理器的“文件和存储服务”中，选择“iSCSI”，选择“任务”，然后在下拉菜单中选择“新建 iSCSI 虚拟磁盘”    。
1. 在“新建 iSCSI 虚拟磁盘向导”中，在“选择 iSCSI 虚拟磁盘位置”页面的“SEA-SVR3”服务器下，选择“E:”卷，然后选择“下一步”    。
1. 在“指定 iSCSI 虚拟磁盘名称”页面的“名称”文本框中输入“iSCSIDisk1”，然后选择“下一步”   。
1. 在“指定 iSCSI 虚拟磁盘大小”页面的“大小”文本框中，输入“5”  。 将所有其他设置保留默认值，然后选择“下一步”。
1. 在“分配 iSCSI 目标”页面，确保选择“新建 iSCSI 目标”单选按钮，然后选择“下一步”  。
1. 在“指定目标名称”页面的“名称”字段中，输入“iSCSIFarm”，然后选择“下一步”   。
1. 在“指定访问服务器”页面中，选择“添加”按钮 。
1. 在“选择用于标识发起程序的方法”窗口中，选择“浏览”按钮 。
1. 在“选择计算机”窗口的“输入要选择的对象名称”文本框中，输入“SEA-DC1”，选择“检查名称”，然后选择“确定”    。
1. 在“选择用于标识发起程序的方法”窗口中，选择“确定” 。
1. 在“指定访问服务器”页面上，选择“下一步” 。
1. 在“启用身份验证”页面上，选择“下一步” 。
1. 在“确认选择”页面上，选择“创建” 。
1. 在“查看结果”页面上，选择“关闭” 。
1. 创建第二个 iSCSI 虚拟磁盘 (F:)，方法是重复步骤 6 到 9，选择现有的 iSCSI 目标，并使用以下设置通过步骤 18 到 19 完成向导：

   - 存储位置：F:
   - 名称：iSCSIDisk2
   - 磁盘大小：5 GB，动态扩展
   - iSCSI 目标：iSCSIFarm

1. 切换到 SEA-DC1 控制台会话，然后根据需要以 CONTOSO\Administrator 身份使用密码 Pa55w.rd 登录  。
1. 如果显示“SConfig”菜单，请在“输入编号以选择一个选项”下，输入“15”并按 Enter 键以退出到 PowerShell 控制台会话   。
1. 在 Windows PowerShell 提示符下，输入以下命令，然后在每个命令后按 Enter 键，启动 iSCSI 发起程序服务并显示 iSCSI 发起程序配置：

   ```powershell
   Start-Service -Name MSiSCSI
   iscsicpl
   ```

   > 注意：iscsicpl 命令将打开“iSCSI 发起程序属性”窗口  。

1. 在 SEA-DC1 上，在“iSCSI 发起程序属性”对话框的“目标”标签上的“目标”文本框中，键入“SEA-SVR3.contoso.com”，然后选择“快速连接”     。
1. 在“快速连接”对话框中，注意“发现的目标名称”是“iqn.1991-05.com.microsoft:sea-svr3-iscscifarm-target”，然后选择“完成”   。
1. 在“iSCSI 发起程序属性”对话框中，选择“确定” 。

#### 任务 3：验证 iSCSI 磁盘配置

1. 将控制台会话切换回 SEA-ADM1，并且将“服务器管理器”窗口保持活动状态 。
1. 在服务器管理器中，从“文件和存储服务”中的“iSCSI”窗格切换到“磁盘”窗格，然后在右上角的“任务”菜单中选择“刷新”   。 
1. 查看 SEA-DC1 磁盘配置，并验证它是否包含两个状态为“脱机”，总线类型为“iSCSI”的 5 GB 磁盘   。
1. 切换到 SEA-DC1 控制台会话。 
1. 在 Windows PowerShell 提示符下，输入以下命令，然后按 Enter 键列出磁盘配置：

   ```powershell
   Get-Disk
   ```

   > 注意：这两个磁盘均存在且正常，但处于脱机状态。 若要使用它们，需要对其进行初始化和格式化。

1. 输入以下命令，然后在每个命令后按 Enter 键以创建使用 ReFS 格式化且驱动器号为 E 的卷。

   ```powershell
   Initialize-Disk -Number 1
   New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
   Format-Volume -DriveLetter E -FileSystem ReFS
   ```
1. 重复上一步，创建使用 ReFS 格式化的新驱动器，但这次使用磁盘号 2 和驱动器号 F 。
1. 将控制台会话切换回 SEA-ADM1，并且将“服务器管理器”窗口保持活动状态 。
1. 在服务器管理器中，通过在窗口右上角的“任务”菜单中选择“刷新”，刷新“文件和存储服务”中的“磁盘”窗格   。 
1. 查看 SEA-DC1 磁盘配置，验证这两个驱动器现在是否都为联机状态 。

#### 任务 4：还原磁盘配置 

1. 将控制台会话切换回 SEA-SVR3。
1. 在 Windows PowerShell 提示符下，输入以下命令，并在每个命令后按 Enter 键将 SEA-SVR3 上的磁盘重置为原始状态（当系统提示确认 Clear-Disk cmdlet 的执行时，按 Y 键，然后在每个循环迭代之前按 Enter 键）   ：

   ```powershell
   for ($num = 1;$num -le 4; $num++) {Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue}
   for ($num = 1;$num -le 4; $num++) {Set-Disk -Number $num -IsOffline $true}
   ```

   > 注意：这是准备下一次练习的必需的操作。

## 练习 3：配置冗余存储空间

#### 任务 1：创建存储池

1. 将控制台会话切换回 SEA-ADM1，并且将“服务器管理器”窗口保持活动状态 。
1. 在服务器管理器中，通过在窗口右上角的“任务”菜单中选择“刷新”，刷新“文件和存储服务”中的“磁盘”窗格   。 
1. 在“磁盘”窗格中向下滚动，注意列出的 SEA-SVR3 磁盘 1 到 4 具有“未知”分区和“脱机”状态  。 
1. 依次选择四个磁盘中的每一个，显示其上下文相关菜单，选择菜单中的“联机”选项，然后在“磁盘联机”窗口中，选择“是”  。
1. 验证所有列出的磁盘是否都为“联机”状态。 在“服务器管理器”窗格中，选择“存储池” 。
1. 在服务器管理器的“存储池”区域中的“任务”列表中，选择“新建存储池”   。 
1. 在“新建存储池向导”中，在“准备工作”页上，单击“下一步”  。
1. 在“指定存储池名称和子系统”页上的“名称”文本框中，输入“SP1”  。 在“说明”文本框中，输入“存储池 1” 。 在“选择你要使用的可用磁盘组(也称为原始池)”列表中，选择“SEA-SVR3”条目，然后选择“下一步”  。
1. 在“选择存储池的物理磁盘”页上，选中大小为 127 GB 的三个磁盘旁边的复选框，然后选择“下一步”  。
1. 在“确认选择”页上，查看设置，然后选择“创建” 。
1. 选择“关闭”。

#### 任务 2：基于三向镜像磁盘创建卷 

1. 在 SEA-ADM1 上，在服务器管理器的“存储池”窗格中，选择“SP1”  。
1. 在“虚拟磁盘”区域中，选择“任务”，然后选择“新建虚拟磁盘”  。
1. 在“选择存储池”对话框中，选择“SP1”，然后选择“确定”  。
1. 在“新建虚拟磁盘向导”中，在“准备工作”页上，单击“下一步”  。
1. 在“指定虚拟磁盘名称”页面的“名称”文本框中输入“Three-Mirror”，然后选择“下一步”   。
1. 在“指定机箱复原”页上，选择“下一步” 。
1. 在“选择存储布局”页上，选择“镜像”，然后选择“下一步”  。
1. 在“指定预配类型”页上，选择“精简”，然后选择“下一步”  。
1. 在“指定虚拟磁盘大小”页上的“指定大小”文本框中，输入“25”，然后选择“下一步”   。
1. 在“确认选择”页上，查看设置，然后选择“创建” 。
1. 在“查看结果”页上，清除“在此向导关闭时创建卷”复选框，然后选择“关闭”  。
1. 在服务器管理器的导航窗格中，确保选中“卷”条目 。
1. 在“卷”区域中，选择“任务”，然后选择“新建卷”  。
1. 在“新建卷向导”中，在“准备工作”页上，单击“下一步”  。
1. 在“选择服务器和磁盘”页上，选择“SEA-SVR3”，选择“Three-Mirror”，然后选择“下一步”   。
1. 在“指定卷的大小”页上，选择“下一步” 。
1. 在“分配到驱动器号或文件夹”页上，选择“驱动器号”，选择“T”，然后选择“下一步”   。
1. 在“选择文件系统设置”页的“文件系统”下拉列表中，选择“ReFS”  。 在“卷标签”文本框中，输入“TestData”，然后选择“下一步”  。
1. 在“启用重复数据删除”页上，选择“下一步” 。
1. 在“确认选择”页面上，选择“创建” 。
1. 在“完成”页中，选择“关闭” 。

#### 任务 3：在文件资源管理器中管理卷

1. 在 SEA-ADM1 上，切换到托管 SEA-SVR3 的 PowerShell 远程会话的 Windows PowerShell  。
1. 在 Windows PowerShell 控制台的 [SEA-SVR3] 提示符下，输入以下命令，然后按 Enter 键以启用具有高级安全性的 Windows Defender 防火墙的所有文件和打印机共享规则 ：

   ```powershell
   Enable-NetFirewallRule -Group "@FirewallAPI.dll,-28502"
   ```

1. 在 SEA-ADM1 的任务栏中，选择“文件资源管理器”图标 。
1. 在“文件资源管理器”窗口的“地址栏”中，输入 \\\\SEA-SVR3.contoso.com\\t$  。
1. 在文件资源管理器的“详细信息”窗格中，显示上下文相关菜单，然后在菜单中选择“新建文件夹”。 将分配给新文件夹的默认名称替换为 TestData，然后按 Enter 键。
1. 在文件资源管理器中，双击新创建的 TestData 文件夹。
1. 在文件资源管理器的“详细信息”窗格中，显示上下文相关的菜单，然后在菜单中选择“新建”，然后选择“文本文档” 。 将分配给新文件的默认名称替换为 TestDocument，然后按 Enter 键。

#### 任务 4：断开磁盘与存储池的连接并验证卷的可用性 

1. 在 SEA-ADM1 上，切换到服务器管理器 。 在“文件和服务存储”树形窗格中，选择“存储池”，然后选择“SP1”  。
1. 在“物理磁盘”窗格中，选择“任务”下拉列表，然后选择“添加物理磁盘” 。
1. 在“添加物理磁盘”对话框，在表示要添加到池中的第四个磁盘的行中，选中磁盘名称旁边的复选框。 在“分配”下拉列表中，确保选中“自动”条目，然后选择“确定”  。
1. 在“物理磁盘”窗格中，右键单击列表中的顶部磁盘，然后选择“删除磁盘”。
1. 在“删除物理磁盘”窗口中，选择“是” 。
1. 在“删除物理磁盘”对话框中，查看说明“Windows 正在修复受影响的虚拟磁盘”的消息，然后选择“确定”  。
1. 切换回“文件资源管理器”窗口，其中显示 TestData 文件夹的内容 。
1. 打开 TestDocument.txt 并验证其内容是否仍然可用。

#### 任务 5：将磁盘添加到存储池并验证卷可用性 

1. 在 SEA-ADM1 上，切换到服务器管理器 。 在“文件和存储服务”树形窗格中，选择“存储池”条目后，在右上角的“任务”菜单中，选择“重新扫描存储”   。
1. 出现提示时，在“重新扫描存储”对话框中，选择“是” 。
1. 在“物理磁盘”窗格中，选择“任务”，然后在下拉菜单中选择“添加物理磁盘” 。
1. 在“添加物理磁盘”窗口，在表示要添加到池中的第四个磁盘的行中，选中磁盘名称旁边的复选框。 在“分配”下拉列表中，确保选中“自动”条目，然后选择“确定”  。
1. 在 SEA-ADM1 上，切换到“文件资源管理器”窗口，并验证 TestData 文件夹及其内容是否仍然可用  。

#### 任务 6：还原磁盘配置 

1. 将控制台会话切换回 SEA-SVR3。
1. 在 Windows PowerShell 提示符下，输入以下命令，并在每个命令后按 Enter 键将 SEA-SVR3 上的磁盘重置为原始状态（当系统提示确认 Remove-VirtualDisk、Remove-StoragePool 和 Clear-Disk cmdlet 的执行时，按 Y 键，然后在每个 cmdlet 执行之前按 Enter 键）     ：

   ```powershell
   Get-VirtualDisk -FriendlyName 'Three-Mirror' | Remove-VirtualDisk
   Get-StoragePool -FriendlyName 'SP1' | Remove-StoragePool
   for ($num = 1;$num -le 4; $num++) {Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue}
   for ($num = 1;$num -le 4; $num++) {Set-Disk -Number $num -IsOffline $true}
   ```

   > 注意：这是准备下一次练习的必需的操作。

## 练习 4：实施存储空间直通

#### 任务 1：准备安装存储空间直通 

1. 将控制会话切换回 SEA-ADM1，然后选择“所有服务器” 。
1. 在 SEA-ADM1 上，在服务器管理器的控制台树中，选择“所有服务器”，并确认 SEA-SVR1、SEA-SVR2 和 SEA-SVR3 的可管理性状态为“联机 - 性能计数器未启动”，然后再继续       。
1. 在服务器管理器的导航窗格中，选择“文件和存储服务”，然后选择“磁盘”  。
1. 选择“磁盘”窗格后，在其右上角的“任务”菜单中，选择“刷新” 。
1. 在“磁盘”窗格中，向下滚动到 SEA-SVR3 的磁盘 1 至 4 的列表，并验证它们各自在“分区”列中的条目是否被列为“未知”  。
1. 依次选择四个磁盘中的每一个，然后显示其上下文相关菜单。 在菜单中，选择“联机”选项，然后在“磁盘联机”窗口中选择“是”  。
1. 使用相同的方法，将 SEA-SVR1 和 SEA-SVR2 的所有磁盘均联机 。
1. 在 SEA-ADM1 上，选择“开始”，然后在“开始”菜单中选择“Windows PowerShell ISE”   。
1. 在 Windows PowerShell ISE 中，选择“文件”菜单 。 在“文件”菜单中，选择“打开”，然后在“打开”对话框中，选择“C:\Labfiles\Lab09”   。
1. 选择“Implement-StorageSpacesDirect.ps1”，然后选择“打开” 。

   > 注意：脚本分为多个带编号的步骤。 共有 8 个步骤，每个步骤都有多个命令。 若要执行单个行，可以将光标置于该行中的任意位置，然后按 F8，或在 Windows PowerShell ISE 窗口的工具栏中选择“运行选择” 。 若要执行多个行，请完全选择所有行，然后使用 F8 或“运行选择”工具栏图标。 此练习的说明中介绍了步骤顺序。 确保每个步骤完成后再开始下一个步骤。

1. 选择步骤 1 中的第一行，然后按 F8 在 SEA-SVR1、SEA-SVR2 和 SEA-SVR3 上安装“文件服务器角色和故障转移群集”功能   。

   > 注意：请等待安装完成。 这大约需要 2 分钟。 验证在每个命令的输出中，Success 属性是否设置为 True 。

1. 选择步骤 1 中的第二行，然后按 F8 重新启动 SEA-SVR1、SEA-SVR2 和 SEA-SVR3  。

   > 注意：调用第二个命令重新启动服务器后，可以运行第三个命令来安装故障转移群集管理工具，而无需等待重新启动完成。

1. 选择步骤 1 中以“Install”开头的第三行，然后按 F8 键在 SEA-ADM1 上安装故障转移群集管理器工具  。

   > 注意：服务器重新启动时请等待几分钟，故障转移群集管理器工具将安装在 SEA-ADM1 上  。

#### 任务 2：创建并验证故障转移群集 

1. 在 SEA-ADM1 上，切换到“服务器管理器”窗口 。
1. 在服务器管理器中，选择“工具”，然后选择“故障转移群集管理器”以验证其安装是否已成功完成  。
1. 在 SEA-ADM1 上，切换到“Administrator: Windows PowerShell ISE”窗口，选择步骤 2 中以“Test-Cluster”开头的行，然后按 F8 键调用群集验证测试  。

   > 注意：请等待测试完成。 这大约需要 2 分钟。 验证所有测试均未失败。 忽略所有警告，因为这是意料之中的。 

1. 在“Administrator: Windows PowerShell ISE”窗口中，选择步骤 3 中以“New-Cluster”开头的行，然后按 F8 键创建群集 。

   > 注意：请等待步骤完成。 这大约需要 2 分钟。 

1. 在 SEA-ADM1 上，切换到“故障转移群集管理器”窗口 。 在“操作”窗格中，选择“连接到群集”，输入 S2DCluster.Contoso.com，然后选择“确定”  。

#### 任务 3：启用存储空间直通

1. 在 SEA-ADM1 上，切换到“Administrator: Windows PowerShell ISE”窗口，选择步骤 4 中以“Invoke-Command”开头的行，然后按 F8 键在新安装的群集上启用存储空间直通  。

   > 注意：请等待步骤完成。 这大约需要 1 分钟。

1. 在“Administrator: Windows PowerShell ISE”窗口中，选择步骤 5 中以“Invoke-Command”开头的行，然后按 F8 键创建 S2DStoragePool  。

   > 注意：请等待步骤完成。 此过程应该会在 1 分钟内完成。 在命令输出中，验证 FriendlyName 属性的值是否为 S2DStoragePool 。

1. 切换到“故障转移群集管理器”窗口，展开“S2DCluster.Contoso.com”，展开“存储”，然后选择“池”   。
1. 验证群集池 1 是否存在。
1. 在“Administrator: Windows PowerShell ISE”窗口中，选择步骤 6 中以“Invoke-Command”开头的行，然后按 F8 键创建虚拟机 。

   > 注意：请等待步骤完成。 此过程应该会在 1 分钟内完成。 

1. 切换到“故障转移群集管理器”窗口，然后在“存储”节点中，选择“磁盘”  。
1. 验证群集虚拟磁盘 (CSV) 是否存在。

#### 任务 4：创建存储池、虚拟磁盘和共享

1. 在“Administrator: Windows PowerShell ISE”窗口中，选择步骤 7 中以“Invoke-Command”开头的行，然后按 F8 键创建文件服务器群集角色 。

   > 注意：请等待步骤完成。 此过程应该会在 1 分钟内完成。 

1. 验证该命令的输出是否包含角色定义，并将属性 FriendlyName 设置为 S2D-SOFS 。 这会验证命令是否成功。
1. 切换到“故障转移群集管理器”窗口并选择“角色” 。
1. 验证是否存在 S2D-SOFS 角色。 这还会验证命令是否已成功完成。
1. 在“Administrator: Windows PowerShell ISE”窗口中，选择步骤 8 中以“Invoke-Command”开头的三行，然后按 F8 键创建文件共享 。

   > 注意：请等待步骤完成。 此过程应该会在 1 分钟内完成。 

1. 验证该命令的输出是否包含文件共享的定义，并且 Path 属性设置为 C:\\ClusterStorage\\CSV\\VM01 。 这会验证命令是否成功完成。
1. 在“故障转移群集管理器”窗口中的“角色”窗格中，选择“名称”列下的“S2D-SOFS”，然后选择“共享”选项卡。
1. 验证名为 VM01 的共享是否存在。 这还会验证命令是否已成功完成。

#### 任务 5：验证存储空间直通功能

1. 在 SEA-ADM1 的任务栏中，选择“文件资源管理器”图标 。
1. 在文件资源管理器的地址栏中，输入 \\\\S2D-SOFS.contoso.com\\VM01，然后按 Enter 键打开目标文件共享 。
1. 在文件资源管理器的“详细信息”窗格中，显示上下文相关菜单，然后在菜单中选择“新建文件夹” 。 将分配给新文件夹的默认名称替换为 VMFolder，然后按 Enter 键。
1. 在 SEA-ADM1 上，切换到“Administrator: Windows PowerShell ISE”窗口 。
1. 在“Administrator: Windows PowerShell ISE”窗口的控制台窗格中，输入以下命令，然后按 Enter 键关闭 SEA-SVR3 ：

   ```powershell
   Stop-Computer -ComputerName SEA-SVR3 -Force
   ```

1. 在 SEA-ADM1 上，切换到“服务器管理器”窗口，选择“所有服务器”，然后在右上角的“任务”菜单中选择“刷新”    。
1. 验证“SEA-SVR3”条目在“可管理性”列中是否有“目标计算机不可访问”条目  。
1. 切换回“文件资源管理器”窗口，并验证 VMFolder 是否仍可访问 。
1. 切换到故障转移群集管理器，选择“磁盘”，然后选择“群集虚拟磁盘(CSV)”  。
1. 确认群集虚拟磁盘 (CSV) 的运行状况状态是否设置为“警告”以及“操作状态”是否设置为“降级”（“操作状态”也可能被列为“未完成”）      。
1. 在 SEA-ADM1 上，切换到显示 Windows Admin Center 的 Microsoft Edge 窗口 。 
1. 浏览到“所有连接”窗格，然后选择“+ 添加”。
1. 在“添加或创建资源”窗格的“服务器群集”窗格中，选择“添加”  。
1. 在“群集名称”文本框中，输入“S2DCluster.Contoso.com” 。
1. 确保已选中“为此连接使用另一个帐户”选项，输入以下凭据，然后选择“使用帐户连接” ：

   - 用户名：`CONTOSO\Administrator`
   - 密码：`Pa55w.rd`
   
1. 清除“同时在群集中添加服务器”选项，然后选择“添加” 。
1. 返回到“所有连接”页，选择“s2dcluster.contoso.com” 。
1. 验证在加载页面时，“仪表板”窗格中是否有一个警报，指出无法访问 SEA-SVR3。 
1. 将控制台会话切换到 SEA-SVR3 并启动它。 

   > 注意：自动删除警报可能需要几分钟时间。

1. 刷新显示 Windows Admin Center 的浏览器页面，并验证所有服务器是否正常运行。
