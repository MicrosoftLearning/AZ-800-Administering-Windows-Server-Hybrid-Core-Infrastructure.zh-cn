---
lab:
  title: 实验室：实现 AD DS 与 Microsoft Entra AD 之间的集成
  type: Answer Key
  module: 'Module 2: Implementing Identity in Hybrid Scenarios'
---

# 实验室答案密钥：实现 AD DS 与 Microsoft Entra AD 之间的集成

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Implementing%20integration%20between%20AD%20DS%20and%20Azure%20AD)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## 实验室教学设置

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到 Azure 门户，并使用 Azure 凭据进行身份验证。
1. 在 **Azure 门户**中浏览到 Microsoft Entra ID。
1. 在概览中，选择**管理租户**，然后选择 **+ 创建**。
1. 应选择 Microsoft Entra ID，然后选择“ **下一步：配置**”。
1. 在组织名称中，输入唯一名称，例如 **Contoso Organization**。
1. 在域名称中，输入唯一名称，例如 **Contoso35501731**。
1. 选择“查看 + 创建”，然后选择“创建”。

   > **注意**：完成验证码，单击提交，等待租户创建完成。

1. 在通知中，选择消息中创建的租户：**租户创建成功。单击此处导航到新租户：（此处为租户链接）**

## 练习 1：为 AD DS 集成准备本地 Microsoft Entra ID

#### 任务 1：在 Azure 中创建自定义域

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到 Azure 门户，并使用 Azure 凭据进行身份验证。
1. 在 **Azure 门户**中浏览到 Microsoft Entra ID。
1. 在“Microsoft Entra ID”选项页上，选择“自定义域名”。
1. 在“自定义域名”页上，选择“添加自定义域” 。
1. 在**自定义域名**窗格的**自定义域名**文本框中，输入`contoso.com`，然后选择**添加域**。
1. 在 **contoso.com** 自定义域名页上，查看要用于验证域的域名系统 (DNS) 记录类型。
1. 关闭窗格而不验证域名。

   > 注意：虽然一般来说要使用 DNS 记录来验证域，但此实验不需要使用验证的域。

#### 任务 2：创建具有全局管理员角色的用户

1. 在 **SEA-ADM1** 上，在 Azure 门户的** Microsoft Entra ID** 页面上选择**用户**。
1. 在**所有用户**页面上，选择 **+ 新用户**，从下拉列表中选择**创建新用户**。
1. 在“创建新建用户”页的“标识”下，在“用户名”和“显示名称”文本框中，输入“admin”    。

   > **注意**：确保**用户 principal nam** 的域名下拉菜单列出以 .NET 结尾的默认域名`onmicrosoft.com`。

1. 在**密码**中，选择**复制图标**，并记录本实验室稍后将使用的密码。
1. 选择下一步： 属性 >.
1. 在“属性”选项卡上，在“使用位置”中选择“美国”，然后选择 "下一步”： 作业 >。
1. 在**分配**选项卡上，选择 **+ 添加角色**，在目录角色列表中选择全局管理员，然后选择“选择”按钮。
1. 在“新用户”页面，选择“审查 + 创建”，然后选择“创建”。在“新建用户”页上，选择“查看 + 创建”，然后选择“创建”。

#### 任务 3：更改具有全局管理员角色的用户的密码

1. 在 Azure 门户中，选择你的用户帐户，然后选择“注销”。
1. 在“选取帐户”页上，选择“使用另一个帐户” 。
1. 在“登录”页上，输入之前创建的用户帐户的完全限定用户名，然后选择“下一步” 。
1. 对于当前密码，请使用在上一步中记下的密码。
1. 输入复杂密码两次，然后选择“登录”。

   > 注意：记录所使用的复杂密码，因为稍后将在本实验中使用。

## 练习 2：为 Microsoft Entra AD 集成准备本地 AD DS

#### 任务 1：安装 IdFix

1. 在 SEA-ADM1 上，打开 Microsoft Edge，然后浏览到  。
1. 在“Github”页面上的“ClickOnce 启动”下，选择“启动”  。
1. 在状态栏上，选择“打开文件”。
1. 在“应用程序安装 - 安全警告”对话框中，选择“安装” 。
1. 在“IdFix 隐私声明”对话框中，查看免责声明，然后选择“确定” 。

#### 任务 2：运行 IdFix

1. 在“IdFix”窗口中，选择“查询” 。
1. 如果出现“架构警告”对话框，请选择“是” 。
1. 查看 Active Directory 中的对象列表，并查看 ERROR 和 ATTRIBUTE 列 。 在这种情况下，“ContosoAdmin”的 displayName 的值为空，UPDATE 列中会显示工具建议的新值  。
1. 在“IdFix”窗口中的 ACTION 下拉菜单中，选择“编辑”，然后选择“应用”以自动实现建议的更改   。
1. 在“应用挂起”对话框中，选择“是” 。
1. 关闭 IdFix 工具。

## 练习 3：下载、安装和配置 Microsoft Entra Connect

#### 任务 1：安装和配置 Microsoft Entra Connect

   > 注意：下载 Microsoft Entra Connect 应用程序时，应用程序仍显示旧名称 Azure AD Connect。

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到 Microsoft Entra ID。
1. 在 Microsoft Entra ID 页面上，选择 Microsoft Entra Connect。
1. 在“Microsoft Entra Connect | 开始”页面上，选择“连接同步”。
1. 在“Microsoft Entra Connect | Connect Sync "页面上，选择“下载 Microsoft Entra Connect”。
1. 在新打开的页面上，在 Azure AD Connect V2 下选择下载。
1. 在状态栏上，选择“打开文件”。
1. 在“Microsoft Azure Active Directory Connect”页面上，选中“我同意许可条款和隐私通知”复选框，然后选择“继续”  。
1. 在“Express 设置”页面上，选择“使用快速设置” 。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码，然后选择“下一步” 。
1. 在“登录到您的帐户 "页面，使用新创建的用户登录，然后选择“下一步”。

1. 在“连接 AD DS”页上，输入以下凭据，然后选择“下一步” ：

   - 用户名：`CONTOSO\Administrator`
   - 密码：`Pa55w.rd`

1. 在“Azure AD 登录配置”页面上，注意添加的新域包含在 Active Directory UPN 后缀列表中，但其状态列为“未验证” 。

   > 注意：提供的域名不必是已验证的域。 虽然通常会在安装 Microsoft Entra Connect 之前验证某个域，但此实验不需要执行该验证步骤。

1. 选择“继续操作且不将所有 UPN 后缀与已验证的域进行匹配”复选框，然后选择“下一步” 。
1. 在“准备好配置”页上，查看操作列表，然后选择“安装” 。
1. 在“配置完成”页面上，选择“退出” 。

## 练习 4：验证 AD DS 与 Microsoft Entra AD 之间的集成

#### 任务 1：验证 Azure 门户中的同步

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口。 
1. 刷新“Microsoft Entra Connect”页面，查看“从 Active Directory 预配”下的信息 。
1. 在“Microsoft Entra ID”选项页上，选择“用户”。
1. 请注意，用户列表包含从 Active Directory 同步的用户。

   > 注意：目录同步启动后，Active Directory 对象会在最多 15 分钟后显示在 Microsoft Entra ID 门户中。

1. 在 Microsoft Edge 中，返回“Microsoft Entra ID”页面。
1. 在“Microsoft Entra ID”选项页上，选择“组”。
1. 请注意从 Active Directory 同步的组的列表。 


#### 任务 2：更新 Active Directory 中的用户帐户

1. 在 SEA-ADM1 上，在服务器管理器的“工具”菜单中选择“Active Directory 用户和计算机”  。
1. 在“Active Directory 用户和计算机”中，展开“销售”组织单位 (OU)，然后打开“Sumesh Rajan”的属性  。
1. 在用户的属性中，选择“组织”选项卡。
1. 在“职务”文本框中，输入“经理”，然后选择“确定”  。

#### 任务 3：在 Active Directory 中创建用户帐户

1. 在“Active Directory 用户和计算机”中，右键单击或访问“销售”OU 的上下文菜单，选择“新建”，然后选择“用户”   。
1. 在“新建对象 - 用户”窗口中，输入每个字段的以下用户详细信息，然后选择“下一步” ：

   - 名字：Jordan
   - 姓氏：Mitchell
   - 用户登录名：Jordan

1. 在“密码”和“确认密码”字段中，输入“Pa55w.rd”，然后选择“下一步”   。
1. 选择“完成”。

#### 任务 4：将更改同步到 Microsoft Entra ID

1. 在 SEA-ADM1 上，在“开始”菜单中选择“Windows PowerShell”  。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 触发同步：

   ```powershell
   Start-ADSyncSyncCycle
   ```

   > 注意：同步周期启动后，Active Directory 对象会在最多 15 分钟后显示在 Microsoft Entra ID 门户中。

#### 任务 5：验证更改到 Microsoft Entra ID

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后返回“Microsoft Entra ID”页面 。
1. 在“Microsoft Entra ID”选项页上，选择“用户”。
1. 在“所有用户”页上，搜索用户“Sumesh” 。
1. 打开用户 Sumesh Rajan 的属性页，然后验证“职务”属性是否已从 Active Directory 同步 。
1. 在 Microsoft Edge 中，返回“所有用户”页面。
1. 在“所有用户”页面上，搜索用户“Jordan” 。
1. 打开用户 Jordan Mitchell 的属性页，然后查看已从 Active Directory 同步的该用户帐户的属性。


## 练习 5：在 AD DS 中实现 Microsoft Entra AD 集成功能

#### 任务 1：在 Azure 中启用自助式密码重置

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到 Microsoft Entra ID 页面。
1. 在“Microsoft Entra ID”选项页上，选择“许可”。
1. 在“许可证”页上，选择“所有产品” 。
1. 在“所有产品”页上，选择“+ 试用/购买” 。
1. 在“激活”窗格中的“**Microsoft Entra ID P2**”下，选择“免费试用版”，然后选择“激活”。
1. 浏览到“所有产品”页面并选择 Microsoft Entra ID P2。
1. 在“Microsoft Entra ID P2 授权用户”页面，选择 + 分配。
1. 在“分配许可证”页上，选择“+ 添加用户和组” 。
1. 在“添加用户和组”页上，搜索“管理员”，从结果列表中选择管理员帐户，然后选择“选择”   。
1. 返回“分配许可证”页，选择“查看 + 分配”，然后选择“分配”  。

   > 注意：这是在此实验室中实现 Microsoft Entra ID 密码保护所必需的。

1. 返回“Microsoft Entra ID”页面。
1. 在“Microsoft Entra ID”选项页上，选择“密码重置”。
1. 注意，在“密码重置”页面上可以选择要应用配置的用户范围。

   > 注意：不要启用密码重置功能，因为它会中断本实验稍后要执行的配置步骤。

#### 任务 2：在 Microsoft Entra Connect 中启用密码写回

1. 在 **SEA-ADM1** 上的**开始**菜单中，搜索并选择 **Azure Ad Connect**。
1. 在“Microsoft Azure Active Directory Connect”窗口中，选择“配置” 。
1. 在“其他任务”页上，依次选择“自定义同步选项”和“下一步”。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码，然后选择“下一步” 。
1. 在“连接目录”页上选择“下一步”。********
1. 在“域和 OU 筛选”页上选择“下一步”。********
1. 在“可选功能”页面上，选中“密码写回”，然后选择“下一步”  。

   > 注意：Active Directory 用户的自助式密码重置需要密码写回。 这使用户在 Microsoft Entra ID 中更改的密码能够同步到 Active Directory。

1. 在“准备好配置”页面上，查看要执行的操作的列表，然后选择“配置” 。
1. 在“配置完成”页面上，选择“退出” 。

#### 任务 3：在 Microsoft Entra Connect 中启用直通身份验证

1. 在 **SEA-ADM1** 上的**开始**菜单中，搜索并选择 **Azure Ad Connect**。
1. 在“Microsoft Azure Active Directory Connect”窗口中，选择“配置” 。
1. 在“其他任务”页面上，选择“更改用户登录名”，然后选择“下一步”  。
1. 在“连接到 Azure AD”页上，输入在练习 1 中创建的 Azure AD 全局管理员用户帐户的用户名和密码，然后选择“下一步” 。
1. 在“用户登录”页面上，选择“直通身份验证” 。
1. 验证是否选中了“启用单一登录”复选框，然后选择“下一步” 。
1. 在“启用单一登录”页面上，选择“输入凭证” 。
1. 在“林凭据”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：`Administrator`
   - 密码：`Pa55w.rd`

1. 在“启用单一登录”页面上，验证“输入凭据”旁边是否有绿色复选标记，然后选择“下一步”  。
1. 在“准备好配置”页面上，查看要执行的操作的列表，然后选择“配置” 。
1. 在“配置完成”页面上，选择“退出” 。

#### 任务 4：验证 Azure 中的直通身份验证

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后返回“Microsoft Entra ID”页面 。
1. 在 Azure 门户的 **Microsoft Entra ID** 页面上，选择 **Microsoft Entra Connect**。
1. 在“Microsoft Entra Connect | 开始”页面上，选择“连接同步”。
1. 在 **Microsoft Entra Connect | Connect Sync** 页面上，查看**用户登录**下的信息。
1. 在“用户登录”下，选择“无缝单一登录” 。
1. 在“无缝单一登录”页面上，留意本地域名。
1. 在 Microsoft Edge 中，返回 **Microsoft Entra Connect** 页面。
1. 在**Microsoft Entra Connect**页上的**用户登录**下，选择**直通身份验证**。
1. 在“直通身份验证”页上，留意“身份验证代理”下的 SEA-ADM1 服务器名称  。

   > **注意**：若要在环境中的多台服务器上安装 Microsoft Entra ID 身份验证代理，可以从 Azure 门户中的**直通身份验证**页面下载其二进制文件 。 

#### 任务 5：安装并注册 Azure AD 密码保护代理服务和 DC 代理

1. 在 **SEA-ADM1** 上启动 Microsoft Edge，转到`https://www.microsoft.com/en-us/download/details.aspx?id=57071`
1. 在** Azure AD Password Protection** 页面上，选择**下载**。
1. 在**选择所需的下载**页面上，选择 **AzureADPasswordProtectionProxySetup.exe** 和 **AzureADPasswordProtectionDCAgentSetup.msi** 文件，然后选择 **下载**。
1. 在“下载多个文件”对话框中，选择“允许” 。

   > 注意：建议在不是域控制器的服务器上安装代理服务。 此外，代理服务不应安装在与 Microsoft Entra Connect 代理相同的服务器上。 将在 SEA-SVR1 上安装代理服务，并在 SEA-DC1 上安装密码保护 DC 代理 。

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台窗口 。
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，删除 Zone.Identifier 替代数据流，指示已从 internet 下载文件：

   ```powershell
   Get-ChildItem -Path "$env:USERPROFILE\Downloads" -File | Unblock-File
   ```
1. 运行以下命令，在 SEA-SVR1 上创建 C:\Temp 目录，将 AzureADPasswordProtectionProxySetup.exe 安装程序复制到该目录，然后调用该安装程序  ：

   ```powershell
   New-Item -Type Directory -Path '\\SEA-SVR1.contoso.com\C$\Temp' -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\AzureADPasswordProtectionProxySetup.exe" -Destination '\\SEA-SVR1.contoso.com\C$\Temp\'
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Temp\AzureADPasswordProtectionProxySetup.exe -ArgumentList '/quiet /log C:\Temp\AzureADPPProxyInstall.log' -Wait }
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

1. 根据指示，打开另一个 Microsoft Edge 窗口，浏览到  ，当出现提示时，输入 PowerShell 远程处理会话中显示的消息中所包含的代码。 
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

1. 根据指示，打开另一个 Microsoft Edge 窗口，浏览到  ，当出现提示时，输入 PowerShell 远程处理会话中显示的消息中所包含的代码。 
1. 出现提示时，使用在练习 1 中创建的 Microsoft Entra ID 全局管理员用户帐户进行身份验证，然后选择“继续”。
1. 切换回 PowerShell 远程处理会话，输入以下命令，然后按 Enter，将 PowerShell 远程处理会话退出到 SEA-DC1：

   ```powershell
   Exit-PSsession
   ```

#### 任务 6：在 Azure 中启用密码保护

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，，返回“Microsoft Entra ID”页面 。 
1. 在“Microsoft Entra ID”选项页上，选择“安全”。
1. 在“安全”页面上，选择“身份验证方法” 。
1. 在“身份验证方法”页面上，选择“密码保护” 。
1. 在“密码保护”页面上，将“强制执行自定义列表”更改未“是”  。
1. 在“自定义受禁密码列表”文本框中，输入以下单词（每行一个）：
 
   - **Contoso**
   - **伦敦**

   > 注意：受禁密码列表中的词语应与组织相关。

1. 验证“在 Windows Server Active Directory 上启用密码保护”是否设置为“是” 。
1. 验证“模式”是否设置为“审核”，然后选择“保存”  。

## 练习 6：清理

#### 任务 1：卸载 Azure AD Connect

1. 在 SEA-ADM1 上，在“开始”菜单中选择“控制面板”  。
1. 在“控制面板”窗口中的“程序”下选择“卸载程序”  。
1. 在“卸载或更改程序”窗口中选择“Azure AD Connect”，然后选择“卸载”  。
1. 在“程序和功能”对话框中，选择“是” 。
1. 在“卸载 Azure AD Connect”窗口中，选择“删除” 。
1. 卸载 Azure AD Connect 后，请在“卸载 Azure AD Connect”窗口中选择“退出” 。

#### 任务 2：在 Azure 中禁用目录同步

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台窗口 。
1. 在“Windows PowerShell”控制台中，输入以下命令并按 Enter，安装适用于 Microsoft Entra ID 的 Microsoft Online 模块：

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
1. 在 Windows PowerShell 控制台中，输入以下命令然后按 Enter，对 Microsoft Entra 租户进行身份验证：

   ```powershell
   Connect-MsolService -Credential $msolCred
   ```
1. 输入以下命令并按 Enter，以禁用目录同步：

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false
   ```
1. 出现确认提示时，输入 Y，然后按 Enter。
