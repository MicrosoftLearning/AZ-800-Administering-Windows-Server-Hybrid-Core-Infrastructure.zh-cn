---
lab:
  title: 实验室：管理 Windows Server
  type: Answer Key
  module: 'Module 3: Windows Server administration'
---

# 实验室答案密钥：管理 Windows Server

## 练习 1：实现和使用远程服务器管理

#### 任务 1：安装 Windows Admin Center

1. 连接到 **SEA-ADM1**，如果需要，请使用讲师提供的凭据登录。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
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

   > 备注：请等待安装完成。 这大约需要 2 分钟。
   
   > **注意**：安装完成后，可能会遇到错误消息“ERR_Connection_Refused”。 如果发生这种情况，请重启 SEA-ADM1 以更正此问题。

#### 任务 2：添加用于远程管理的服务器

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 `https://SEA-ADM1.contoso.com`。

   >注意：如果链接不起作用，请在 SEA-ADM1 上打开文件资源管理器，选择“下载”文件夹，在“下载”文件夹中选择 WindowsAdminCenter.msi 文件并手动安装。 安装完成后，刷新 Microsoft Edge。

   >注意：如果收到 NET::ERR_CERT_DATE_INVALID 错误，请在 Microsoft Edge 浏览器页上选择“高级”，在页面底部选择“继续访问 sea-adm1-contoso.com (不安全)” 。

1. 出现提示时，在“**Windows 安全**”对话框中，输入讲师提供的凭据。
1. 查看“配置 Windows Admin Center 设置和环境”弹出窗口上的所有选项卡，包括“扩展”选项卡，然后选择“完成”以关闭窗口。************
1. 查看“此版本中的新增功能”弹出窗口，然后在其右上角选择“关闭” 。
1. 查看“所有连接”页，注意它包含 sea-adm1.contoso.com 条目 。 
1. 在“所有连接”页面上，选择“+ 添加” 。 
1. 在“添加或创建资源”窗格中的“服务器”磁贴上，选择“添加” 。
1. 在“服务器名称”文本框中，输入“sea-dc1.contoso.com” 。
1. 出现提示时，请使用讲师提供的凭据登录。

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派。

#### 任务 3：配置 Windows Admin Center 扩展

1. 在 SEA-ADM1 上，在显示 Windows Admin Center 的 Microsoft Edge 窗口的右上角，选择“设置”图标（齿轮） 。
1. 在左窗格中，选择“扩展”****。 查看可用的扩展。
1. 选择“DNS”扩展，然后选择“安装”。******** 该扩展将安装，Windows Admin Center 将刷新。
1. 在“详细信息”窗格中，选择“已安装扩展”并确认列表包含刚安装的扩展。
1. 在顶部菜单的“设置”旁边，选择下拉箭头，然后选择“服务器管理器” 。
1. 在“服务器连接”页上，选择“sea-dc1.contoso.com”链接 。

1. 若要安装 DNS PowerShell 工具，请在左侧窗格的“工具”列表中，选择“DNS”，然后选择“安装”  。 安装这些工具需要不到一分钟的时间。
1. 选择“Contoso.com”区域并查看其 DNS 记录的列表。

#### 任务 4：验证远程管理

1. 在 SEA-ADM1 上，在 Windows Admin Center 左侧窗格的“工具”列表中，选择“概述”  。 请注意，Windows Admin Center 的“详细信息”窗格显示基本的服务器信息和性能监视。
1. 在左侧窗格的“工具”列表中，向下滚动并查看可用的基本管理工具。 选择“角色和功能”，并注意哪些角色和功能被列为已安装，哪些角色和功能可以安装。 向下滚动，选中“Telnet 客户端”复选框，然后选择窗格顶部的“+ 安装” 。
1. 在“安装角色和功能”窗格中，选择“是”，等待确认已成功安装 Telnet 客户端的消息 。
1. 在左窗格最顶部的“工具”列表下，选择“设置”。********
1. 在右侧的“设置”部分，选择“远程桌面” 。
1. 在“远程桌面”部分中，选中“允许与此计算机的远程连接”复选框，然后选择“保存”  。
1. 在左侧窗格的“工具”列表中，选择“远程桌面” 。
1. 在“远程桌面”窗格中，输入讲师提供的密码，选中“自动连接到此计算机提供的证书”复选框，然后选择“连接”********。
1. 出现提示时，选择“确认”，然后选择“连接” 。
1. 确认通过远程桌面在 Windows Admin Center 界面中成功连接到的 SEA-DC1。
1. 选择“断开”。
1. 关闭 Microsoft Edge 窗口。

#### 任务 5：使用远程 PowerShell 管理服务器

1. 在 SEA-ADM1 上，切换到 PowerShell 控制台会话 。 
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按下 Enter 键，将一个 PowerShell 远程处理会话启动到 SEA-DC1 ：

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```
1. 从 [SEA-DC1] 提示符中，输入以下命令并按 Enter 以显示应用程序标识服务 (AppIDSvc) 的状态：

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > 注意：确认服务当前已停止。

1. 从 [SEA-DC1] 提示符中，输入以下命令并按 Enter 以启动应用程序标识服务：

   ```powershell
   Start-Service -Name AppIDSvc
   ```
1. 从 [SEA-DC1] 提示符中，输入以下命令并按 Enter 以显示应用程序标识服务 (AppIDSvc) 的状态：

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > 注意：确认服务当前正在运行。

### 结果

完成本练习后，你已安装 Windows Admin Center 并将其连接到实验室环境中的服务器。 你执行了许多远程管理任务，包括安装功能以及启用和测试远程桌面连接。 最后，你使用了 PowerShell 远程处理来检查服务的状态，然后启动它。
