---
lab:
  title: 实验室：实现 Azure 文件同步
  module: 'Module 10: Implementing a hybrid file server infrastructure'
---

# <a name="lab-implementing-azure-file-sync"></a>实验室：实现 Azure 文件同步

## <a name="scenario"></a>场景

为了解决关于 Contoso 伦敦总部和西雅图分公司之间分布式文件系统 (DFS) 复制的问题，你决定测试 Azure 文件同步是否能成为两个本地文件共享之间的替代复制机制。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 在本地环境中实施 DFS 复制。
- 创建和配置同步组。
- 将 DFS 复制替换为基于 Azure 文件同步的复制。
- 验证复制并启用云分层。
- 对复制冲突进行故障排除。

## <a name="estimated-time-60-minutes"></a>预计时间：60 分钟

## <a name="lab-setup"></a>实验室设置

虚拟机：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1、AZ-800T00A-SEA-SVR2 和 AZ-800T00A-ADM1 必须处于运行状态   。 

> 注意：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1、AZ-800T00A-SEA-SVR2 和 AZ-800T00A-SEA-ADM1 虚拟机分别托管 SEA-DC1、SEA-SVR1、SEA-SVR2 和 SEA-ADM1 的安装        。

1. 选择“SEA-ADM1”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

For this lab, you'll use the available VM environment and an Azure subscription. Before you begin the lab, ensure that you have an Azure subscription and a user account with the Owner or Contributor role in that subscription.

## <a name="exercise-1-implementing-dfs-replication-in-your-on-premises-environment"></a>练习 1：在本地环境中实施 DFS 复制

### <a name="scenario"></a>场景

练习场景：在开始测试本地 DFS 复制迁移之前，首先需要在 SVR1 和 SVR2的概念证明环境中实现 DFS 复制操作 。

此练习的主要任务是：

1. 部署 DFS。
1. 测试 DFS 部署。

#### <a name="task-1-deploy-dfs"></a>任务 1：部署 DFS

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 并通过运行以下命令安装分布式文件系统 (DFS) 管理工具：

   ```powershell
   Install-WindowsFeature -Name RSAT-DFS-Mgmt-Con -IncludeManagementTools
   ```
1. 在 SEA-ADM1 上，打开文件资源管理器，浏览至 C:\\Labfiles\\Lab10 文件夹，并在新的 Windows PowerShell ISE 窗口的脚本窗格中打开 L10-DeployDFS.ps1   。
1. 在脚本窗格中查看脚本，然后执行该脚本以创建示例 DFS 命名空间和 DFS 复制组。

#### <a name="task-2-test-dfs-deployment"></a>任务 2：测试 DFS 部署

1. 在 ADM1 上，启动 DFS 管理控制台并将其添加到在上一任务中创建的 \\\\Contoso.com\\Root\\ 命名空间和 Branch1 复制组   。 
1. Verify that the <bpt id="p1">**</bpt><ph id="ph1">\\</ph><ph id="ph2">\\</ph>Contoso.com<ph id="ph3">\\</ph>Root<ph id="ph4">\\</ph>Data<ept id="p1">**</ept> folder has targets on <bpt id="p2">**</bpt>SEA-SVR1<ept id="p2">**</ept> and <bpt id="p3">**</bpt>SEA-SVR2<ept id="p3">**</ept>. Note the folders configured as the targets.
1. Verify that the <bpt id="p1">**</bpt>Branch1<ept id="p1">**</ept> replication group has two members, <bpt id="p2">**</bpt>SEA-SVR1<ept id="p2">**</ept> and <bpt id="p3">**</bpt>SEA-SVR2<ept id="p3">**</ept>. Note the folders replicated on each server.
1. Open two instances of File Explorer. In the first instance, connect to <bpt id="p1">**</bpt><ph id="ph1">\\</ph><ph id="ph2">\\</ph>SEA-SVR1<ph id="ph3">\\</ph>Data<ept id="p1">**</ept>, and then in the second instance, connect to <bpt id="p2">**</bpt><ph id="ph4">\\</ph><ph id="ph5">\\</ph>SEA-SVR2<ph id="ph6">\\</ph>Data<ept id="p2">**</ept>.
1. Create a new file with your name in <bpt id="p1">**</bpt><ph id="ph1">\\</ph><ph id="ph2">\\</ph>SEA-SVR1<ph id="ph3">\\</ph>Data<ept id="p1">**</ept>, and then confirm that the file replicates to <bpt id="p2">**</bpt><ph id="ph4">\\</ph><ph id="ph5">\\</ph>SEA-SVR2<ph id="ph6">\\</ph>Data<ept id="p2">**</ept> after a few seconds. This confirms that DFS Replication is working.

   >注意：等待文件复制完毕，两个文件资源管理器窗口中会记录相同的内容。

### <a name="results"></a>结果

After completing this exercise, you'll have created a working DFS infrastructure. This includes DFS Replication, which replicates content between <bpt id="p1">**</bpt>SEA-SVR1<ept id="p1">**</ept> and <bpt id="p2">**</bpt>SEA-SVR2<ept id="p2">**</ept>.

## <a name="exercise-2-creating-and-configuring-a-sync-group"></a>练习 2：创建和配置同步组

### <a name="scenario"></a>场景

若要准备将 DFS 复制环境迁移到文件同步，必须先创建并配置一个文件同步组。

此练习的主要任务是：

1. 创建 Azure 文件共享。
1. 使用 Azure 文件共享。
1. 部署存储同步服务和文件同步组。

#### <a name="task-1-create-an-azure-file-share"></a>任务 1：创建 Azure 文件共享

1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到 Azure 门户，然后使用 Azure 凭据进行身份验证。
1. 在 Azure 门户中，在名为 AZ800-L1001-RG 的资源组中创建具有本地冗余存储 (LRS) 的 Azure 存储帐户 。

   >注意：请使用同一区域来部署此实验中的所有 Azure 资源。

1. 在存储帐户中，创建一个名为 share1 的文件共享。

#### <a name="task-2-use-an-azure-file-share"></a>任务 2：使用 Azure 文件共享

1. 在 SEA-ADM1 上，将 C:\\Labfiles\\Lab10\\File1.txt 上传至 share1  。
1. 在 Azure 门户中，创建 share1 的快照。
1. 在 SEA-ADM1 上，通过使用 Azure 门户提供的连接脚本，将 share1 装载为驱动器 Z  。
1. 在文件资源管理器中已装载驱动器上，打开名为 File1.txt 的文件，输入名称，然后保存该文件。
1. 使用文件资源管理器中的先前版本还原 File1.txt 的以前版本 。
1. 打开 File1.txt，然后验证其不包含你的名称。

#### <a name="task-3-deploy-storage-sync-service-and-a-file-sync-group"></a>任务 3：部署存储同步服务和文件同步组

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, use the Azure portal to create an Azure File Sync resource named <bpt id="p2">**</bpt>FileSync1<ept id="p2">**</ept>. Use the same region and Resource Group as you used when deploying the storage account.

   >**注意：** 部署文件同步将创建存储同步服务资源。

1. Create a sync group named <bpt id="p1">**</bpt>Sync1<ept id="p1">**</ept> in the <bpt id="p2">**</bpt>FileSync1<ept id="p2">**</ept> Storage Sync Service. Use the storage account that you created earlier and <bpt id="p1">**</bpt>share1<ept id="p1">**</ept> as the Azure file share when creating <bpt id="p2">**</bpt>Sync1<ept id="p2">**</ept>.
1. 验证当前没有任何服务器注册到 FileSync1。

### <a name="results"></a>结果

After completing this exercise, you will have created a File Sync group. You also have created the cloud endpoint mapped on <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept> so that you can inspect the Azure file share content.

## <a name="exercise-3-replacing-dfs-replication-with-file-sync-based-replication"></a>练习 3：将 DFS 复制替换为基于文件同步的复制

### <a name="scenario"></a>场景

现在已准备好所有必要的组件，请使用基于文件同步的复制来替换 DFS 复制。

此练习的主要任务是：

1. 添加 SEA-SVR1 作为服务器终结点。
1. 将 SEA-SVR2 注册到文件同步。
1. 删除“DFS 复制”并添加 SEA-SVR2 作为服务器终结点。

#### <a name="task-1-add-sea-svr1-as-a-server-endpoint"></a>任务 1：添加 SEA-SVR1 作为服务器终结点

1. 在 SEA-ADM1 上，在 Azure 门户中，下载适用于 Windows Server 2022 的文件同步代理 (StorageSyncAgent_WS2022.msi)，然后将其保存至 C:\\\\Labfiles\\Lab10 文件夹  。
1. 在 SEA-ADM1 上，在文件资源管理器中，浏览至 C:\\Labfiles\\Lab10 文件夹，并在 Windows PowerShell ISE 窗口的脚本窗格中打开 Install-FileSyncServerCore.ps1  。
1. 在 Windows PowerShell ISE 脚本窗格中，查看该脚本，然后执行该脚本以在 SEA-SVR1 上安装文件同步代理 。
1. 出现提示时，请向 Azure 订阅进行身份验证。 
1. 在 Azure 门户中，刷新 FileSync1 存储同步服务中已注册的服务器，然后指出 SEA-SVR1.Contoso.com 现在已注册 。
1. 在文件资源管理器中，打开 \\\\SEA-SVR1\\Data，并指出文件夹不包含 File1.txt 。
1. 使用 Azure 门户将 SEA-SVR2.Contoso.com 上的 S:\\Data 添加为 Sync1 上的服务器终结点  。
1. 使用文件资源管理器验证 \\\\SEA-SVR1\\Data\\ 中有 File1.txt 。

   >注意：已将“File1.txt”上传到 File1.txtAzure 文件共享中，上传的位置是文件同步将其同步到 SEA-SVR2 的位置   。

#### <a name="task-2-register-sea-svr2-with-file-sync"></a>任务 2：将 SEA-SVR2 注册到文件同步

1. 在 SEA-ADM1 上，在显示 Install-FileSyncServerCore.ps1 脚本的 Windows PowerShell ISE 窗口中，将 `SEA-SVR1` 替换为 `SEA-SVR2`然后保存更改 。
1. 运行 C:\\Labfiles\\Lab10\\Install-FileSyncServerCore.ps1，在 SEA-SVR2 上安装文件同步代理 。
1. 出现提示时，请向 Azure 订阅进行身份验证。 
1. 使用 Azure 门户验证 SEA-SVR2.Contoso.com 和 SEA-SVR1.Contoso.com 现在是否注册了 FileSync1 存储同步服务  。

#### <a name="task-3-remove-dfs-replication-and-add-sea-svr2-as-a-server-endpoint"></a>任务 3：删除“DFS 复制”并添加 SEA-SVR2 作为服务器终结点

1. 在 SEA-ADM1 上，使用 DFS 管理删除 Branch1 复制组  。
1. 使用 Azure 门户将 SEA-SVR2.Contoso.com 上的 S:\\Data 添加为 Sync1 上的服务器终结点  。

### <a name="results"></a>结果

完成本练习后，你会将 DFS 复制替换为文件同步。

## <a name="exercise-4-verifying-replication-and-enabling-cloud-tiering"></a>练习 4：验证复制并启用云分层

### <a name="scenario"></a>场景

练习场景：你现在需要验证是否已成功将 DFS 复制替换为文件同步，确认之后，需要启用云分层。

此练习的主要任务是：

1. 验证文件同步。
1. 启用云分层。

#### <a name="task-1-verify-file-sync"></a>任务 1：验证文件同步

1. 在 SEA-ADM1 上，使用两个文件资源管理器实例显示 \\\\SEA-SVR1\\Data 和 \\\\SEA-SVR2\\Data 的内容  。
1. 使用 \\\\SEA-SVR1\\Data 文件夹中创建任意命名的文件。
1. 验证具有相同名称的文件是否在不久之后出现在 \\\\SEA-SVR2\\Data 文件夹中。

   >注意：已在上一个练习中删除了“DFS 复制”，这意味着“文件同步”复制了新创建的文件。

#### <a name="task-2-enable-cloud-tiering"></a>任务 2：启用云分层

1. 在 SEA-ADM1 上，使用 Azure 门户浏览至 FileSync1 存储同步服务中的 Sync1 同步组  。
1. In the Azure portal, enable cloud tiering for the <bpt id="p1">**</bpt>SEA-SVR1.Contoso.com<ept id="p1">**</ept> endpoint in <bpt id="p2">**</bpt>Sync1<ept id="p2">**</ept>. Set the <bpt id="p1">**</bpt>free disk space<ept id="p1">**</ept> policy to <bpt id="p2">**</bpt>80<ept id="p2">**</ept> percent and the <bpt id="p3">**</bpt>date policy<ept id="p3">**</ept> to cache files that were accessed in the last <bpt id="p4">**</bpt>7<ept id="p4">**</ept> days.
1. 在连接到 \\\\SEA-SVR1\\Data 文件夹的文件资源管理器实例中，在详细信息窗格中，通过右键单击或访问“标题”列的上下文菜单添加“属性”列；例如，在“名称”列中选择“更多”，然后选择“属性”     。

   ><bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> After some time, files on <bpt id="p2">**</bpt>SEA-SVR2<ept id="p2">**</ept> would be automatically tiered. You will trigger this process by using PowerShell.

1. 在 SEA-ADM1 中，在 Windows PowerShell ISE 的控制台窗格中，通过运行以下命令立即触发分层 ：

   ```powershell
   Enter-PSSession -computername SEA-SVR2
   fsutil file createnew S:\Data\report1.docx 254321098
   fsutil file createnew S:\Data\report2.docx 254321098
   fsutil file createnew S:\Data\report3.docx 254321098
   fsutil file createnew S:\Data\report4.docx 254321098
   Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
   Invoke-StorageSyncCloudTiering -Path S:\Data 
   ```
1. 在 ADM1 上切换到文件资源管理器，然后在 \\\\SEA-SVR2\\Data 共享上，使用属性 L、M 和 O 对文件进行标识，指示发生了分层    。

### <a name="results"></a>结果

完成本练习后，你将创建正常运行的文件同步复制和已配置的云分层。

## <a name="exercise-5-troubleshooting-replication-issues"></a>练习 5：排查复制问题

### <a name="scenario"></a>场景

Exercise scenario: Contoso relies heavily on its DFS Replication implementation. You must ensure that any replication issues, including replication conflicts, can be quickly identified and resolved. To do so, you'll simulate the most common replication issues in your proof-of-concept environment and test their resolutions.

此练习的主要任务是：

1. 监视文件同步复制。
1. 测试复制冲突的解决方法。

#### <a name="task-1-monitor-file-sync-replication"></a>任务 1：监视文件同步复制

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, use File Explorer to copy the <bpt id="p2">**</bpt>C:<ph id="ph1">\\</ph>Windows<ph id="ph2">\\</ph>INF<ept id="p2">**</ept> folder to <bpt id="p3">**</bpt><ph id="ph3">\\</ph><ph id="ph4">\\</ph>SEA-SVR1\Data<ph id="ph5">\\</ph><ept id="p3">**</ept>. The folder will sync to the cloud endpoint, causing sync traffic.
1. 在 Azure 门户中浏览至 FileSync1 存储同步服务中的 Sync1 同步组 。
1. 在“服务器终结点”部分中，验证两个终结点的运行状况 。
1. 选择“SEA-SVR1.Contoso.com”终结点，然后在“服务器终结点属性”窗格中查看“同步活动” 。
1. 选择“同步文件”图，探索如何使用过滤器自定义该图。
1. 验证 INF 文件夹是否正在同步到驱动器 Z 。
1. In the Azure portal, verify that the <bpt id="p1">**</bpt>INF<ept id="p1">**</ept> sync traffic is reflected in the <bpt id="p2">**</bpt>Files Synced<ept id="p2">**</ept> and <bpt id="p3">**</bpt>Bytes Synced<ept id="p3">**</ept> graphs. The <bpt id="p1">**</bpt>INF<ept id="p1">**</ept> folder has more than 800 files, and its size is more than 40 megabytes (MB).

   >注意：可能需要刷新显示 Azure 门户的页面来查看更新的统计信息。

#### <a name="task-2-test-replication-conflict-resolution"></a>任务 2：测试复制冲突的解决

1. 在 SEA-ADM1 上，找到并排显示 SEA-SVR1\Data 和 SEA-SVR2\Data 内容的文件浏览器窗口 **\\\\\\** **\\\\\\** 。
1. 在显示 SEA-SVR1\Data 内容的文件资源管理器窗口中，创建一个名为“Demo.txt”的文件 **\\\\\\** 。 
1. 在显示 SEA-SVR2\Data 内容的文件资源管理器窗口中，创建一个名为“Demo.txt”的文件 **\\\\\\** 。 
1. 在第一个“Demo.txt”文件中添加任意文本并保存更改。
1. 紧接着向第二个“Demo.txt”文件添加任意文本（与上一步中使用的文本不同）并保存更改。

   ><bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> Make sure to save the change to the second file as soon as possible. You're creating files with the same name but different content to intentionally trigger a sync conflict.

1. 在每个文件资源管理器窗口中，查看其中内容并验证除“Demo.txt”文件外还包含什么，同时检查是否有 Demo-SEA-SVR2.txt（可能是 Demo Cloud.txt）  。 

   >注意：这是因为文件同步检测到同步冲突，并在导致冲突的文件中添加了表示终结点名称 (SEA-SVR2) 或“云”的后缀  。

### <a name="results"></a>结果

完成本练习后，你会监视文件同步复制并解决复制冲突。

## <a name="exercise-6-cleaning-up-the-azure-subscription"></a>练习 6：清理 Azure 订阅

练习场景：为了最大限度地减少与 Azure 相关的费用，你要对 Azure 订阅进行清理。

#### <a name="task-1-delete-the-azure-resources-that-were-created-in-the-lab"></a>任务 1：删除在实验室中创建的 Azure 资源

1. 在 SEA-ADM1 上，使用 Azure 门户浏览至 FileSync1 存储同步服务页面 。
1. 删除作为已注册服务器的 SEA-SVR1.Contoso.com 和 SEA-SVR2.Contoso.com 。
1. 删除 Sync1 同步组中的 share1 云终结点 。
1. 删除 Sync1 同步组。
1. 删除 FileSync1 存储同步服务以及在实验室中创建的 Azure 存储帐户。
1. 删除 AZ800-L1001-RG 资源组。

### <a name="results"></a>结果

完成本练习后，你会清理在实验室中创建的 Azure 资源。
