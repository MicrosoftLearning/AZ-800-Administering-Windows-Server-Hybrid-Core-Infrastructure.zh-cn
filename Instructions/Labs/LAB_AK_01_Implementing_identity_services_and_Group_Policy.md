---
lab:
  title: 实验室：实现标识服务和组策略
  type: Answer Key
  module: 'Module 1: Identity services in Windows Server'
ms.openlocfilehash: d68a50f3573f2817f4f83bc25cd69aba82e1d455
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906931"
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
1. 在上一个命令的输出中，搜索“Active Directory 域服务”复选框，然后确认它已被选中。 然后，搜索“远程服务器管理工具”。 请注意其下方的“角色管理工具”节点，然后验证“AD DS 和 AD LDS 工具”节点是否也已被选中 。

   > **注意**：在“AD DS 和 AD LDS 工具”节点下，仅安装了 Windows PowerShell 的 Active Directory 模块，未安装图形工具（如 Active Directory 管理中心） 。 如果要集中管理服务器，则通常不需要在每个服务器上使用这些工具。 如果要安装 AD DS 工具，则必须通过使用 RSAT-ADDS 命令运行 Add-WindowsFeature cmdlet 进行指定 。

   > **注意**：安装过程完成后，可能需要稍等片刻，然后才能验证 AD DS 角色是否已安装。 如果未观察到 Get-WindowsFeature 命令的预期结果，可在几分钟后重试。

#### <a name="task-2-prepare-the-ad-ds-installation-and-promote-a-remote-server"></a>任务 2：准备安装 AD DS 并提升远程服务器

1.  在 SEA-ADM1 的“开始”菜单上，选择“服务器管理器”，然后在“服务器管理器”视图中选择“所有服务器”视图    。
1. 在“管理”菜单上，选择“添加服务器” 。
1. 在“添加服务器”对话框中，保留默认设置，然后选择“立即查找” 。
1. 在服务器的 Active Directory 列表中，选择 SEA-SVR1，选择箭头将其添加到“选定项”列表，然后选择“确定”   。
1. 在 SEA-ADM1 上，确保已在 SEA-SRV1 上安装 AD DS 角色，并且服务器已添加到服务器管理器  。 然后选择“通知”标志符号。
1. 记下 SEA-SVR1 的部署后配置，然后选择“将此服务器提升到域控制器”链接 。
1. 在“Active Directory 域服务配置向导”的“部署配置”页上，在“选择部署操作”下，确认已选中“向现有域添加域控制器”   。
1. 确保指定了 `Contoso.com` 域，然后在“提供凭据以执行此操作”部分，选择“更改” 。
1. 在“部署操作的凭据”对话框的“用户名”框中，输入“CONTOSO\\Administrator”，在“密码”框中，输入“Pa55w.rd”    。
1. 选择“确定”  ，然后选择“下一步”  。
1. 在“域控制器选项”页上，确保已选中“域名系统(DNS)服务器”和“全局编录(GC)”复选框  。 确保清除“只读域控制器(RODC)”复选框。
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

1. 现在，Install-ADDSDomainController 命令以及所有参数都位于一行中。 将光标置于该行的前面，然后在“编辑”菜单上，选择“全选”以选择整行 。 在菜单上，选择“编辑”，然后选择“复制” 。
1. 切换到 Active Directory 域服务配置向导，然后选择“取消” 。
1. 当系统提示确认时，选择“是”以取消该向导。
1. 在 Windows PowerShell 命令提示符下，输入以下命令：

   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 { }
   ```
1. 将光标置于大括号 ({ }) 之间，然后粘贴剪贴板中复制的脚本行的内容。 完整的命令应采用以下格式：
    
   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 {Install-ADDSDomainController -NoGlobalCatalog:\$false -CreateDnsDelegation:\$false -Credential (Get-Credential) -CriticalReplicationOnly:\$false -DatabasePath "C:\Windows\NTDS" -DomainName "Contoso.com" -InstallDns:\$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:\$false -SiteName "Default-First-Site-Name" -SysvolPath "C:\Windows\SYSVOL" -Force:\$true}
   ```
1. 要调用命令，请按 Enter。
1. 在“Windows PowerShell 凭据请求”对话框中，在“用户名”框中输入 CONTOSO\\Administrator，在“密码”框中输入 Pa55w.rd，然后选择“确定”     。
1. 当系统提示输入密码时，在 SafeModeAdministratorPassword 文本框中，输入 Pa55w.rd，然后按 Enter 。
1. 当系统提示进行确认时，在 Confirm SafeModeAdministratorPassword 文本框中，输入 Pa55w.rd，然后按 Enter 。
1. 等到命令运行并返回“状态成功”消息。 重启 SEA-SVR1 虚拟机。
1. 关闭记事本而不保存文件。
1. 重启 SEA-SVR1 后，在 SEA-ADM1 上切换到“服务器管理器”，在左侧选择 AD DS 节点   。 请注意，SEA-SVR1 已添加为服务器，警告通知已消失。

   > **注意**：可能需要选择“刷新”。

### <a name="task-3-manage-objects-in-ad-ds"></a>任务 3：管理 AD DS 中的对象

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
1. 在“屏幕保护程序超时”对话框中，选择“启用” 。 在“秒”文本框中，输入 600，然后选择“确定”  。 
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
1. 在“控制面板”的搜索框中，输入“屏幕保护程序”，然后选择“更改屏幕保护程序” 。 （可能需要几分钟时间才能显示选项。）
1. 在“屏幕保护程序设置”对话框中，请注意，“等待”选项将灰显 。 不能更改超时。请注意，“恢复时显示登录屏幕”选项处于选中状态并灰显，并且无法更改设置。

   > **注意**：如果“恢复时显示登录屏幕”选项未选中且不为灰显，请打开命令提示符，运行 `gpupdate /force`，然后重复上述步骤。

1. 右键单击或访问“开始”的上下文菜单，然后选择“运行” 。
1. 在“运行”对话框中的“打开”文本框中，输入 Regedit，然后选择“确定”   。 请注意指出“注册表编辑已被管理员禁用”的错误消息。
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

   > **注意**：Seattle Application Override GPO 的优先级高于 CONTOSO 标准 GPO。 刚刚在 Seattle Application Override GPO 中配置的屏幕保护程序超时策略设置将在设置 CONTOSO 标准 GPO 之后应用。 因此，新设置将覆盖 CONTOSO 标准 GPO 设置。 对于 Seattle Application Override GPO 范围内的用户，将禁用屏幕保护程序超时。

#### <a name="task-6-configure-the-scope-of-a-gpo-with-security-filtering"></a>任务 6：使用安全筛选配置 GPO 的范围

1. 在 SEA-ADM1 上，在“组策略管理”控制台的“导航”窗格中，如有必要，展开“Seattle”组织单位，然后选择其下方的“Seattle Application Override”GPO    。
1. 在“组策略管理控制台”对话框中，查看以下消息：“你选择了一个指向组策略对象 (GPO) 的链接。除了对链接属性所做的更改，你在此处所做的更改对 GPO 是全局的，并且会影响在其中链接此 GPO 的所有其他位置” 。
1. 选中“不再显示此消息”选框，然后选择“确定” 。
1. 查看“安全筛选”部分，请注意，GPO 默认情况下适用于经过身份验证的用户。
1. 在“安全筛选”部分，选择“经过身份验证的用户”，然后选择“删除”  。
1. 在“组策略管理”对话框中，选择“确定”，查看“组策略管理”警告，然后再次选择“确定”   。

   > **注意**：组策略要求每个计算机帐户都有权从域控制器读取 GPO 数据，才能成功地应用用户 GPO 设置。 修改 GPO 的安全筛选设置时，应考虑到这一点。

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
1. 在报表中向下滚动，直到找到“用户详细信息”部分，然后找到“控制面板/个性化”部分 。 请注意，禁用了 “幕保护程序超时”设置，并且入选的 GPO 设置为 Seattle Application Override GPO。
1. 关闭“组策略管理”控制台。

**结果**：完成此练习后，应已成功创建和配置了 GPO。
