---
lab:
  title: 实验室：管理 Windows Server
  type: Answer Key
  module: 'Module 3: Windows Server administration'
---

# <a name="lab-answer-key-managing-windows-server"></a>实验室答案密钥：管理 Windows Server

## <a name="exercise-1-implementing-and-using-remote-server-administration"></a>练习 1：实现和使用远程服务器管理

#### <a name="task-1-install-windows-admin-center"></a>任务 1：安装 Windows Admin Center

1. 连接到 SEA-ADM1，然后根据需要，以 Contoso\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait until the installation completes. This should take about 2 minutes.
   
   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: After installation completes, you may encounter the error message 'ERR_Connection_Refused'. If this occurs, restart SEA-ADM1 to correct the issue.

#### <a name="task-2-add-servers-for-remote-administration"></a>任务 2：添加用于远程管理的服务器

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 `https://SEA-ADM1.contoso.com`。 
1. 出现提示时，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 查看“此版本中的新增功能”弹出窗口，然后在其右上角选择“关闭” 。
1. 查看“所有连接”页，注意它包含 sea-adm1.contoso.com 条目 。 
1. 在“所有连接”页面上，选择“+ 添加” 。 
1. 在“添加或创建资源”窗格中的“服务器”磁贴上，选择“添加” 。
1. 在“服务器名称”文本框中，输入“sea-dc1.contoso.com” 。
1. 确保已选中“为此连接使用另一个帐户”选项，输入以下凭据，然后选择“使用凭据添加” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: After performing step 7, if an error message that says <bpt id="p2">**</bpt>You can add this server to your list of connections, but we can't confirm it's available.<ept id="p2">**</ept> appears, select <bpt id="p1">**</bpt>Add<ept id="p1">**</ept>. In the All Connections pane,  select <bpt id="p1">**</bpt>sea-svr1.contoso.com<ept id="p1">**</ept>, and then select <bpt id="p2">**</bpt>Manage as<ept id="p2">**</ept>. In the <bpt id="p1">**</bpt>Specify your credentials<ept id="p1">**</ept> dialog box, ensure that the <bpt id="p2">**</bpt>Use another account for this connection<ept id="p2">**</ept> option is selected, enter the Administrator credentials, and then select <bpt id="p3">**</bpt>Continue<ept id="p3">**</ept>.

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派。

#### <a name="task-3-configure-windows-admin-center-extensions"></a>任务 3：配置 Windows Admin Center 扩展

1. 在 SEA-ADM1 上，在显示 Windows Admin Center 的 Microsoft Edge 窗口的右上角，选择“设置”图标（齿轮） 。
1. In the left pane, select <bpt id="p1">**</bpt>Extensions<ept id="p1">**</ept>. Review the available extensions.
1. Select the <bpt id="p1">**</bpt>Security (Preview)<ept id="p1">**</ept> extension, and then select <bpt id="p2">**</bpt>Install<ept id="p2">**</ept>. The extension will install and Windows Admin Center will refresh.

   > 注意：如果“安全性(预览版)”扩展不可用，请选择其他 Microsoft 扩展 。

1. 在“详细信息”窗格中，选择“已安装扩展”并确认列表包含“DNS (预览版)”扩展。
1. 在顶部菜单的“设置”旁边，选择下拉箭头，然后选择“服务器管理器” 。
1. 在“服务器连接”页上，选择“sea-dc1.contoso.com”链接 。
1. 确保已选中“对此连接使用另一个帐户”选项，选择“对所有连接使用这些凭据”，输入以下凭据，然后选择“继续”  ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. To install the DNS PowerShell tools, in the left pane, in the list of <bpt id="p1">**</bpt>Tools<ept id="p1">**</ept>, select <bpt id="p2">**</bpt>DNS<ept id="p2">**</ept>, and then select <bpt id="p3">**</bpt>Install<ept id="p3">**</ept>. The tools will take less than a minute to install.
1. 选择“Contoso.com”区域并查看其 DNS 记录的列表。

#### <a name="task-4-verify-remote-administration"></a>任务 4：验证远程管理

1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, in Windows Admin Center, in the left pane, in the list of <bpt id="p2">**</bpt>Tools<ept id="p2">**</ept>, select <bpt id="p3">**</bpt>Overview<ept id="p3">**</ept>. Note that the details pane of Windows Admin Center displays basic server information and performance monitoring.
1. In the left pane, in the list of <bpt id="p1">**</bpt>Tools<ept id="p1">**</ept>, scroll down and review the basic administration tools available. Select <bpt id="p1">**</bpt>Roles &amp; features<ept id="p1">**</ept> and note which roles and features are listed as installed and which ones are available to install. Scroll down, select the <bpt id="p1">**</bpt>Telnet Client<ept id="p1">**</ept> checkbox, and then select <bpt id="p2">**</bpt>+ Install<ept id="p2">**</ept> at the top of the pane.
1. 在“安装角色和功能”窗格中，选择“是”，等待确认已成功安装 Telnet 客户端的消息 。
1. 在左侧窗格的最底部，在“工具”列表下，选择“设置” 。
1. 在右侧的“设置”部分，选择“远程桌面” 。
1. 在“远程桌面”部分中，选中“允许与此计算机的远程连接”复选框，然后选择“保存”  。
1. 在左侧窗格的“工具”列表中，选择“远程桌面” 。
1. 在远程桌面窗格中，选中“自动连接到此计算机提供的证书”复选框，然后选择“连接” 。
1. 出现提示时，选择“确认”，然后选择“连接” 。
1. 确认通过远程桌面在 Windows Admin Center 界面中成功连接到的 SEA-DC1。
1. 选择“断开”。
1. 关闭 Microsoft Edge 窗口。

#### <a name="task-5-administer-servers-with-remote-powershell"></a>任务 5：使用远程 PowerShell 管理服务器

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

### <a name="results"></a>结果

After completing this exercise, you will have installed Windows Admin Center and connected it to the servers in your lab environment. You performed a number of remote management tasks including installing a feature as well as enabling and testing Remote Desktop connectivity. Finally, you used PowerShell Remoting to check the status of a service and then to start it.
