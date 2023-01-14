---
lab:
  title: 实验室：实现标识服务和组策略
  module: 'Module 1: Identity services in Windows Server'
---

# <a name="lab-implementing-identity-services-and-group-policy"></a>实验室：实现标识服务和组策略

## <a name="scenario"></a>场景

你在 Contoso Ltd. 担任管理员。公司正在多个新地方扩展其业务 Active Directory 域服务 (AD DS) 管理团队当前正在评估 Windows Server 中可用于非交互式远程域控制器部署的方法。 该团队还在寻找一种方法来自动执行某些 AD DS 管理任务。 此外，该团队希望建立基于组策略对象 (GPO) 的配置管理。

## <a name="objectives"></a>                **注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Implementing%20identity%20services%20and%20Group%20Policy)** ，让你能以自己的节奏点击浏览实验室。

你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。

- 目标
- 完成本实验室后，你将能够：

## <a name="estimated-time-45-minutes"></a>在 Server Core 上部署新的域控制器。

## <a name="lab-setup"></a>配置组策略。

估计时间：45 分钟 实验室设置

> 虚拟机：AZ-800T00A-SEA-DC1、AZ-800T00A-ADM1 和 AZ-800T00A-SEA-SVR1 必须处于运行状态  。 

1. 其他 VM 可以运行，但此实验室不需要这些 VM。
1. 注意：AZ-800T00A-SEA-DC1、AZ-800T00A-ADM1 和 AZ-800T00A-SEA-SVR1 虚拟机承载 SEA-DC1、SEA-SVR1 和 SEA-ADM1 的安装      。

   - 选择“SEA-ADM1”。
   - 使用以下凭据登录：
   - 用户名：Administrator

## <a name="exercise-1-deploying-a-new-domain-controller-on-server-core"></a>密码：Pa55w.rd

### <a name="scenario"></a>域名：CONTOSO

练习 1：在 Server Core 上部署新的域控制器 场景

作为业务重构的一部分，Contoso 想要在远程站点中部署新的域控制器，同时尽量减少 IT 在远程位置的参与。

1. 需要使用 DC 部署来部署新的域控制器。
1. 此练习的主要任务如下：

#### <a name="task-1-deploy-ad-ds-on-a-new-windows-server-core-server"></a>在新的 Windows Server Core 服务器上部署 AD DS。

1. 使用 GUI 工具和 Windows PowerShell 管理 AD DS 对象。
1. 任务 1：在新的 Windows Server Core 服务器上部署 AD DS
1. 切换到 SEA-ADM1，并从“服务器管理器”打开 Windows PowerShell 。
1. 在 Windows PowerShell 中使用 Install-WindowsFeature cmdlet 在 SEA-SVR1 上安装 AD DS 角色 。 使用 Get-WindowsFeature cmdlet 验证安装。

> 确保选中“Active Directory 域服务”、“远程服务器管理工具”和“角色管理工具”复选框  。 对于“AD DS”和“AD LDS 工具”节点，应仅安装 Windows PowerShell 的 Active Directory 模块，而不应安装图形工具（如 Active Directory 管理中心）  。

> 注意：如果要集中管理服务器，则通常不需要在每个服务器上使用 GUI 工具。 如果要安装 AD DS 工具，则需要通过使用 RSAT-ADDS 命令运行 Add-WindowsFeature cmdlet 进行指定 。

#### <a name="task-2-prepare-the-ad-ds-installation-and-promote-a-remote-server"></a>注意：你可能需要等待安装过程完成后，才能验证 AD DS 角色是否已安装。

1. 如果未观察到 Get-WindowsFeature 命令的预期结果，可在几分钟后重试。
1. 任务 2：准备安装 AD DS 并提升远程服务器

   - 在 SEA-ADM1 上，从“服务器管理器”的“所有服务器”节点上，将 SEA-SVR1 添加为托管服务器   。
   - 在 SEA-ADM1 上，从“服务器管理器”中，使用以下设置将 SEA-SVR1 配置为 AD DS 域控制器  ：
   - 类型：现有域的附加域控制器
   - 域：`contoso.com`
   - 凭据：密码为 Pa55w.rd 的 CONTOSO\\Administrator 

1. 目录服务还原模式 (DSRM) 密码：Pa55w.rd
1. 请勿删除针对 DNS 和全局编录的选项

   - 在“查看选项”页上，选择“查看脚本” 。
   - 在记事本中，编辑生成的 Windows PowerShell 脚本，如下所示：
   - 删除以数字符号 (#) 开头的注释行。
   - 删除 Import-Module 行。

1. 删除每行末尾的重音符号 (`)。
1. 删除换行符。
1. 现在，Install-ADDSDomainController 命令以及所有参数都位于一行中，请复制该命令。

   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 { }
   ```
1. 切换到 Windows PowerShell，然后在命令提示符处输入以下命令： 将复制的命令粘贴在大括号 ({ }) 之间，并运行生成的命令以开始安装。

   ```powershell
   Invoke-Command –ComputerName SEA-SVR1 {Install-ADDSDomainController -NoGlobalCatalog:\$false -CreateDnsDelegation:\$false -Credential (Get-Credential) -CriticalReplicationOnly:\$false -DatabasePath "C:\Windows\NTDS" -DomainName "Contoso.com" -InstallDns:\$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:\$false -SiteName "Default-First-Site-Name" -SysvolPath "C:\Windows\SYSVOL" -Force:\$true}
   ```

1. 完整的命令应采用以下格式：

   - 提供以下凭据：
   - 用户名：CONTOSO\\Administrator

1. 密码：Pa55w.rd
1. 将“SafeModeAdministratorPassword”设置为“Pa55w.rd” 。 重启 SEA-SVR1 后，在 SEA-ADM1 上切换到“服务器管理器”，然后选择“AD DS”节点   。 请注意，SEA-SVR1 已添加为域控制器，警告通知已消失。

#### <a name="task-3-manage-objects-in-ad-ds"></a>可能必须选择“刷新”。

1. 任务 3：管理 AD DS 中的对象
1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台 。

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle" -Path "DC=contoso,DC=com" -ProtectedFromAccidentalDeletion $true -Server SEA-DC1.contoso.com
   ```
1. 若要创建名为“Seattle”的组织单位 (OU)，请在 Windows PowerShell 控制台中运行以下命令 ：

   ```powershell
   New-ADUser -Name Ty -DisplayName 'Ty Carlson' -GivenName Ty -Surname Carlson -Path 'OU=Seattle,DC=contoso,DC=com'
   ```
1. 若要在 Seattle OU 中为 Ty Carlson 创建用户帐户，请运行以下命令 ：

   ```powershell
   Set-ADAccountPassword Ty
   ```

> 若要将用户的密码设置为“Pa55w.rd”，请运行以下命令：

1. 注意：当前密码为空。

   ```powershell
   Enable-ADAccount Ty
   ```
1. 若要启用用户帐户，请运行以下命令：

   ```powershell
   New-ADGroup SeattleBranchUsers -Path 'OU=Seattle,DC=contoso,DC=com' -GroupScope Global -GroupCategory Security
   ```
1. 若要创建名为 SeattleBranchUsers 的域全局组，请运行以下命令：

   ```powershell
   Add-ADGroupMember -Identity SeattleBranchUsers -Members Ty
   ```
1. 要将 Ty 用户帐户添加到新创建的组，请运行以下命令：

   ```powershell
   Get-ADGroupMember -Identity SeattleBranchUsers
   ```
1. 若要确认用户是否在该组中，请运行以下命令：

   ```powershell
   Add-LocalGroupMember -Group 'Administrators' -Member 'CONTOSO\Ty'
   ```

   > 若要将用户添加到本地管理员组，请运行以下命令：

### <a name="results"></a>**注意**:使用 **CONTOSO\\Ty** 用户帐户登录 SEA-ADM1 时必需此信息

结果

## <a name="exercise-2-configuring-group-policy"></a>完成本练习后，应已成功在 AD DS 中创建了新的域控制器和托管对象。

### <a name="scenario"></a>练习 2：配置组策略

场景

作为组策略实现的一部分，你想要为 Office 应用导入自定义管理模板，并配置设置。

1. 此练习的主要任务如下：
1. 创建并编辑 GPO 设置。

#### <a name="task-1-create-and-edit-a-gpo"></a>应用并验证客户端计算机上的设置。

1. 任务 1：创建并编辑 GPO
1. 在 SEA-ADM1 上，从“服务器管理器”中，打开“组策略管理”控制台  。
1. 在“组策略对象”容器中创建一个名为“Contoso Standards”的 GPO 。
1. 在组策略管理编辑器中打开“Contoso Standards”GPO，然后浏览到 User Configuration\Policies\Administrative Templates\System 。
1. 启用“阻止访问注册表编辑工具”策略设置。
1. 浏览到 User Configuration\Policies\Administrative Templates\Control Panel\Personalization 文件夹，然后将“屏幕保护”超时策略配置为“600”秒  。

#### <a name="task-2-link-the-gpo"></a>启用“对屏幕保护使用密码保护”策略设置，然后关闭“组策略管理编辑器”窗口 。

  - 任务 2：链接 GPO

#### <a name="task-3-review-the-effects-of-the-gpos-settings"></a>将“Contoso Standards”GPO 链接到 `contoso.com` 域。

1. 任务 3：查看 GPO 设置的效果
1. 在 SEA-ADM1 上，打开“控制面板” 。 
1. 使用“Windows Defender 防火墙”接口，启用“远程事件日志管理”域流量 。
1. 注销，然后以 CONTOSO\\Ty 身份使用密码 Pa55w.rd 登录 。 尝试更改屏幕保护等待时间和恢复设置。
1. 确认组策略会阻止这些操作。 尝试运行注册表编辑器。 
1. 确认组策略会阻止此操作。

#### <a name="task-4-create-and-link-the-required-gpos"></a>注销，然后以 **CONTOSO\\Administrator 身份使用密码 Pa55w.rd 登录** 。

1. 任务 4：创建并链接所需的 GPO
1. 在 SEA-ADM1 上，在“组策略管理”控制台中，创建名为“Seattle Application Override”的新 GPO，该 GPO 链接到 Seattle OU   。

#### <a name="task-5-verify-the-order-of-precedence"></a>将“屏幕保护超时”策略设置配置为禁用，然后关闭“组策略管理编辑器”窗口 。

1. 任务 5：验证优先顺序
1. 在 SEA-ADM1 上，从“服务器管理器”中，打开“组策略管理”控制台  。
1. 在“组策略管理控制台”树中，选择“Seattle”OU 。

   > 选择“组策略继承”选项卡并查看其内容。 注意：Seattle Application Override GPO 的优先级高于 CONTOSO Standards GPO。 刚刚在 Seattle Application Override GPO 中配置的屏幕保护程序超时策略设置将在设置 CONTOSO Standards GPO 之后应用。 因此，新设置将覆盖 CONTOSO Standards GPO 设置。

#### <a name="task-6-configure-the-scope-of-a-gpo-with-security-filtering"></a>对于 Seattle Application Override GPO 范围内的用户，将禁用屏幕保护程序超时。

1. 任务 6：使用安全筛选配置 GPO 的范围 在 SEA-ADM1 上，在“组策略管理”控制台中，选择“Seattle Application Override”GPO  。
1. 注意，在“安全筛选”部分，GPO 默认应用于所有经过身份验证的用户。

#### <a name="task-7-verify-the-application-of-settings"></a>在“安全筛选”部分中，首先删除“经过身份验证的用户”，然后添加“SeattleBranchUsers”组和“SEA-ADM1”计算机帐户   。

1. 任务 7：验证应用程序的设置
1. 在“组策略管理”的导航窗格中，选择“组策略建模”。
1. 启动“组策略建模向导”。
1. 将目标用户和计算机分别设置为 CONTOSO\Ty 用户帐户和 CONTOSO\SEA-ADM1 计算机 。
1. 逐步完成向导的剩余页面，在不修改默认设置的情况下查看这些设置，并完成向导，这将生成包含其结果的报表。
1. 创建报表后，在“详细信息”窗格中，选择“详细信息”选项卡，然后选择“全部显示” 。 在报表中向下滚动，直到找到“用户详细信息”部分，然后找到“控制面板/个性化”部分 。
1. 你应该注意到“屏幕保护超时”设置获取自 Seattle Application Override GPO。

### <a name="results"></a>关闭“组策略管理”控制台。

结果
