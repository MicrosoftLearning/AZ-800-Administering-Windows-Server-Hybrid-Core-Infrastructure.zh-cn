---
lab:
  title: 实验室：实现 AD DS 与 Azure AD 之间的集成
  type: Answer Key
  module: 'Module 2: Implementing Identity in Hybrid Scenarios'
ms.openlocfilehash: 8cbcc563a86ca5c1c997a69884e1b132cbf2f86e
ms.sourcegitcommit: bd43c7961e93ef200b92fb1d6f09d9ad153dd082
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2022
ms.locfileid: "137906932"
---
# <a name="lab-implementing-integration-between-ad-ds-and-azure-ad"></a>实验室：实现 AD DS 与 Azure AD 之间的集成

## <a name="exercise-1-preparing-azure-ad-for-ad-ds-integration"></a>练习 1：为与 AD DS 集成准备 Azure AD

#### <a name="task-1-create-a-custom-domain-in-azure"></a>任务 1：在 Azure 中创建自定义域

1. 连接到 SEA-ADM1，然后根据需要，以 Contoso\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到 Azure 门户，并使用 Azure 凭据进行身份验证。
1. 在 Azure 门户中，浏览到“Azure Active Directory”。
1. 在“Azure Active Directory”  页上，选择“自定义域名”  。
1. 在“自定义域名”页上，选择“添加自定义域” 。
1. 在“自定义域名”窗格的“自定义域名”文本框中，输入“contoso.com”，然后选择“添加域”   。
1. 在 `contoso.com` 自定义域名页上，查看要用于验证域的域名系统 (DNS) 记录类型。
1. 关闭窗格而不验证域名。

   > 注意：虽然一般来说要使用 DNS 记录来验证域，但此实验不需要使用验证的域。

#### <a name="task-2-create-a-user-with-the-global-administrator-role"></a>任务 2：创建具有全局管理员角色的用户

1. 在 SEA-ADM1 上，在 Azure 门户中的“Azure Active Directory”页上选择“用户”  。
1. 在“所有用户”页上，选择“新建用户” 。
1. 在“新建用户”页的“标识”下，在“用户名”和“名称”文本框中，输入“admin”    。

   > 注意：确保“用户名”的域名下拉菜单列出了以 `onmicrosoft.com` 结尾的默认域名 。

1. 在“密码”下，选中“显示密码”复选框 。 记录密码，因为稍后将在本实验室中使用它。
1. 在“组和角色”下，选择“角色”旁边的“用户”  。
1. 在“目录角色”页上，从角色列表中选择“全局管理员”，然后选择“选择”  。
1. 返回到“新建用户”页，在“使用位置”下拉列表中选择“美国”  。
1. 在“新建用户”页上，选择“创建” 。

#### <a name="task-3-change-the-password-for-the-user-with-the-global-administrator-role"></a>任务 3：更改具有全局管理员角色的用户的密码

1. 在 Azure 门户中，选择你的用户帐户，然后选择“注销”。
1. 在“选取帐户”页上，选择“使用另一个帐户” 。
1. 在“登录”页上，输入之前创建的用户帐户的完全限定用户名，然后选择“下一步” 。
1. 对于当前密码，请使用在上一步中记下的密码。
1. 输入复杂密码两次，然后选择“登录”。

   > 注意：记录所使用的复杂密码，因为稍后将在本实验中使用。

## <a name="exercise-2-preparing-on-premises-ad-ds-for-azure-ad-integration"></a>练习 2：为 Azure AD 集成准备本地 AD DS

#### <a name="task-1-install-idfix"></a>任务 1：安装 IdFix

1. 在 SEA-ADM1 上，打开 Microsoft Edge，然后浏览到 https://github.com/microsoft/idfix 。
1. 在“Github”页面上的“ClickOnce 启动”下，选择“启动”  。
1. 在状态栏上，选择“打开文件”。
1. 在“应用程序安装 - 安全警告”对话框中，选择“安装” 。
1. 在“IdFix 隐私声明”对话框中，查看免责声明，然后选择“确定” 。

#### <a name="task-2-run-idfix"></a>任务 2：运行 IdFix

1. 在“IdFix”窗口中，选择“查询” 。
1. 如果出现“架构警告”对话框，请选择“是” 。
1. 查看 Active Directory 中的对象列表，并查看 ERROR 和 ATTRIBUTE 列 。 在这种情况下，“ContosoAdmin”的 displayName 的值为空，UPDATE 列中会显示工具建议的新值  。
1. 在“IdFix”窗口中的 ACTION 下拉菜单中，选择“编辑”，然后选择“应用”以自动实现建议的更改   。
1. 在“应用挂起”对话框中，选择“是” 。
1. 关闭 IdFix 工具。

## <a name="exercise-3-downloading-installing-and-configuring-azure-ad-connect"></a>练习 3：下载、安装和配置 Azure AD Connect

#### <a name="task-1-install-and-configure-azure-ad-connect"></a>任务 1：安装和配置 Azure AD Connect

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“Azure Active Directory” 。
1. 在“Azure Active Directory”页上，选择“Azure AD Connect” 。
1. 在“Azure AD Connect”页上，选择“下载 Azure AD Connect” 。
1. 在“Microsoft Azure Active Directory Connect”页面中，选择“下载” 。
1. 在状态栏上，选择“打开文件”。
1. 在“Microsoft Azure Active Directory Connect”页面上，选中“我同意许可条款和隐私通知”复选框，然后选择“继续”  。
1. 在“快速设置”页面上，选择“使用快速设置” 。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码，然后选择“下一步” 。
1. 在“连接 AD DS”页上，输入以下凭据，然后选择“下一步” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在“Azure AD 登录配置”页面上，注意添加的新域包含在 Active Directory UPN 后缀列表中，但其状态列为“未验证” 。

   > 注意：提供的域名不必是已验证的域。 虽然通常会在安装 Azure AD Connect 之前验证某个域，但此实验不需要执行该验证步骤。

1. 选择“继续操作且不将所有 UPN 后缀与已验证的域进行匹配”复选框，然后选择“下一步” 。
1. 在“准备好配置”页上，查看操作列表，然后选择“安装” 。
1. 在“配置完成”页面上，选择“退出” 。

## <a name="exercise-4-verifying-integration-between-ad-ds-and-azure-ad"></a>练习 4：验证 AD DS 与 Azure AD 之间的集成

#### <a name="task-1-verify-synchronization-in-the-azure-portal"></a>任务 1：验证 Azure 门户中的同步

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口。 
1. 刷新“Azure AD Connect”页面，查看“从 Active Directory 预配”下的信息 。
1. 在“Azure Active Directory”页上，选择“用户” 。
1. 请注意，用户列表包含从 Active Directory 同步的用户。

   > 注意：目录同步启动后，Active Directory 对象会在最多 15 分钟后显示在 Azure AD 门户中。

1. 在 Microsoft Edge 中，返回到“Azure Active Directory”页。
1. 在“Azure Active Directory”页上，选择“组” 。
1. 请注意从 Active Directory 同步的组的列表。 

#### <a name="task-2-verify-synchronization-in-the-synchronization-service-manager"></a>任务 2：验证 Synchronization Service Manager 中的同步

1. 在 SEA-ADM1 上的“开始”菜单上，展开“Azure AD Connect”，然后选择“同步服务”   。
1. 在“Synchronization Service Manager”窗口中的“操作”选项卡下，观察为同步 Active Directory 对象执行的任务 。
1. 选择“连接器”选项卡，并记下这两个连接器。

   > 注意：一个连接器用于 AD DS，另一个连接器用于 Azure AD 租户。 

1. 关闭“Synchronization Service Manager”窗口。

#### <a name="task-3-update-a-user-account-in-active-directory"></a>任务 3：更新 Active Directory 中的用户帐户

1. 在 SEA-ADM1 上，在服务器管理器的“工具”菜单中选择“Active Directory 用户和计算机”  。
1. 在“Active Directory 用户和计算机”中，展开“销售”组织单元 (OU)，然后打开“Ben Miller”的属性  。
1. 在用户的属性中，选择“组织”选项卡。
1. 在“职务”文本框中，输入“经理”，然后选择“确定”  。

#### <a name="task-4-create-a-user-account-in-active-directory"></a>任务 4：在 Active Directory 中创建用户帐户

1. 在“Active Directory 用户和计算机”中，右键单击或访问“销售”OU 的上下文菜单，选择“新建”，然后选择“用户”   。
1. 在“新建对象 - 用户”窗口中，输入每个字段的以下用户详细信息，然后选择“下一步” ：

   - 名字：Jordan
   - 姓氏：Mitchell
   - 用户登录名：Jordan

1. 在“密码”和“确认密码”字段中，输入“Pa55w.rd”，然后选择“下一步”   。
1. 选择“完成”。

#### <a name="task-5-sync-changes-to-azure-ad"></a>任务 5：将更改同步到 Azure AD

1. 在 SEA-ADM1 上，在“开始”菜单中选择“Windows PowerShell”  。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 触发同步：

   ```powershell
   Start-ADSyncSyncCycle
   ```

   > 注意：同步周期启动后，Active Directory 对象会在最多 15 分钟后显示在 Azure AD 门户中。

#### <a name="task-6-verify-changes-in-azure-ad"></a>任务 6：验证 Azure AD 中的更改

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后返回“Azure Active Directory”页面 。
1. 在“Azure Active Directory”页上，选择“用户” 。
1. 在“所有用户”页面上，搜索用户“Ben” 。
1. 打开用户 Ben Miller 的属性页，然后验证“职务”属性是否已从 Active Directory 同步 。
1. 在 Microsoft Edge 中，返回“所有用户”页面。
1. 在“所有用户”页面上，搜索用户“Jordan” 。
1. 打开用户 Jordan Mitchell 的属性页，然后查看已从 Active Directory 同步的该用户帐户的属性。

## <a name="exercise-5-implementing-azure-ad-integration-features-in-ad-ds"></a>练习 5：在 AD DS 中实现 Azure AD 集成功能

#### <a name="task-1-enable-self-service-password-reset-in-azure"></a>任务 1：在 Azure 中启用自助式密码重置

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“Azure Active Directory”页 。
1. 在“Azure Active Directory”页上，选择“许可证” 。
1. 在“许可证”页上，选择“所有产品” 。
1. 在“所有产品”页上，选择“+ 试用/购买” 。
1. 在“激活”页上的“AZURE AD PREMIUM P2”下，选择“免费试用版”，然后选择“激活”   。
1. 浏览到“所有产品”页，然后选择“Azure Active Directory Premium P2” 。
1. 在“Azure Active Directory Premium P2 \| 许可用户”页上，选择“+ 分配” 。
1. 在“分配许可证”页上，选择“+ 添加用户和组” 。
1. 在“添加用户和组”页上，搜索“管理员”，从结果列表中选择管理员帐户，然后选择“选择”   。
1. 返回“分配许可证”页，选择“查看 + 分配”，然后选择“分配”  。

   > 注意：这是在此实验室中实现 Azure AD 密码保护所必需的。

1. 返回到“Azure Active Directory”页。
1. 在“Azure Active Directory”页上，选择“密码重置” 。
1. 注意，在“密码重置”页面上可以选择要应用配置的用户范围。

   > 注意：不要启用密码重置功能，因为它会中断本实验稍后要执行的配置步骤。

#### <a name="task-2-enable-password-writeback-in-azure-ad-connect"></a>任务 2：在 Azure AD Connect 中启用密码写回

1. 在 SEA-ADM1 上的“开始”菜单上，展开“Azure AD Connect”，然后选择“Azure AD Connect”   。
1. 在“Microsoft Azure Active Directory Connect”窗口中，选择“配置” 。
1. 在“其他任务”页上，依次选择“自定义同步选项”和“下一步”。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码，然后选择“下一步” 。
1. 在“连接目录”页上选择“下一步”。
1. 在“域和 OU 筛选”页上选择“下一步”。
1. 在“可选功能”页面上，选中“密码写回”，然后选择“下一步”  。

   > 注意：Active Directory 用户的自助式密码重置需要密码写回。 这使用户在 Azure AD 中更改的密码能够同步到 Active Directory。

1. 在“准备好配置”页面上，查看要执行的操作的列表，然后选择“配置” 。
1. 在“配置完成”页面上，选择“退出” 。

#### <a name="task-3-enable-pass-through-authentication-in-azure-ad-connect"></a>任务 3：在 Azure AD Connect 中启用直通身份验证

1. 在 SEA-ADM1 上的“开始”菜单上，展开“Azure AD Connect”，然后选择“Azure AD Connect”   。
1. 在“Microsoft Azure Active Directory Connect”窗口中，选择“配置” 。
1. 在“其他任务”页面上，选择“更改用户登录名”，然后选择“下一步”  。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码，然后选择“下一步” 。
1. 在“用户登录”页面上，选择“直通身份验证” 。
1. 验证是否选中了“启用单一登录”复选框，然后选择“下一步” 。
1. 在“启用单一登录”页面上，选择“输入凭证” 。
1. 在“林凭据”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：Administrator
   - 密码：Pa55w.rd

1. 在“启用单一登录”页面上，验证“输入凭据”旁边是否有绿色复选标记，然后选择“下一步”  。
1. 在“准备好配置”页面上，查看要执行的操作的列表，然后选择“配置” 。
1. 在“配置完成”页面上，选择“退出” 。

#### <a name="task-4-verify-pass-through-authentication-in-azure"></a>任务 4：验证 Azure 中的直通身份验证

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后返回“Azure Active Directory”页面 。
1. 在 Azure 门户中的“Azure Active Directory”页上选择“Azure AD Connect” 。
1. 在“Azure AD Connect”页面上，查看“用户登录”下的信息 。
1. 在“用户登录”下，选择“无缝单一登录” 。
1. 在“无缝单一登录”页面上，留意本地域名。
1. 在Microsoft Edge中，返回到“Azure AD Connect”页。
1. 在“Azure AD Connect”页上的“用户登录”下，选择“直通身份验证”  。
1. 在“直通身份验证”页面上，留意“身份验证代理”下的 SEA-ADM1 服务器名称  。

   > 注意：若要在环境中的多台服务器上安装 Azure AD 身份验证代理，可以从 Azure 门户中的“直通身份验证”页面下载其二进制文件 。 

#### <a name="task-5-install-and-register-the-azure-ad-password-protection-proxy-service-and-dc-agent"></a>任务 5：安装并注册 Azure AD 密码保护代理服务和 DC 代理

1. 在 SEA-ADM1 上，启动 Microsoft Edge，转到“Microsoft 下载”网站，然后浏览到可下载安装程序的“Windows Server Active Directory 的 Azure AD 密码保护”页面，然后选择“下载”  。
1. 在“对 Windows Server Active Directory 的 Azure AD 密码保护”页上，选择“AzureADPasswordProtectionProxySetup.msi”和“AzureADPasswordProtectionDCAgentSetup.msi”文件，然后选择“下一步”   。
1. 选择“下载”  。
1. 在“下载多个文件”对话框中，选择“允许” 。

   > 注意：建议在不是域控制器的服务器上安装代理服务。 此外，代理服务不应安装在与 Azure AD Connect 代理相同的服务器上。 将在 SEA-SVR1 上安装代理服务，并在 SEA-DC1 上安装密码保护 DC 代理 。

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台窗口 。
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，删除 Zone.Identifier 替代数据流，指示已从 internet 下载文件：

   ```powershell
   Get-ChildItem -Path "$env:USERPROFILE\Downloads" -File | Unblock-File
   ```
1. 运行以下命令在 SEA-SVR1 上创建 C:\Temp 目录，将 AzureADPasswordProtectionProxySetup.msi 安装程序复制到该目录，然后调用该安装程序  ：

   ```powershell
   New-Item -Type Directory -Path '\\SEA-SVR1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionProxySetup.msi" -Destination '\\SEA-SVR1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Temp\AzureADPasswordProtectionProxySetup.msi -ArgumentList '/quiet /log C:\Temp\AzureADPPProxyInstall.log' -Wait }
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

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter，将一个 PowerShell 远程处理会话启动到 SEA-SVR1 ：

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1
   ```

1. 从 [SEA-SVR1] 提示符处，输入以下命令并按 Enter，以在 Active Directory 中注册代理服务（将 `<Azure_AD_Global_Admin>` 占位符替换为在练习 1 中创建的 AD 全局管理员帐户的完全限定用户主体名称）：

   ```powershell
   Register-AzureADPasswordProtectionProxy -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. 根据指示，打开另一个 Microsoft Edge 窗口，浏览到 https://microsoft.com/devicelogin ，当出现提示时，输入 PowerShell 远程处理会话中显示的消息中所包含的代码。 
1. 出现提示时，使用在练习 1 中创建的 Azure AD 全局管理员用户帐户进行身份验证，然后选择“继续”。
1. 切换回 PowerShell 远程处理会话，输入以下命令，然后按 Enter，将 PowerShell 远程处理会话退出到 SEA-SVR1：

   ```powershell
   Exit-PSsession
   ```

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter，将一个 PowerShell 远程处理会话启动到 SEA-DC1 ：

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```

1. 从 [SEA-DC1] 提示符处，输入以下命令并按 Enter，以在 Active Directory 中注册代理服务（将 `<Azure_AD_Global_Admin>` 占位符替换为在练习 1 中创建的 Azure AD 全局管理员用户帐户的完全限定用户主体名称）：

   ```powershell
   Register-AzureADPasswordProtectionForest -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
   ```

1. 根据指示，打开另一个 Microsoft Edge 窗口，浏览到 https://microsoft.com/devicelogin ，当出现提示时，输入 PowerShell 远程处理会话中显示的消息中所包含的代码。 
1. 出现提示时，使用在练习 1 中创建的 Azure AD 全局管理员用户帐户进行身份验证，然后选择“继续”。
1. 切换回 PowerShell 远程处理会话，输入以下命令，然后按 Enter，将 PowerShell 远程处理会话退出到 SEA-DC1：

   ```powershell
   Exit-PSsession
   ```

#### <a name="task-6-enable-password-protection-in-azure"></a>任务 6：在 Azure 中启用密码保护

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，返回“Azure Active Directory”页面，然后在“Azure Active Directory”页上选择“安全”   。
1. 在“安全”页面上，选择“身份验证方法” 。
1. 在“身份验证方法”页面上，选择“密码保护” 。
1. 在“密码保护”页面上，将“强制执行自定义列表”滑块更改未“是”  。
1. 在“自定义受禁密码列表”文本框中，输入以下单词（每行一个）：
 
   - **Contoso**
   - **伦敦**

   > 注意：受禁密码列表中的词语应与组织相关。

1. 验证“在 Windows Server Active Directory 上启用密码保护”滑块是否设置为“是” 。
1. 验证“模式”滑块是否设置为“审核”，然后选择“保存”  。

## <a name="exercise-6-cleaning-up"></a>练习 6：清理

#### <a name="task-1-uninstall-azure-ad-connect"></a>任务 1：卸载 Azure AD Connect

1. 在 SEA-ADM1 上，在“开始”菜单中选择“控制面板”  。
1. 在“控制面板”窗口中的“程序”下选择“卸载程序”  。
1. 在“卸载或更改程序”窗口中选择“Microsoft Azure AD Connect”，然后选择“卸载”  。
1. 在“程序和功能”对话框中，选择“是” 。
1. 在“卸载 Azure AD Connect”窗口中，选择“删除” 。
1. 卸载 Azure AD Connect 后，请在“卸载 Azure AD Connect”窗口中选择“退出” 。

#### <a name="task-2-disable-directory-synchronization-in-azure"></a>任务 2：在 Azure 中禁用目录同步

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台窗口 。
1. 在“Windows PowerShell”控制台中，输入以下命令并按 Enter，安装适用于 Azure AD 的 Microsoft Online 模块：

   ```powershell
   Install-Module -Name MSOnline
   ```
1. 当系统提示安装 NuGet 提供程序时，请输入 Y，然后按 Enter。
1. 当系统提示从不受信任的存储库中安装模块时，请输入 A，然后按 Enter。
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，将 Azure AD 凭据存储在变量中：

   ```powershell
   $msolCred = Get-Credential
   ```
1. 在“Windows PowerShell 凭据请求”对话框中，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的凭据，然后选择“确定” 。
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，对 Azure AD 租户进行身份验证：

   ```powershell
   Connect-MsolService -Credential $msolCred
   ```
1. 输入以下命令并按 Enter，以禁用目录同步：

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false
   ```
1. 出现确认提示时，输入 Y，然后按 Enter。
