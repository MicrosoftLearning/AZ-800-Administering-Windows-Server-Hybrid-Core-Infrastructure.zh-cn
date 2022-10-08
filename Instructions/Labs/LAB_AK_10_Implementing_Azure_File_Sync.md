---
lab:
  title: 实验室：实现 Azure 文件同步
  type: Answer Key
  module: 'Module 10: Implementing a hybrid file server infrastructure'
---

# <a name="lab-answer-key-implementing-azure-file-sync"></a>实验室答案密钥：实施 Azure 文件同步

## <a name="exercise-1-implementing-distributed-file-system-dfs-replication-in-your-on-premises-environment"></a>练习 1：在本地环境中实现分布式文件系统 (DFS) 复制

### <a name="task-1-deploy-dfs"></a>任务 1：部署 DFS

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\\Administrator 的身份使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，在“开始”菜单中选择 Windows PowerShell  。
1. 在 Windows PowerShell 控制台中，输入以下内容，然后按 Enter 键安装分布式文件系统 (DFS) 管理工具：

   ```powershell
   Install-WindowsFeature -Name RSAT-DFS-Mgmt-Con -IncludeManagementTools
   ```
1. 在任务栏上，选择“文件资源管理器”。
1. 在文件资源管理器中，浏览到 C:\\Labfiles\\Lab10 文件夹。
1. 在文件资源管理器的“详细信息”窗格中，选择“L10_DeployDFS.ps1”文件，显示其上下文相关菜单，然后在菜单中选择“编辑” 。

   >注意：这会自动在 Windows PowerShell ISE 的脚本窗格中打开 L10_DeployDFS.ps1 文件 。

1. 在 Windows PowerShell ISE 脚本窗格中，查看脚本，然后通过选择工具栏中的“运行脚本”图标或按 F5 来执行该脚本 。 

### <a name="task-2-test-dfs-deployment"></a>任务 2：测试 DFS 部署

1. 在 SEA-ADM1 上，选择“开始”，输入“DFS”，然后选择“DFS管理”   。
1. 在“DFS 管理”的导航窗格中，右键单击或访问“名称空间”的上下文菜单，然后选择“添加要显示的名称空间”  。
1. 在“添加要显示的命名空间”对话框中，在命名空间列表中，选择“\\\contoso.com\Root”，然后选择“确定”  。
1. 在导航窗格中，右键单击或访问“复制”的上下文菜单，然后选择“添加要显示的复制组” 。
1. 在“添加要显示的复制组”对话框中，在“复制组”部分，选择“分支 1”，然后选择“确定”   。
1. 在导航窗格中，展开“\\\contoso.com\Root”命名空间，然后选择“数据”文件夹 。
1. 在详细信息窗格中，验证“数据”文件夹是否有两个分别对 SEA-SVR1 和 SEA-SVR2 上的“数据”文件夹的引用   。
1. 在导航窗格中，选择“分支 1”。
1. 在详细信息窗格中，验证 SEA-SVR1 和 SEA-SVR2 上的“S:\\Data”文件夹是否是“分支 1”复制组的成员   。

   >注意：“DFS 复制”在 SEA-SVR1 和 SEA-SVR2 上的“S:\\Data”文件夹之间复制内容   。

1. 打开两个文件资源管理器实例。 在第一个文件资源管理器实例中连接到“\\\\SEA-SVR1\\Data”，然后在第二个文件资源管理器实例中，连接到“\\\\SEA-SVR2\\Data” 。
1. 在“\\\\SEA-SVR1\\Data”中创建一个用你名字命名的新文件。
1. 验证使用你名字的文件是否在几秒钟后复制到“\\\\SEA-SVR2\\Data”。 这样可以确认“DFS 复制”是否在正常工作。

   >注意：等待文件复制完毕，两个文件资源管理器窗口中会记录相同的内容。

## <a name="exercise-2-creating-and-configuring-a-sync-group"></a>练习 2：创建和配置同步组

### <a name="task-1-create-an-azure-file-share"></a>任务 1：创建 Azure 文件共享

1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录 。
1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“存储帐户” 。
1. 在“存储帐户”页上，选择“+ 创建” 。
1. 在“创建存储帐户”页的“基本信息”选项卡上，指定以下设置 ：

   - 资源组：选择“新建”，输入“AZ800-L1001-RG”作为资源组名称，然后选择“确定”  。
   - 存储帐户名称：键入一个字符串，该字符串以小写字母开头，后跟小写字母和数字的任意组合，总长度介于 3 到 24 个字符之间。 

   >注意：选择具有全局唯一性的名称。 例如，可以使用以下格式指定存储帐户名：“*DDMMYY”；例如，如果你的名字是 Devon Torres，并且你要在 2022 年 1月 30 日创建一个存储帐户，那么该存储帐户的名称将是 dt013022\<*yourlowercaseinitials*>* 。 如果该名称已被占用，请为该名称再添加一个字符，直到该名称可用为止。 

   - 区域：可创建存储帐户的你所在地理区域中的任何 Azure 区域。

   >注意：请使用同一区域来部署此实验中的所有资源。

   - 冗余：本地冗余存储 (LRS)

1. 接受所有其他设置的默认值，选择“查看 + 创建”，然后选择“创建” 。
1. 创建存储帐户后，在“部署”页面上，选择“转到资源” 。
1. 在“存储帐户”页面上，选择“文件共享”，然后选择“+ 文件共享”  。
1. 在“新文件共享”选项卡上，在“名称”文本框中输入“share1”，然后选择“创建”   。

### <a name="task-2-use-an-azure-file-share"></a>任务 2：使用 Azure 文件共享

1. 在 SEA-ADM1 上，在 Azure 门户的详细信息窗格中，选择“share1” 。
1. 在详细信息窗格中，选择“上传”。
1. 在“上传文件”选项卡上，浏览到“**C:\\Labfiles\\Lab10\\File1.txt**”，选择上传，上传完成后，关闭“上传文件”选项卡  。
1. 在“share1”页面上，选择“快照”，选择“添加快照”，然后选择“确定”   。
1. 在“share1”页面上，选择“概览”，选择“连接”，使用“复制到剪贴板”按钮复制脚本，然后关闭“连接”选项卡    。
1. 在 SEA-ADM1 上，切换到“Windows PowerShell ISE”窗口，在脚本窗格中打开另一个选项卡，并将复制的脚本粘贴到其中 。
1. 查看脚本的内容，然后通过选择工具栏中的“运行脚本”图标或按 F5 执行脚本。 

   >注意：脚本已将 Azure 文件共享装载到 Z 驱动器 。

1. 在任务栏上，右键单击或访问文件资源管理器的上下文菜单，选择“文件资源管理器”，然后在“快速访问”文本框中，键入“*Z:\*”，然后按 Enter 键 。
1. 验证“详细信息”窗格中显示文件 File1.txt。 这是上传到 Azure 文件共享的文件。
1. 双击或选择“File1.txt”，然后按 Enter 在记事本中打开文件。 
1. 使用记事本，在最后一行中追加自己的姓名来修改文件内容，保存更改并关闭记事本。
1. 右键单击或访问“File1”的上下文菜单，选择“属性”，然后在“File1 属性”窗口中，选择“以前的版本”选项卡   。
1. 验证以前的一个文件版本是否可用。 选择该版本 (File1.txt)，选择“还原”两次，然后选择“确定”两次  。
1. 双击或选择“File1.txt”，选择 Enter，然后确认它不包含你的名字。 这是因为你还原了在修改该文件之前创建的快照。
1. 关闭“记事本”。

### <a name="task-3-deploy-storage-sync-service-and-a-file-sync-group"></a>任务 3：部署存储同步服务和文件同步组

1. 在 SEA-ADM1 上，在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“Azure 文件同步”  。
1. 在“部署文件同步”页面的“基本信息”选项卡上，在“资源组”下拉列表中，选择“AZ800-L1001-RG”   。 
1. 在“存储同步服务名称”文本框中，输入“FileSync1” 。 
1. 在“区域”下拉列表中，选择创建了存储帐户的同一区域。 
1. 在“部署文件同步”页面的“基本信息”选项卡上，依次选择“审阅 + 创建”和“创建”   。
1. 在“部署”边栏选项卡上，预配好文件同步后，选择“转到资源” 。
1. 在“FileSync1”的“存储同步服务”页面上，选择“同步组”，然后选择“+ 同步组”以创建新的文件同步组   。
1. 在“同步组”页面上，在“同步组名称”文本框中输入“Sync1”  。
1. 选择“选择存储帐户”，然后在“选择存储帐户”页面上选择创建的存储帐户 。 

   >注意：如果找不到存储帐户，可能是之前已将其部署到其他 Azure 区域。 需要确保该存储帐户与存储同步服务位于同一区域中。

1. 在“Azure 文件共享”下拉列表中，选择“share1”，然后选择“创建”  。
1. 在“存储同步服务”页面上，选择“注册的服务器”，并验证当前没有注册的服务器 。

## <a name="exercise-3-replacing-dfs-replication-with-file-sync-based-replication"></a>练习 3：将“DFS 复制”替换为“基于文件同步的复制”

### <a name="task-1-add-sea-svr1-as-a-server-endpoint"></a>任务 1：添加 SEA-SVR1 作为服务器终结点

1. 在 SEA-ADM1 上，在 Azure门 户中“FileSync1 \| 注册的服务器”页面上，选择“Azure 文件同步代理”链接，转到“Azure 文件同步代理”的 Microsoft 下载页面   。  
1. 在“Azure 文件同步代理”的 Microsoft 下载页面上，选择“下载”，选中 Windows Server 2022 的文件同步代理条目旁边的复选框 (StorageSyncAgent_WS2022.msi)，然后选择“下一步”开始下载   。 下载完成后，关闭为下载打开的“Microsoft Edge”选项卡。
1. 使用文件资源管理器将下载的文件复制到“C:\\Labfiles\\Lab10”文件夹。
1. 在显示“C:\\Labfiles\\Lab10”文件夹内容的文件资源管理器中，在详细信息窗格中，选择文件“Install-FileSyncServerCore.ps1”，显示其上下文相关的菜单，然后在菜单中选择“编辑“  。

   >注意：这会自动在 Windows PowerShell ISE 的脚本窗格中打开 Install-FileSyncServerCore.ps1 文件 。

1. 在 Windows PowerShell ISE 脚本窗格中，查看脚本，然后通过选择工具栏中的“运行脚本”图标或按 F5 来执行该脚本 。 

   >注意：监视脚本执行。 这大约需要 3 分钟。

1. 当出现“WARNING”消息提示登录时，将警告消息中的九个字符的代码复制到剪贴板中。
1. 切换到显示 Azure 门户的 Microsoft Edge 窗口，通过选择 + 打开一个新选项卡，然后在新选项卡上浏览到 https://microsoft.com/devicelogin。
1. 在 Microsoft Edge 中，在“输入代码”对话框中，将复制的代码粘贴到剪贴板，然后，如果需要，在显示消息“是否正在尝试登录 Microsoft Azure PowerShell？”的页面上使用 Azure 凭据登录，然后选择“继续”，并关闭上一步中打开的“Microsoft Edge”选项卡  。
1. 切换到“Windows PowerShell ISE”窗口，并确保脚本成功完成。 
1. 切换回显示 Azure 门户的 Microsoft Edge 窗口，然后在“FileSync1 \| 注册的服务器”页面上，选择“刷新”以显示注册的服务器的当前列表 。
1. 验证“FileSync1”存储同步服务的注册的服务器列表上显示了“SEA-SVR1.Contoso.com”服务器 。
1. 在 SEA-ADM1 上，切换到文件资源管理器窗口，浏览到“\\\\SEA-SVR1\\Data”共享，并验证该文件夹当前不包含“File1.txt”  。
1. 切换到显示 Azure 门户的 Microsoft Edge 窗口，在 “FileSync1 \| 注册的服务器”页面上，选择“同步组”，选择“Sync1”，然后在 Sync1 页面上，选择“添加服务器终结点”    。
1. 在“添加服务器终结点”选项卡上，在“注册的服务器”列表中选择“SEA-SVR1.Contoso.com”  。
1. 在“路径”文本框中，输入“S:\\Data”，然后选择“创建”  。
1. 切换到“文件资源管理器”窗口，验证“SEA-SVR1\\Data”文件夹现在包含“File1.txt” **\\\\** 。

   >注意：已将“File1.txt”上传到 Azure 文件共享中，上传的位置是文件同步将其同步到 SEA-SVR1 的位置  。

### <a name="task-2-register-sea-svr2-with-file-sync"></a>任务 2：将 SEA-SVR2 注册到文件同步

1. 在 SEA-ADM1 上，切换到“Windows PowerShell ISE”窗口，进入显示“Install-FileSyncServerCore.ps1”文件的脚本窗格的选项卡  。
1. 在“Windows PowerShell ISE”脚本窗格的第一行，将 `SEA-SVR1` 替换为 `SEA-SVR2`，保存更改，并通过选择工具栏中的“运行脚本”图标或按 F5 执行脚本 。 

   >注意：监视脚本执行。 这大约需要 3 分钟。

1. 当出现“WARNING”消息提示登录时，将警告消息中的九个字符的代码复制到剪贴板中。
1. 切换到显示 Azure 门户的 Microsoft Edge 窗口，通过选择 + 打开一个新选项卡，然后在新选项卡上浏览到 https://microsoft.com/devicelogin。
1. 在 Microsoft Edge 中，在“输入代码”对话框中，将复制的代码粘贴到剪贴板，然后，如果需要，在显示消息“是否正在尝试登录 Microsoft Azure PowerShell？”的页面上使用 Azure 凭据登录，然后选择“继续”，并关闭上一步中打开的“Microsoft Edge”选项卡  。
1. 切换到“Windows PowerShell ISE”窗口，并确保脚本成功完成。 
1. 脚本完成后，切换到显示 Azure 门户的 Microsoft Edge 窗口，并浏览回“FileSync1 \| 注册的服务器”页面。
1. 确认 SEA-SVR1.Contoso.com 和 SEA-SVR2.Contoso.com 现在均列为注册了 FileSync1 存储同步服务的服务器  。

### <a name="task-3-remove-dfs-replication-and-add-sea-svr2-as-a-server-endpoint"></a>任务 3：删除“DFS 复制”并添加 SEA-SVR2 作为服务器终结点

1. 在 SEA-ADM1 上，选择任务栏上的“DFS 管理” 。
1. 在“DFS 管理”的导航窗格中，右键单击或访问 Branch1 的上下文菜单，选择“删除”，选择“是，删除该复制组，停止复制所有关联的已复制的文件夹并删除该复制组的所有成员”选项，然后选择“确定”    。
1. 切换到显示 Azure 门户的 Microsoft Edge 窗口，浏览回 FileSync1 存储同步服务页面，在同步组列表中，选择“Sync1”，然后在“Sync1”页面上选择“添加服务器终结点”    。
1. 在“添加服务器终结点”窗格中，选择“注册的服务器”列表中的“SEA-SVR2.Contoso.com”，在“路径”文本框中输入“S:\\Data”，然后选择“创建”    。

## <a name="exercise-4-verifying-replication-and-enabling-cloud-tiering"></a>练习 4：验证复制并启用云分层

### <a name="task-1-verify-file-sync"></a>任务 1：验证文件同步

1. 在 SEA-ADM1 上，切换到显示“\\\\SEA-SVR1\\Data”共享内容的文件资源管理器窗口， 。 
1. 在“\\SEA-SVR1\\Data **”文件夹中创建另一个任意命名的文件\\** 。
1. 在 SEA-ADM1 上，切换到显示“\\\\SEA-SVR2\\Data”共享内容的文件浏览器窗口，并验证不久之后，同名文件也出现在“\\\\SEA-SVR2\\Data”文件夹中  。

   >注意：已在上一个练习中删除了“DFS 复制”，这意味着“文件同步”复制了新创建的文件。

### <a name="task-2-enable-cloud-tiering"></a>任务 2：启用云分层

1. 在 SEA-ADM1 上，在 Azure 门户的 Sync1 同步组页面上，选择“服务器终结点”部分的“SEA-SVR2.Contoso.com”   。
1. 在“服务器终结点属性”窗格中，选择“云分层”部分的“启用” 。
1. 在“始终在卷上保留指定百分比的可用空间”文本框中，输入“90”，并将“日期策略”设置为“启用”   。 在“仅缓存在指定天数内访问或修改过的文件”文本框中，输入“14”，然后选择“保存”  。

   >注意：一段时间后，SEA-SVR2 上的文件将自动分层 。 将使用 PowerShell 触发此过程。

1. 在 SEA-ADM1 上，切换到“Windows PowerShell ISE”窗口 ：
1. 在 Windows PowerShell ISE 的控制台窗格中，通过输入以下命令并在每个命令后按 Enter 键，立即触发分层：

   ```powershell
   Enter-PSSession -computername SEA-SVR2
   fsutil file createnew S:\Data\report1.docx 254321098
   fsutil file createnew S:\Data\report2.docx 254321098
   fsutil file createnew S:\Data\report3.docx 254321098
   fsutil file createnew S:\Data\report4.docx 254321098
   Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
   Invoke-StorageSyncCloudTiering -Path S:\Data 
   ```
1. 在 SEA-ADM1 上，切换到显示“\\\\SEA-SVR2\\Data”文件夹内容的文件资源管理器窗口， 。
1. 在文件浏览器窗口中，通过右键单击或访问详细信息窗格中“标题”列的上下文菜单，在详细信息窗格中添加“属性”列；例如，在“名称”列中，选择“更多”，选择“属性”复选框，然后选择“确定”     。
1. 将“属性”列拖到“名称”列旁边，然后记下文件日期及其属性 。
1. 使用属性 L、M 和 O 对文件进行标识，指示发生了分层  。 

## <a name="exercise-5-troubleshooting-replication-issues"></a>练习 5：排查复制问题

### <a name="task-1-monitor-file-sync-replication"></a>任务 1：监视文件同步复制

1. 在 SEA-ADM1 上，使用文件资源管理器将 C:\\Windows\\INF 文件夹复制到“\\\\SEA-SVR2\\Data”  **\\** 。 文件夹将同步到云终结点，这将导致同步流量。
1. 在 SEA-ADM1 上，切换到显示 FileSync1 存储同步服务的 Sync1 同步组页面的 Azure 门户  。
1. 在“服务器终结点”部分，验证两个终结点的“运行状况”是否都有绿色复选标记 。
1. 选择“服务器终结点属性”窗格中的“SEA-SVR2.Contoso.com”终结点，查看“同步活动”，然后关闭窗格 。
1. 选择“同步文件”图，然后探索如何使用过滤器自定义该图。
1. 切换到显示映射到 Azure 文件共享的驱动器 Z 的内容的文件资源管理器窗口，并验证该驱动器是否包含从“\\\\SEA-SVR2\\Data”同步的 INF 文件夹的内容  。
1. 切换到 Azure 门户并验证“同步文件”和“同步字节”图中是否反映了 INF 同步流量  。 INF 文件夹有 800 多个文件，大小超过 40 MB。

   >注意：可能需要刷新显示 Azure 门户的页面来查看更新的统计信息。

### <a name="task-2-test-replication-conflict-resolution"></a>任务 2：测试复制冲突的解决

1. 在 SEA-ADM1 上，找到并排显示 SEA-SVR1\Data 和 SEA-SVR2\Data 内容的文件浏览器窗口 **\\\\\\** **\\\\\\** 。
1. 在显示 SEA-SVR1\Data 内容的文件资源管理器窗口中，创建一个名为“Demo.txt”的文件 **\\\\\\** 。 
1. 在显示 SEA-SVR2\Data 内容的文件资源管理器窗口中，创建一个名为“Demo.txt”的文件 **\\\\\\** 。 
1. 在第一个“Demo.txt”文件中添加任意文本并保存更改。
1. 紧接着向第二个“Demo.txt”文件添加任意文本（与上一步中使用的文本不同）并保存更改。

   >注意：请确保尽快将更改保存到第二个文件。 你要创建名称相同但内容不同的文件，故意触发同步冲突。

1. 在每个文件资源管理器窗口中，查看其中内容并验证除“Demo.txt”文件外还包含什么，同时检查是否有 Demo-SEA-SVR2.txt（可能是 Demo Cloud.txt）  。 

   >注意：这是因为文件同步检测到同步冲突，并在导致冲突的文件中添加了表示终结点名称 (SEA-SVR2) 或“云”的后缀  。

   >注意：可能需要等待几分钟才会看到同步冲突。

## <a name="exercise-6-cleaning-up-the-azure-subscription"></a>练习 6：清理 Azure 订阅

### <a name="task-1-delete-the-azure-resources-that-were-created-in-the-lab"></a>任务1：删除在实验中创建的 Azure 资源

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，并浏览到“FileSync1 存储同步服务”页面 。
1. 在“存储同步服务”页面中，选择“注册的服务器” 。
1. 在详细信息窗格中，右键单击或访问 SEA-SVR2.Contoso.com 的上下文菜单，然后选择“注销服务器” 。
1. 在“注销服务器”窗格的文本框中输入“SEA-SVR2.Contoso.com”，然后选择“注销” 。
1. 在“存储同步服务”窗格中，选择“注册的服务器”。
1. 在详细信息窗格中，右键单击或访问 SEA-SVR1.Contoso.com 的上下文菜单，然后选择“注销服务器” 。 
1. 在“注销服务器”窗格的文本框中输入“SEA-SVR1.Contoso.com”，然后选择“注销” 。
1. 等待两台服务器的注册被删除。
1. 在存储同步服务窗格中，选择“同步组”，然后在详细信息窗格中，选择“Sync1” 。
1. 在 Sync1 窗格中，右键单击或访问“云终结点”部分中 share1 的上下文菜单，选择“删除”，然后选择“确定”   。
1. 等待 share1 被删除。
1. 选择“删除”，然后选择“确定” 。
1. 在导航窗格中，选择“所有资源”。
1. 在详细信息窗格中，选择“FileSync1”和在此实验中创建的 Azure 存储帐户。
1. 在“删除资源”窗格中，选择“删除”，在文本框中输入“是”，然后选择“删除”  。
1. 在导航窗格中，选择“资源组”。
1. 在详细信息窗格中，选择“AZ800-L1001-RG”，选择“删除资源组”，输入“AZ800-L1001-RG”，然后选择“删除”   。
