---
lab:
  title: 实验室：实现 AD DS 与 Azure AD 之间的集成
  module: 'Module 2: Implementing Identity in Hybrid Scenarios'
---

# <a name="lab-implementing-integration-between-ad-ds-and-azure-ad"></a>实验室：实现 AD DS 与 Azure AD 之间的集成

## <a name="scenario"></a>场景

为解决因使用 Microsoft Azure Active Directory (Azure AD) 针对 Azure 资源的访问进行身份验证和授权所产生的管理和监视开销的问题，你决定测试本地 Active Directory 域服务 (AD DS) 与 Azure AD 之间的集成，来验证是否可以通过混合使用本地资源和云资源来解决有关管理多个用户帐户的业务问题。

此外，你还需要确保自己的方法能解决信息安全团队的问题，并保留应用于 Active Directory 用户的现有控件，例如登录时间和密码策略。 最后，你需要确定可实现以下目的的 Azure AD 集成功能：能进一步增强本地 Active Directory 安全性，且管理开销尽可能小，包括对 Windows Server Active Directory 的 Azure AD 密码保护、通过密码写回实现的自助式密码重置 (SSPR)。

你的目标是在本地 AD DS 和 Azure AD 之间实现直通身份验证。

                **注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Implementing%20integration%20between%20AD%20DS%20and%20Azure%20AD)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## <a name="objectives"></a>目标

完成本实验后，你将能够：

- 做好 Azure AD 的相关准备工作，以便与本地 AD DS 集成，包括添加和验证自定义域。
- 做好本地 AD DS 的相关准备工作，以便与 Azure AD 集成，包括运行 IdFix DirSync 错误修正工具。
- 安装并配置 Azure AD Connect。
- 通过测试同步过程来验证 AD DS 与 Azure AD 之间的集成。
- 实现 Active Directory 中的 Azure AD 集成功能，包括对 Windows Server Active Directory 的 Azure AD 密码保护和通过密码写回实现的 SSPR。

## <a name="estimated-time-60-minutes"></a>预计时间：60 分钟

## <a name="lab-setup"></a>实验室设置

虚拟机：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1 和 AZ-800T00A-ADM1 必须处于运行状态  。 其他 VM 可以运行，但本实验室不需要这些 VM。 

> 注意：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1 和 AZ-800T00A-ADM1 虚拟机承载 SEA-DC1、SEA-SVR1 和 SEA-ADM1 的安装      

1. 选择“SEA-ADM1”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境和 Azure AD 租户。 在开始实验之前，请确保拥有 Azure AD 租户，且有一个具有该租户中全局管理员角色的用户帐户。

## <a name="exercise-1-preparing-azure-ad-for-ad-ds-integration"></a>练习 1：为与 AD DS 集成准备 Azure AD

### <a name="scenario"></a>场景

需要确保 Azure AD 环境已准备好与本地 AD DS 集成。 因此需要创建并验证自定义 Azure AD 域名和具有全局管理员角色的帐户。

此练习的主要任务是：

1. 在 Azure 中创建自定义域名。
1. 创建具有全局管理员角色的用户。
1. 更改具有全局管理员角色的用户的密码。

#### <a name="task-1-create-a-custom-domain-name-in-azure"></a>任务 1：在 Azure 中创建自定义域名

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 Azure 门户。
1. 使用讲师提供的凭据登录到 Azure 门户。
1. 在 Azure 门户中，浏览到“Azure Active Directory”。
1. 在“Azure Active Directory”页面上，选择“自定义域名”，然后添加 `contoso.com` 。
1. 查看要用于验证域的 DNS 记录类型，然后关闭窗格而不验证域名。

   > 注意：虽然一般来说要使用 DNS 记录来验证域，但此实验不需要使用验证的域。

#### <a name="task-2-create-a-user-with-the-global-administrator-role"></a>任务 2：创建具有全局管理员角色的用户

1. 在 SEA-ADM1 上，在显示“Azure Active Directory”页面的 Microsoft Edge 窗口中，浏览到“所有用户”页面，并创建具有以下属性的用户帐户  ： 

   - 用户名：admin

   > 注意：确保“用户名”的域名下拉菜单列出了以 `onmicrosoft.com` 结尾的默认域名 。

   - 名称：admin
   - 角色：全局管理员
   - 密码：使用自动生成的密码

   > 注意：使用“显示密码”选项来显示自动生成的密码，并将其记录下来，以便稍后在本实验中使用 。

#### <a name="task-3-change-the-password-for-the-user-with-the-global-administrator-role"></a>任务 3：更改具有全局管理员角色的用户的密码

1. 从 Azure 门户注销，然后用在上一个任务中创建的用户帐户登录。 
1. 出现提示时，请更改密码。

   > 注意：记录所使用的复杂密码，因为稍后将在本实验中使用。

## <a name="exercise-2-preparing-on-premises-ad-ds-for-azure-ad-integration"></a>练习 2：为 Azure AD 集成准备本地 AD DS

### <a name="scenario"></a>场景

需要确保现有 Active Directory 环境已为 Azure AD 集成做好准备。 因此，将运行 IdFix 工具，然后确保 Active Directory 用户的 UPN 与 Azure AD 租户的自定义域名匹配。

此练习的主要任务是：

1. 安装 IdFix。
1. 运行 IdFix。

#### <a name="task-1-install-idfix"></a>任务 1：安装 IdFix

1. 在 SEA-ADM1 上，打开 Microsoft Edge，然后转到 https://github.com/microsoft/idfix 。
1. 在“Github”页面上的“ClickOnce 启动”下，选择“启动”  。
1. 在“IdFix 隐私声明”对话框中，查看免责声明，然后选择“确定” 。

#### <a name="task-2-run-idfix"></a>任务 2：运行 IdFix

1. 在“IdFix”窗口中，选择“查询” 。
1. 查看本地 Active Directory 中的对象列表，并查看 ERROR 和 ATTRIBUTE 列 。 在这种情况下，“ContosoAdmin”的displayName 的值为空，UPDATE 列中会显示工具建议的新值  。
1. 在“IdFix”窗口中的 ACTION 下拉菜单中，选择“编辑”，然后选择“应用”以自动实现建议的更改   。
1. 在“应用挂起”对话框中，选择“是”，然后关闭 IdFix 工具 。

## <a name="exercise-3-downloading-installing-and-configuring-azure-ad-connect"></a>练习 3：下载、安装和配置 Azure AD Connect

### <a name="scenario"></a>场景

练习方案：现在已可以实现集成，方法是下载 Azure AD Connect、在 SEA-ADM1 上安装并将其设置配置为与集成目标相匹配。

此练习的主要任务是：

1. 安装并配置 Azure AD Connect。

#### <a name="task-1-install-and-configure-azure-ad-connect"></a>任务 1：安装和配置 Azure AD Connect

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，从“Azure Active Directory”页面浏览到“Azure AD Connect”页面  。
1. 在”Microsoft Azure Active Directory Connect”页面中，选择“下载” 。
1. 下载 Azure AD Connect 安装二进制文件并开始安装。
1. 在“Microsoft Azure Active Directory Connect”页面上，选中“我同意许可条款和隐私通知”复选框，然后选择“继续”  。
1. 在“Express 设置”页面上，选择“使用快速设置” 。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码。
1. 在“连接 AD DS”页上，输入以下凭据：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在“Azure AD 登录配置”页面上，验证添加的新域是否在 Active Directory UPN 后缀列表中。

   > 注意：提供的域名不必是已验证的域。 虽然通常会在安装 Azure AD Connect 之前验证某个域，但此实验不需要执行该验证步骤。

1. 选择“继续操作且不将所有 UPN 后缀与已验证的域进行匹配”复选框。
1. 到达“准备好配置”页面后，查看操作列表，然后开始安装。

## <a name="exercise-4-verifying-integration-between-ad-ds-and-azure-ad"></a>练习 4：验证 AD DS 与 Azure AD 之间的集成

### <a name="scenario"></a>场景

现已安装并配置 Azure AD Connect，接下来需要验证其同步机制。 你计划对本地用户帐户进行更改，这将触发同步机制。 然后要验证是否已将更改复制到相应的 Azure AD 用户对象。

此练习的主要任务是：

1. 验证 Azure 门户中的同步。
1. 验证同步服务管理器中的同步。
1. 更新 Active Directory 中的用户帐户。
1. 在 Active Directory 中创建用户帐户。
1. 将更改同步到 Azure AD。
1. 在 Azure AD 中验证更改。

#### <a name="task-1-verify-synchronization-in-the-azure-portal"></a>任务 1：验证 Azure 门户中的同步

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口。 
1. 刷新“Azure AD Connect”页面，查看“从 Active Directory 预配”下的信息 。
1. 从“Azure Active Directory”页面浏览到“用户”页面 。
1. 观察从 Active Directory 同步的用户的列表。

   > 注意：目录同步启动后，Active Directory 对象会在最多 15 分钟后显示在 Azure AD 门户中。

1. 从“用户”页面浏览到“组”页面 。
1. 查看从 Active Directory 同步的组的列表。

#### <a name="task-2-verify-synchronization-in-the-synchronization-service-manager"></a>任务 2：验证同步服务管理器中的同步

1. 在 SEA-ADM1 上的“开始”菜单上，展开“Azure AD Connect”，然后选择“同步服务”   。
1. 在“同步服务管理器”窗口中的“操作”选项卡下，查看为同步 Active Directory 对象执行的任务 。
1. 选择“连接器”选项卡，并记下这两个连接器。

   > 注意：一个连接器用于 AD DS，另一个连接器用于 Azure AD 租户。 

1. 关闭“Synchronization Service Manager”窗口。

#### <a name="task-3-update-a-user-account-in-active-directory"></a>任务 3：更新 Active Directory 中的用户帐户

1. 在 SEA-ADM1 上，在“服务器管理器”中，打开“Active Directory 用户和计算机”  。
1. 在“Active Directory 用户和计算机”中，展开“销售”组织单位 (OU)，然后打开“Sumesh Rajan”的属性  。
1. 在用户的属性中，选择“组织”选项卡。
1. 在“职务”文本框中，输入“经理”，然后选择“确定”  。

#### <a name="task-4-create-a-user-account-in-active-directory"></a>任务 4：在 Active Directory 中创建用户帐户

- 在“销售”OU 中创建以下用户：
   - 名字：Jordan
   - 姓氏：Mitchell
   - 用户登录名：Jordan
   - 密码：Pa55w.rd

#### <a name="task-5-sync-changes-to-azure-ad"></a>任务 5：将更改同步到 Azure AD

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 。
1. 在 Windows PowerShell 控制台中，运行以下命令来触发同步：

   ```powershell
   Start-ADSyncSyncCycle
   ```

   > 注意：同步周期启动后，Active Directory 对象会在最多 15 分钟后显示在 Azure AD 门户中。

#### <a name="task-6-verify-changes-in-azure-ad"></a>任务 6：验证 Azure AD 中的更改

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后返回“Azure Active Directory”页面 。
1. 从“Azure Active Directory”页面浏览到“用户”页面 。
1. 在“所有用户”页上，搜索用户“Sumesh” 。
1. 打开用户 Sumesh Rajan 的属性页，然后验证“职务”属性是否已从 Active Directory 同步 。
1. 在 Microsoft Edge 中，返回“所有用户”页面。
1. 在“所有用户”页面上，搜索用户“Jordan” 。
1. 打开用户 Jordan Mitchell 的属性页，然后查看已从 Active Directory 同步的该用户帐户的属性。

## <a name="exercise-5-implementing-azure-ad-integration-features-in-ad-ds"></a>练习 5：在 AD DS 中实现 Azure AD 集成功能

### <a name="scenario"></a>场景

你想确定能进一步增强本地 Active Directory 安全性且管理开销尽可能小的 Azure AD 集成功能。 你还想实现对 Windows Server Active Directory 的 Azure AD 密码保护和通过密码写回实现的自助式密码重置。

此练习的主要任务是：

1. 在 Azure 中启用自助式密码重置。
1. 在 Azure AD Connect 中启用密码写回。
1. 在 Azure AD Connect 中启用直通身份验证。
1. 验证 Azure 中的直通身份验证。
1. 安装并注册 Azure AD 密码保护代理服务和 DC 代理。
1. 在 Azure 中启用密码保护。

#### <a name="task-1-enable-self-service-password-reset-in-azure"></a>任务 1：在 Azure 中启用自助式密码重置

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到 Azure AD 的“许可证”页面，并激活“Azure AD Premium P2”免费试用版  。 
1. 将 Azure AD Premium P2 许可证分配给练习 1 中创建的 Azure AD 全局管理员用户帐户。
1. 在 Azure 门户中，浏览到 Azure AD“密码重置”页面。
1. 注意，在“密码重置”页面上可以选择要应用配置的用户范围。

   > 注意：不要启用密码重置功能，因为它会中断本实验稍后要执行的配置步骤。

#### <a name="task-2-enable-password-writeback-in-azure-ad-connect"></a>任务 2：在 Azure AD Connect 中启用密码写回

1. 在 SEA-ADM1 上，打开“Azure AD Connect” 。
1. 在“Microsoft Azure Active Directory Connect”窗口中，选择“配置” 。
1. 在“其他任务”页面上，选择“自定义同步选项” 。
1. 在“连接到 Azure AD”页面上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码。
1. 在“可选功能”页面上，选择“密码写回” 。

   > 注意：Active Directory 用户的自助式密码重置需要密码写回。 这使用户在 Azure AD 中更改的密码能够同步到 Active Directory。

1. 在“准备好配置”页面上，查看要执行的操作的列表，然后选择“配置” 。
1. 配置完成后，关闭“Microsoft Azure Active Directory Connect”窗口。

#### <a name="task-3-enable-pass-through-authentication-in-azure-ad-connect"></a>任务 3：在 Azure AD Connect 中启用直通身份验证

1. 在 SEA-ADM1 上的“开始”菜单上，展开“Azure AD Connect”，然后选择“Azure AD Connect”   。
1. 在“Microsoft Azure Active Directory Connect”窗口中，选择“配置” 。
1. 在“其他任务”页面上，选择“更改用户登录名” 。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码。
1. 在“用户登录”页面上，选择“直通身份验证” 。
1. 验证是否选中了“启用单一登录”复选框。
1. 在“启用单一登录”页面上，选择“输入凭证” 。
1. 在“林凭据”对话框中，使用下列凭据进行身份验证：

   - 用户名：Administrator
   - 密码：Pa55w.rd

1. 在“启用单一登录”页面上，验证“输入凭证”旁边是否有绿色复选标记 。
1. 在“准备好配置”页面上，查看要执行的操作的列表，然后选择“配置” 。
1. 配置完成后，关闭“Microsoft Azure Active Directory Connect”窗口。

#### <a name="task-4-verify-pass-through-authentication-in-azure"></a>任务 4：验证 Azure 中的直通身份验证

1. 在 SEA-ADM1 上，从 Azure 门户中的“Azure Active Directory”页面浏览到“Azure AD Connect”页面  。
1. 在“Azure AD Connect”页面上，查看“用户登录”下的信息 。
1. 在“用户登录”下，选择“无缝单一登录” 。
1. 在“无缝单一登录”页面上，查看本地域名。
1. 从“无缝单一登录”页面，浏览到“直通身份验证”页 。
1. 在“直通身份验证”页上，查看“身份验证代理”下的服务器列表 。

   > 注意：若要在环境中的多台服务器上安装 Azure AD 身份验证代理，可以从 Azure 门户中的“直通身份验证”页面下载其二进制文件 。

#### <a name="task-5-install-and-register-the-azure-ad-password-protection-proxy-service-and-dc-agent"></a>任务 5：安装并注册 Azure AD 密码保护代理服务和 DC 代理

1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到“Microsoft 下载”网站，然后浏览到可下载安装程序的“Windows Server Active Directory 的 Azure AD 密码保护”页面，然后选择“下载”  。
1. 将 AzureADPasswordProtectionProxySetup.exe 和 AzureAdPasswordProtectiondAgentSetup.msi 下载到 SEA-ADM1  。

   > 注意：建议在不是域控制器的服务器上安装代理服务。 此外，代理服务不应安装在与 Azure AD Connect 代理相同的服务器上。 将在 SEA-SVR1 上安装代理服务，并在 SEA-DC1 上安装密码保护 DC 代理 。

1. 在 SEA-ADM1 上的 Windows PowerShell 控制台中，运行以下命令，删除 Zone.Identifier 替代数据流，指示已从 internet 下载文件 ：

   ```powershell
   Get-ChildItem -Path "$env:USERPROFILE\Downloads -File | Unblock-File
   ```
1. 运行以下命令在 SEA-SVR1 上创建 C:\Temp 目录，将 AzureADPasswordProtectionProxySetup.exe 安装程序复制到该目录，然后调用该安装程序  ：

   ```powershell
   New-Item -Type Directory -Path '\\SEA-SVR1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionProxySetup.exe" -Destination '\\SEA-SVR1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Temp\AzureADPasswordProtectionProxySetup.exe -ArgumentList `/quiet /log C:\Temp\AzureADPPProxyInstall.log` -Wait }
   ```
1. 运行以下命令在 SEA-DC1 上创建 C:\Temp 目录，将AzureADPasswordProtectionDCAgentSetup.msi 安装程序复制到该目录，调用安装程序，并在安装完成后重新启动域控制器  ：

   ```powershell
   New-Item -Type Directory -Path '\\SEA-DC1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionDCAgentSetup.msi" -Destination '\\SEA-DC1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-DC1.contoso.com -ScriptBlock { Start-Process msiexec.exe -ArgumentList '/i C:\Temp\AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart /log C:\Temp\AzureADPPInstall.log' -Wait }
   Restart-Computer -ComputerName SEA-DC1.contoso.com -Force
   ```
1. 运行以下命令，验证安装过程是否会创建实现 Azure AD 密码保护所需的服务：

   ```powershell
   Get-Service -Computer SEA-SVR1 -Name AzureADPasswordProtectionProxy | fl
   Get-Service -Computer SEA-DC1 -Name AzureADPasswordProtectionDCAgent | fl
   ```

   > 注意：验证每个服务都有“运行”状态 。

1. 运行以下命令，将一个 PowerShell 远程处理会话启动到 SEA-SVR1：

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1
   ```

1. 在 PowerShell 远程处理会话中，运行以下命令以在 Active Directory 中注册代理服务（将 `<Azure_AD_Global_Admin>` 占位符替换为在练习 1 中创建的 Azure AD 全局管理员用户帐户的完全限定用户主体名称）：

   ```powershell
   Register-AzureADPasswordProtectionProxy -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. 按照提示使用在练习 1 中创建的 Azure AD 全局管理员用户帐户进行身份验证。 
1. 退出 PowerShell 远程处理会话。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按下 Enter 键，将一个 PowerShell 远程处理会话启动到 SEA-DC1 ：

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```

1. 在 PowerShell 远程处理会话中，运行以下命令以在 Active Directory 中注册代理服务（将 `<Azure_AD_Global_Admin>` 占位符替换为在练习 1 中创建的 Azure AD 全局管理员用户帐户的完全限定用户主体名称）：

   ```powershell
   Register-AzureADPasswordProtectionForest -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. 按照提示使用在练习 1 中创建的 Azure AD 全局管理员用户帐户进行身份验证。 
1. 退出 PowerShell 远程处理会话。

#### <a name="task-6-enable-password-protection-in-azure"></a>任务 6：在 Azure 中启用密码保护

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，返回“Azure Active Directory”页面，然后浏览到其“安全”页面  。
1. 在“安全”页面上，选择“身份验证方法” 。
1. 在“身份验证方法”页面上，选择“密码保护” 。
1. 在“密码保护”页面上，启用“强制执行自定义列表” 。
1. 在“自定义受禁密码列表”文本框中，输入以下单词（每行一个）：
 
   - **Contoso**
   - **伦敦**

   > 注意：受禁密码列表中的词语应与组织相关。

1. 验证是否已启用“在 Windows Server Active Directory 上启用密码保护”设置。 
1. 验证“模式”是否设置为“审核”，并保存更改 。

## <a name="exercise-6-cleaning-up"></a>练习 6：清理

### <a name="scenario"></a>场景

你需要禁用从本地 Active Directory 到 Azure 的同步。 这涉及删除 Azure AD Connect 和禁用与 Azure 的同步。

此练习的主要任务是：

1. 卸载 Azure AD Connect。
1. 在 Azure 中禁用目录同步。

#### <a name="task-1-uninstall-azure-ad-connect"></a>任务 1：卸载 Azure AD Connect

1. 在 SEA-ADM1 上，打开“控制面板” 。
1. 使用“卸载或更改程序”功能卸载 Microsoft Azure AD Connect 。

#### <a name="task-2-disable-directory-synchronization-in-azure"></a>任务 2：在 Azure 中禁用目录同步

1. 在 SEA-ADM1 上，切换到“Windows PowerShell”窗口 。
1. 在“Windows PowerShell”控制台中，运行以下命令以安装适用于 Azure AD 的 Microsoft Online 模块：

   ```powershell
   Install-Module -Name MSOnline
   ```
1. 运行以下命令，向 Azure 提供凭据：

   ```powershell
   $msolcred=Get-Credential
   ```
1. 出现提示时，在“Windows PowerShell 凭据请求”对话框中，输入在练习 1 中创建的用户帐户的凭据。
1. 运行以下命令以连接到 Azure：

   ```powershell
   Connect-MsolService -Credential $msolcred
   ```
1. 运行以下命令以在 Azure 中禁用目录同步：

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false
   ```

### <a name="prepare-for-the-next-module"></a>为下一个模块做准备

完成下一个模块的准备工作后，结束本实验室。
