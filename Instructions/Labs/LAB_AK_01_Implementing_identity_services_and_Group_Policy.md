---
lab:
  title: 实验室：实现标识服务和组策略
  type: Answer Key
  module: 'Module 1: Identity services in Windows Server'
---

# <a name="lab-answer-key-implementing-identity-services-and-group-policy"></a>实验室解答：实现标识服务和组策略

## <a name="exercise-1-deploying-a-new-domain-controller-on-server-core"></a>练习 1：在 Server Core 上部署新的域控制器

#### <a name="task-1-deploy-ad-ds-on-a-new-windows-server-core-server"></a>任务 1：在新的 Windows Server Core 服务器上部署 AD DS

1. 连接到 SEA-ADM1，然后根据需要，以 Contoso\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。
1. 要安装 AD DS 服务器角色，请在 Windows PowerShell 命令提示符处输入以下命令，然后按 Enter：
    
   ```powershell
   Install-WindowsFeature –Name AD-Domain-Services –ComputerName SEA-SVR1
   ```
1. 要验证 AD DS 角色是否已安装在 SEA-SVR1 上，请输入以下命令，然后按 Enter：
    
   ```powershell
   Get-WindowsFeature –ComputerName SEA-SVR1
   ```
1. In the output of the previous command, search for the <bpt id="p1">**</bpt>Active Directory Domain Services<ept id="p1">**</ept> checkbox, and then verify that it is selected. Then, search for <bpt id="p1">**</bpt>Remote Server Administration Tools<ept id="p1">**</ept>. Notice the <bpt id="p1">**</bpt>Role Administration Tools<ept id="p1">**</ept> node below it, and then verify that the <bpt id="p2">**</bpt>AD DS and AD LDS Tools<ept id="p2">**</ept> node is also selected.

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Under the <bpt id="p2">**</bpt>AD DS and AD LDS Tools<ept id="p2">**</ept> node, only <bpt id="p3">**</bpt>Active Directory module for Windows PowerShell<ept id="p3">**</ept> has been installed and not the graphical tools, such as the Active Directory Administrative Center. If you centrally manage your servers, you will not usually need these on each server. If you want to install them, you must specify the AD DS tools by running the <bpt id="p1">**</bpt>Add-WindowsFeature<ept id="p1">**</ept> cmdlet with the <bpt id="p2">**</bpt>RSAT-ADDS<ept id="p2">**</ept> command.

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: You might need to wait a brief time after the installation process is complete before verifying that the AD DS role has installed. If you do not observe the expected results from the <bpt id="p1">**</bpt>Get-WindowsFeature<ept id="p1">**</ept> command, you can try again after a few minutes.

#### <a name="task-2-prepare-the-ad-ds-installation-and-promote-a-remote-server"></a>任务 2：准备安装 AD DS 并提升远程服务器

1.  在 SEA-ADM1 的“开始”菜单上，选择“服务器管理器”，然后在“服务器管理器”视图中选择“所有服务器”视图    。
1. 在“管理”菜单上，选择“添加服务器” 。
1. 在“添加服务器”对话框中，保留默认设置，然后选择“立即查找” 。
1. 在服务器的 Active Directory 列表中，选择 SEA-SVR1，选择箭头将其添加到“选定项”列表，然后选择“确定”   。
1. On <bpt id="p1">**</bpt>SEA-ADM1<ept id="p1">**</ept>, ensure that the installation of the AD DS role on <bpt id="p2">**</bpt>SEA-SRV1<ept id="p2">**</ept> is complete and that the server was added to <bpt id="p3">**</bpt>Server Manager<ept id="p3">**</ept>. Then select the <bpt id="p1">**</bpt>Notifications<ept id="p1">**</ept> flag symbol.
1. 记下 SEA-SVR1 的部署后配置，然后选择“将此服务器提升到域控制器”链接 。
1. 在“Active Directory 域服务配置向导”的“部署配置”页上，在“选择部署操作”下，确认已选中“向现有域添加域控制器”   。
1. 确保指定了 `Contoso.com` 域，然后在“提供凭据以执行此操作”部分，选择“更改” 。
1. 在“部署操作的凭据”对话框的“用户名”框中，输入“CONTOSO\\Administrator”，在“密码”框中，输入“Pa55w.rd”    。
1. 选择“确定”  ，然后选择“下一步”  。
1. On the <bpt id="p1">**</bpt>Domain Controller Options<ept id="p1">**</ept> page, ensure that the <bpt id="p2">**</bpt>Domain Name System (DNS) server<ept id="p2">**</ept> and <bpt id="p3">**</bpt>Global Catalog (GC)<ept id="p3">**</ept> checkboxes are selected. Ensure that the <bpt id="p1">**</bpt>Read-only domain controller (RODC)<ept id="p1">**</ept> checkbox is cleared.
1. 在“键入目录服务还原模式(DSRM)密码”部分中，输入并确认密码“Pa55w.rd”，然后选择“下一步”  。
1. 在“DNS 选项”页上，选择“下一步” 。
1. 在“其他选项”页上，选择“下一步” 。
1. 在“路径”页上，保留“Database”文件夹、“Log files”文件夹和“SYSVOL”文件夹的默认路径设置，然后选择“下一步”    。
1. 要打开生成的 Windows PowerShell 脚本，请在“查看选项”页上选择“查看脚本” 。
1. 在记事本中，编辑生成的 Windows PowerShell 脚本：

   - 删除以数字符号 (#) 开头的注释行。
   - 删除 Import-Module 行。
   - 删除每行末尾的重音符号 (`)。
   - 删除换行符。

1. Now the <bpt id="p1">**</bpt>Install-ADDSDomainController<ept id="p1">**</ept> command and all the parameters are on one line. Place the cursor in front of the line, and then, on the <bpt id="p1">**</bpt>Edit<ept id="p1">**</ept> menu, select <bpt id="p2">**</bpt>Select All<ept id="p2">**</ept> to select the whole line. On the menu, select <bpt id="p1">**</bpt>Edit<ept id="p1">**</ept>, and then select <bpt id="p2">**</bpt>Copy<ept id="p2">**</ept>.
1. 切换到 Active Directory 域服务配置向导，然后选择“取消” 。
1. 当系统提示确认时，选择“是”以取消该向导。
1. 在 Windows PowerShell 命令提示符下，输入以下命令：

   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 { }
   ```
1. Place the cursor between the braces (<bpt id="p1">**</bpt>{ }<ept id="p1">**</ept>), and then paste the content of the copied script line from the clipboard. The complete command should have the following format:
    
   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 {Install-ADDSDomainController -NoGlobalCatalog:\$false -CreateDnsDelegation:\$false -Credential (Get-Credential) -CriticalReplicationOnly:\$false -DatabasePath "C:\Windows\NTDS" -DomainName "Contoso.com" -InstallDns:\$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:\$false -SiteName "Default-First-Site-Name" -SysvolPath "C:\Windows\SYSVOL" -Force:\$true}
   ```
1. 要调用命令，请按 Enter。
1. 在“Windows PowerShell 凭据请求”对话框中，在“用户名”框中输入 CONTOSO\\Administrator，在“密码”框中输入 Pa55w.rd，然后选择“确定”     。
1. 当系统提示输入密码时，在 SafeModeAdministratorPassword 文本框中，输入 Pa55w.rd，然后按 Enter 。
1. 当系统提示进行确认时，在 Confirm SafeModeAdministratorPassword 文本框中，输入 Pa55w.rd，然后按 Enter 。
1. Wait until the command runs and the <bpt id="p1">**</bpt>Status Success<ept id="p1">**</ept> message is returned. The <bpt id="p1">**</bpt>SEA-SVR1<ept id="p1">**</ept> virtual machine restarts.
1. 关闭记事本而不保存文件。
1. After <bpt id="p1">**</bpt>SEA-SVR1<ept id="p1">**</ept> restarts, on <bpt id="p2">**</bpt>SEA-ADM1<ept id="p2">**</ept>, switch to <bpt id="p3">**</bpt>Server Manager<ept id="p3">**</ept>, and on the left side, select the <bpt id="p4">**</bpt>AD DS<ept id="p4">**</ept> node. Note that <bpt id="p1">**</bpt>SEA-SVR1<ept id="p1">**</ept> has been added as a server and that the warning notification has disappeared.

   > **注意**：可能需要选择“刷新”。

#### <a name="task-3-manage-objects-in-ad-ds"></a>任务 3：管理 AD DS 中的对象

1. 确保已连接到 SEA-ADM1 的控制台会话。
1. 切换到“Windows PowerShell (管理员)”。
1. 要在 Contoso AD DS 域中创建名为 Seattle 的组织单位 (OU)，请输入以下命令，然后按 Enter：

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle" -Path "DC=contoso,DC=com" -ProtectedFromAccidentalDeletion $true -Server SEA-DC1.contoso.com
   ```
1. 要在 Seattle OU 中为 Ty Carlson 创建用户帐户，请输入以下命令，然后按 Enter ：

   ```powershell
   New-ADUser -Name Ty -DisplayName 'Ty Carlson' -GivenName Ty -Surname Carlson -Path 'OU=Seattle,DC=contoso,DC=com'
   ```
1. 要为 Ty 的用户帐户设置密码，请输入以下命令，然后按 Enter：

   ```powershell
   Set-ADAccountPassword Ty
   ```
1. 收到输入当前密码的提示时，请按 Enter。
1. 收到输入所需密码的提示时，请输入 Pa55w.rd，然后按 Enter。
1. 收到输入确认密码的提示时，请输入 Pa55w.rd，然后按 Enter。
1. 要启用帐户，请输入以下命令，然后按 Enter：

   ```powershell
   Enable-ADAccount Ty
   ```
1. 要创建名为 SeattleBranchUsers 的域全局组，请输入以下命令，然后按 Enter：

   ```powershell
   New-ADGroup SeattleBranchUsers -Path 'OU=Seattle,DC=contoso,DC=com' -GroupScope Global -GroupCategory Security
   ```
1. 要将 Ty 用户帐户添加到新创建的组，请输入以下命令，然后按 Enter：

   ```powershell
   Add-ADGroupMember -Identity SeattleBranchUsers -Members Ty
   ```
1. 要确认用户是否位于组中，请输入以下命令，然后按 Enter：

   ```powershell
   Get-ADGroupMember -Identity SeattleBranchUsers
   ```
1. 要将用户添加到本地管理员组，请输入以下命令，然后按 Enter：

   ```powershell
   Add-LocalGroupMember -Group 'Administrators' -Member 'CONTOSO\Ty'
   ```

   > **注意**：使用 **CONTOSO\\Ty** 用户帐户登录 SEA-ADM1 时需要此信息。

**结果**：完成本练习后，应已成功在 AD DS 中创建了新的域控制器和托管对象。

## <a name="exercise-2-configuring-group-policy"></a>练习 2：配置组策略

#### <a name="task-1-create-and-edit-a-gpo"></a>任务 1：创建和编辑 GPO

1. 在 SEA-ADM1 上，从服务器管理器中选择“工具”，然后选择“组策略管理”  。
1. 如有必要，请切换到“组策略管理”窗口。
1. 在“组策略管理”控制台的导航窗格中，展开“Forest:Contoso.com”、“域”和“Contoso.com”，然后选择“组策略对象”容器    。
1. 在导航窗格中，右键单击或访问“组策略对象”容器的上下文菜单，然后选择“新建” 。
1. 在“名称”文本框中，输入“CONTOSO 标准”，然后选择“确定”  。
1. 在详细信息窗格中，右键单击或访问 CONTOSO 标准组策略对象 (GPO) 上下文菜单，然后选择“编辑” 。
1. 在“组策略管理编辑器”窗口中，在导航窗格中依次展开“用户配置”、“策略”“管理模板”，然后选择“系统”    。
1. 双击“阻止访问注册表编辑工具”策略设置或选择该设置，然后按 Enter。
1. 在“阻止访问注册表编辑工具”对话框中，选择“启用”，然后选择“确定”  。
1. 在导航窗格中，依次展开“用户配置”、“策略”、“管理模板”、“控制面板”，然后选择“个性化”     。
1. 在详细信息窗格中，双击或选择“屏幕保护程序超时”策略设置，然后按 Enter。
1. 在上一个命令的输出中，搜索“Active Directory 域服务”复选框，然后确认它已被选中。 
1. 双击或选择“对屏幕保护程序使用密码保护”策略设置，然后按 Enter。
1. 在“对屏幕保护程序使用密码保护”对话框中，选择“启用”，然后选择“确定”  。
1. 关闭“组策略管理编辑器”窗口。

#### <a name="task-2-link-the-gpo"></a>任务 2：链接 GPO

1. 在“组策略管理”窗口中的导航窗格中，右键单击或访问 `Contoso.com` 域的上下文菜单，然后选择“链接现有 GPO” 。
1. 在“选择 GPO”对话框中，选择“CONTOSO 标准”，然后选择“确定”  。

#### <a name="task-3-review-the-effects-of-the-gpos-settings"></a>任务 3：查看 GPO 设置的效果

1. 在 SEA-ADM1 上，在任务栏上的搜索框中，输入“控制面板” 。 
1. 在“最佳匹配”列表中，选择“控制面板” 。
1. 选择“系统和安全”，然后选择“允许应用通过 Windows 防火墙” 。
1. 在“允许的应用和功能”列表中，找到“远程事件日志管理”条目，选中“域”列中的复选框，然后选择“确定”   。 
1. 注销，然后以 CONTOSO\\Ty 身份使用密码 Pa55w.rd 登录 。
1. 在任务栏上的搜索框中，输入“控制面板”。
1. 在“最佳匹配”列表中，选择“控制面板” 。
1. 然后，搜索“远程服务器管理工具”。
1. 请注意其下方的“角色管理工具”节点，然后验证“AD DS 和 AD LDS 工具”节点是否也已被选中 。

   > **注意**：如果“恢复时显示登录屏幕”选项未选中且不为灰显，请打开命令提示符，运行 `gpupdate /force`，然后重复上述步骤。

1. 右键单击或访问“开始”的上下文菜单，然后选择“运行” 。
1. In the <bpt id="p1">**</bpt>Run<ept id="p1">**</ept> dialog box, in the <bpt id="p2">**</bpt>Open<ept id="p2">**</ept> text box, enter <bpt id="p3">**</bpt>regedit<ept id="p3">**</ept>, and then select <bpt id="p4">**</bpt>OK<ept id="p4">**</ept>. Note the error message stating <bpt id="p1">**</bpt>Registry editing has been disabled by your administrator<ept id="p1">**</ept>.
1. 在“注册表编辑器”对话框中，选择“确定” 。
1. 注销，然后以 **CONTOSO\\Administrator 身份使用密码 Pa55w.rd 再次登录** 。

#### <a name="task-4-create-and-link-the-required-gpos"></a>任务 4：创建并链接所需的 GPO

1. 在 SEA-ADM1 上，从服务器管理器中选择“工具”，然后选择“组策略管理”  。
1. 如有必要，请切换到“组策略管理”窗口。
1. 在“组策略管理”控制台的导航窗格中，展开“Forest: Contoso.com”、“域”和“Contoso.com”，然后选择“Seattle”    。
1. 右键单击或访问 Seattle 组织单位 (OU) 的上下文菜单，然后选择“在此域中创建 GPO 并在此处链接” 。
1. 在“新建 GPO”对话框中的“名称”文本框中，输入“Seattle Application Override”，然后选择“确定”   。
1. 在详细信息窗格中，右键单击或访问 Seattle Application Override GPO 的上下文菜单，然后选择“编辑” 。
1. 在“控制台”树中，依次展开“用户配置”、“策略”、“管理模板”、“控制面板”，然后选择“个性化”     。
1. 双击或选择“屏幕保护程序超时”策略设置，然后按 Enter。
1. 选择“禁用”，然后选择“确定” 。
1. 关闭“组策略管理编辑器”窗口。

#### <a name="task-5-verify-the-order-of-precedence"></a>任务 5：验证优先顺序

1. 返回到“组策略管理控制台”树中，确保选中“Seattle”组织单位 。
1. 选择“组策略继承”选项卡并查看其内容。

   > **注意**：在“AD DS 和 AD LDS 工具”节点下，仅安装了 Windows PowerShell 的 Active Directory 模块，未安装图形工具（如 Active Directory 管理中心） 。

#### <a name="task-6-configure-the-scope-of-a-gpo-with-security-filtering"></a>任务 6：使用安全筛选配置 GPO 的范围

1. 在 SEA-ADM1 上，在“组策略管理”控制台的“导航”窗格中，如有必要，展开“Seattle”组织单位，然后选择其下方的“Seattle Application Override”GPO    。
1. 在“组策略管理控制台”对话框中，查看以下消息：“你选择了一个指向组策略对象 (GPO) 的链接。除了对链接属性所做的更改，你在此处所做的更改对 GPO 是全局的，并且会影响在其中链接此 GPO 的所有其他位置” 。
1. 选中“不再显示此消息”选框，然后选择“确定” 。
1. 查看“安全筛选”部分，请注意，GPO 默认情况下适用于经过身份验证的用户。
1. 在“安全筛选”部分，选择“经过身份验证的用户”，然后选择“删除”  。
1. 在“组策略管理”对话框中，选择“确定”，查看“组策略管理”警告，然后再次选择“确定”   。

   > 如果要集中管理服务器，则通常不需要在每个服务器上使用这些工具。

1. 在详细信息窗格中，选择“添加”。
1. 在“选择用户、计算机或组”对话框的“输入要选择的对象名称(示例):”文本中，输入 SeattleBranchUsers，然后选择“确定”   。
1. 在详细信息窗格中的“安全筛选”下，选“添加” 。
1. 在“选择用户、计算机或组”对话框中，选择“对象类型” 。
1. 在“对象类型”对话框中，选中“计算机”复选框，然后选择“确定”  。
1. 在“选择用户、计算机或组”对话框的“输入要选择的对象名称(示例):”文本中，输入 SEA-ADM1，然后选择“确定”   。

#### <a name="task-7-verify-the-application-of-settings"></a>任务 7：验证应用程序的设置

1. 在导航窗格的“组策略管理”中，选择 “组策略建模” 。
1. 右键单击或访问“组策略建模”的上下文菜单，然后选择“组策略建模向导” 。
1. 在“组策略建模向导”中，选择“下一步” 。
1. 在“域控制器选择”页上，接受默认设置，然后选择“下一步” 。
1. 在“用户和计算机选择”页面上的“用户信息”部分，选择“用户”，然后在 “用户”文本框中，输入 CONTOSO\Ty，或使用“浏览”命令按钮查找 Ty 用户帐户      。
1. 在“用户和计算机选择”页面上的“计算机信息”部分，选择“计算机”，然后在“计算机”文本框中输入 CONTOSO\SEA-ADM1    。
1. 在“用户和计算机选择”页上，选择“下一步” 。
1. 在“高级模拟选项”页面上，接受默认设置，然后选择“下一步” 。
1. 在“备用 Active Directory 路径”上，记下用户和计算机位置，然后选择“下一步” 。
1. 在“用户安全组”页面上，验证组列表是否包含 CONTOSO\\SeattleBranchUsers，然后选择“下一步”  。
1. 在“计算机安全组”页上，选择“下一步” 。
1. 在“面向用户的 WMI 筛选器”页上，接受默认设置，然后选择“下一步” 。
1. 在“面向计算机的 WMI 筛选器”页上，接受默认设置，然后选择“下一步” 。
1. 在“选择摘要”页上，选择“下一步” 。
1. 在出现提示时选择“完成”。
1. 在详细信息窗格中，选择“详细信息”选项卡，然后选择“全部显示” 。
1. 如果要安装 AD DS 工具，则必须通过使用 RSAT-ADDS 命令运行 Add-WindowsFeature cmdlet 进行指定 。
1. 关闭“组策略管理”控制台。

**结果**：完成此练习后，应已成功创建和配置了 GPO。
