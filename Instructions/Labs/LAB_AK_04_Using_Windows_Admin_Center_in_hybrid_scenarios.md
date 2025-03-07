---
lab:
  title: 实验室：在混合场景中使用 Windows Admin Center
  type: Answer Key
  module: 'Module 4: Facilitating hybrid management'
---

# 实验室解答：在混合场景中使用 Windows Admin Center

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Using%20Windows%20Admin%20Center%20in%20hybrid%20scenarios)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## 练习 1：预配运行 Windows Server 的 Azure VM

#### 任务 1：使用 Azure 资源管理器模板创建 Azure 资源组

1. 连接到 **SEA-ADM1**，如果需要，请使用讲师提供的凭据登录。
1. 在 **SEA-ADM1** 上，启动 Microsoft Edge，转到 Azure 门户`https://portal.azure.com`，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录。
1. 在 Azure 门户中，通过选择搜索文本框旁边的工具栏图标打开“Cloud Shell”窗格。
1. 如果系统提示选择 Bash 或 PowerShell，请选择 PowerShell  。

   >备注：如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择要在此实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中选择“上传”，然后将文件 C:\\Labfiles\\Lab04\\L04-sub_template.json 上传到 Cloud Shell 主目录中  。
1. 在 Cloud Shell 窗格中，运行以下命令以创建一个资源组，其中包含在此实验室中预配的资源。 （将 `<Azure region>` 占位符替换为可在其中部署 Azure 虚拟机的 Azure 区域的名称，例如 eastus。）

   >注意：此实验室已使用“美国东部”进行了测试和验证，因此你应该使用该区域。 通常，若要确定可在其中预配 Azure VM 的 Azure 区域，请参阅[查找你所在区域的 Azure 额度套餐](https://aka.ms/regions-offers)。

   ```powershell
   $location = '<Azure region>'
   $rgName = 'AZ800-L0401-RG'
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az800l04subDeployment `
     -TemplateFile $HOME/L04-sub_template.json `
     -rgLocation $location `
     -rgName $rgName
   ```

#### 任务 2：使用 Azure 资源管理器模板创建 Azure VM

1. 在 Cloud Shell 窗格中，上传 Azure 资源管理器模板 **C:\\Labfiles\\Lab04\\L04-rg_template.json**。
1. 从 Cloud Shell 窗格中，运行以下命令以部署运行 Windows Server 的 Azure VM，你将在此实验室中使用它：

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az800l04rgDeployment `
     -ResourceGroupName $rgName `
     -TemplateFile $HOME/L04-rg_template.json
   ```

1. 出现提示时，插入讲师提供的凭据。

   >注意：在继续下一个练习之前，请等待部署完成。 部署大约需要 5 分钟的时间完成。

1. 在 Azure 门户中，关闭 Cloud Shell 窗格。
1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“az800l04-vnet”虚拟网络 。
1. 在“**az800l04-vnet**”页的“**设置**”部分，选择“**子网**”，然后在“**子网**”页上，选择“**+ 子网**”。
1. 在“**添加子网**”页的“设置”窗格中，指定以下设置，然后选择“**添加**”（将其他项保留为默认值）：

   |设置|“值”|
   |---|---|
   |子网用途|**虚拟网络网关**|
   |开始地址|**10.4.3.224/27**|

## 练习 2：使用 Azure 网络适配器实现混合连接

#### 任务 1：向 Azure 注册 Windows Admin Center

1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.exe"
   ```
1. 打开文件资源管理器，导航到 **Downloads** 文件夹，并运行 **WindowsAdminCenter.exe** 文件。 这将启动 **Windows Admin Center (v2) 安装程序** 向导。
1. 在“**欢迎使用 Windows Admin Center 安装向导**”页上，选择“**下一步**”。
1. 在“**许可条款和隐私声明**”页上，**接受条款**并选择“**下一步**”。
1. 在“**选择安装模式**”页上，确保选中“**快速安装**”，然后选择“**下一步**”。
1. 在“**选择 TLS 证书**”页上，确保选中“**生成自签名证书（60 天内过期）**”，然后选择“**下一步**”。
1. 在“**自动更新**”页上，选择“**通知我可用更新而不下载或安装更新**”，然后选择“**下一步**”。
1. 在“**将诊断数据发送到 Microsoft**”页上，确保选中“**必需诊断数据**”，然后选择“**下一步**”。
1. 选择“**安装**”，并在安装完成后，确保选中“**启动 Windows Admin Center: `https://SEA-ADM1.contoso.com:443`**”框，然后选择“**完成**”。

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后浏览到 `https://SEA-ADM1.contoso.com`。

   >注意：如果链接不起作用，请在 SEA-ADM1 上打开文件资源管理器，选择“下载”文件夹，在“下载”文件夹中选择 WindowsAdminCenter.msi 文件并手动安装。 安装完成后，刷新 Microsoft Edge。

   >注意：如果收到 NET::ERR_CERT_DATE_INVALID 错误，请在 Microsoft Edge 浏览器页上选择“高级”，在页面底部选择“继续访问 sea-adm1-contoso.com (不安全)” 。 
   
1. 如果出现提示，请在“**Windows 安全**”对话框中输入讲师提供的凭据，然后选择“**确定**”。

1. 在“所有连接”页上，选择“sea-adm1.contoso.com”条目 。 
1. 在 Windows Admin Center 中，选择“网络”，然后选择“+ 添加 Azure 网络适配器(预览版)” 。

   > 注意：根据屏幕分辨率，如果“操作”菜单不可用，可能需要选择省略号图标  。

1. 出现提示时，在“添加 Azure 网络适配器”窗口中选择“向 Azure 注册 Windows Admin Center” 。

   >注意：这将自动在 Windows Admin Center 内的“设置”页上显示 Azure 窗格 。

1. 在 Windows Admin Center 中，在 Azure 窗格中的“设置”页上，选择“注册” 。
1. 在“Windows Admin Center 中的 Azure 入门”窗格中，选择“复制”以复制注册过程步骤列表中显示的代码 。 
1. 在注册过程的步骤列表中，选择“输入代码”链接。

   >注意：这将在 Microsoft Edge 窗口中打开显示“输入代码”页的另一个选项卡 。

1. 在“输入代码”文本框中，将复制的代码粘贴到剪贴板，然后选择“下一步” 。
1. 在“登录”页上，提供在上一个练习中用于登录到 Azure 订阅的用户名，选择“下一步”，提供相应的密码，然后选择“登录”  。
1. 出现提示“是否尝试登录到 Windows Admin Center?”时，选择“继续” 。
1. 在 Windows Admin Center 中，验证登录是否成功，并关闭 Microsoft Edge 窗口中新打开的选项卡。
1. 在“Windows Admin Center 中的 Azure 入门”窗格中，确保“Microsoft Entra 应用程序”设置为“新建”，然后选择“连接”。
1. 在注册过程的步骤列表中，选择“登录”。 这将打开一个标有“请求的权限”的弹出窗口。
   >备注：如果登录时收到错误消息，请尝试刷新上一步中的页面。
1. 在“请求的权限”弹出窗口中，选择“代表你的组织同意”，然后选择“接受”  。

#### 任务 2：创建 Azure 网络适配器

>**备注**：由于 WAC 控制台中最近的更改，此步骤当前无法执行。

## 练习 3：在 Azure 中部署 Windows Admin Center 网关

#### 任务 1：在 Azure 中安装 Windows Admin Center 网关

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的浏览器窗口。
1. 返回 Azure 门户，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格。
1. 在 Cloud Shell 窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中选择“上传”，并将文件 C:\\Labfiles\\Lab04\\Deploy-WACAzVM.ps1 上传到 Cloud Shell 主目录中  。
1. 从 Cloud Shell 窗格中运行以下命令，启用 Windows Admin Center 预配脚本使用的 AzureRm PowerShell cmdlet 的兼容性：

   ```powershell
   Enable-AzureRmAlias -Scope Process
   ```

1. 从 Cloud Shell 窗格中运行以下命令，设置运行 Windows Admin Center 预配脚本所需的变量值：

   ```powershell
   $rgName = 'AZ800-L0401-RG'
   $vnetName = 'az800l04-vnet'
   $nsgName = 'az800l04-web-nsg'
   $subnetName = 'subnet1'
   $location = 'eastus'
   $pipName = 'wac-public-ip'
   $size = 'Standard_D2s_v3'
   ```

1. 从 Cloud Shell 窗格中运行以下命令，设置脚本参数变量：

   ```powershell
   $scriptParams = @{
     ResourceGroupName = $rgName
     Name = 'az800l04-vmwac'
     VirtualNetworkName = $vnetName
     SubnetName = $subnetName
     GenerateSslCert = $true
     size = $size
     PublicIPAddressName = $pipName
     SecurityGroupName = $nsgName
   }
   ```

1. 从 Cloud Shell 窗格中运行以下命令，禁用 PowerShell 远程处理的证书验证（系统提示确认从不受信任的存储库中安装时，请输入 A 并按 Enter）：

   ```powershell
   Install-Module -Name pswsman
   ```

   ```powershell
   Disable-WSManCertVerification -All
   ```

1. 从 Cloud Shell 窗格中运行以下命令，启动预配脚本：

   ```powershell
   ./Deploy-WACAzVM.ps1 @scriptParams
   ```

1. 系统提示提供本地管理员帐户的名称时，请输入讲师提供的**用户名**。
1. 系统提示提供本地管理员帐户的密码时，请输入讲师提供的**密码**。

   >注意：请等待预配脚本完成。 这可能需要大约 5 分钟。
   
1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户工具栏上的“**搜索资源、服务和文档**”文本框中，搜索并选择“**虚拟机**”，然后在“**虚拟机**”页上，选择 **az800l04-vmwac** 条目。
1. 在“**连接**”部分下，选择“**连接**”，然后选择“**下载 RDP 文件**”。
1. 出现提示时，插入讲师提供的凭据。
1. 在与 **az800l04-vmwac** VM 的远程桌面会话中，选择“**开始**”，然后选择“**Windows PowerShell（管理员）**”。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.exe"
   ```
1. 打开文件资源管理器，导航到 **Downloads** 文件夹，并运行 **WindowsAdminCenter.exe** 文件。 这将启动 **Windows Admin Center (v2) 安装程序** 向导。
1. 在“**欢迎使用 Windows Admin Center 安装向导**”页上，选择“**下一步**”。
1. 在“**许可条款和隐私声明**”页上，**接受条款**并选择“**下一步**”。
1. 在“**选择安装模式**”页上，确保选中“**快速安装**”，然后选择“**下一步**”。
1. 在“**选择 TLS 证书**”页上，确保选中“**生成自签名证书（60 天内过期）**”，然后选择“**下一步**”。
1. 在“**自动更新**”页上，选择“**通知我可用更新而不下载或安装更新**”，然后选择“**下一步**”。
1. 在“**将诊断数据发送到 Microsoft**”页上，确保选中“**必需诊断数据**”，然后选择“**下一步**”。
1. 选择“**安装**”，并在安装完成后，确保选中“**启动 Windows Admin Center: `https://az800l04-vmwac:443`**”框，然后选择“**完成**”。

  >**备注**：安装可能需要 5 分钟。

## 练习 4：在 Azure 中验证 Windows Admin Center 网关的功能

#### 任务 1：连接到在 Azure VM 中运行的 Windows Admin Center 网关

1. 在 **SEA-ADM1** 上的 **az800l04-vmwac** 页上，选择左侧菜单上的“**概述**”条目并复制“**DNS 名称**”。
1. 在 **SEA-ADM1** 上，启动 Microsoft Edge， 并以 `https://` 格式粘贴 **DNS 名称**。
1. 在 Microsoft Edge 窗口中，忽略消息“你的连接不是专用连接”，选择“高级”，然后选择以文本“继续转到”开始的链接  。
1. 出现提示时，在“**登录以访问此网站**”对话框中，使用讲师提供的凭据登录。
1. 在 Windows Admin Center 的“所有连接”窗格中，选择“az800l04-vmwac [网关]” 。
1. 查看 Windows Admin Center 的“概述”窗格。

#### 任务 2：在 Azure VM 上启用 PowerShell 远程处理

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口中，然后在工具栏上的“搜索资源、服务和文档”文本框中搜索并选择“虚拟机”  。
1. 在“虚拟机”页上，选择“az800l04-vm0” 。
1. 在“az800l04-vm0”页的“操作”部分，选择“运行命令”，然后选择“RunPowerShellScript”   。
1. 如果已禁用 Windows 远程管理，请在“运行命令脚本”页的“PowerShell 脚本”部分输入以下命令，然后选择“运行”以启用它  。

   ```powershell
   winrm quickconfig -quiet
   ```

1. 在“PowerShell 脚本”部分中，将在上一步中输入的文本替换为以下命令，然后选择“运行”以打开 Windows 远程管理入站端口 ：

   ```powershell
   Set-NetFirewallRule -Name WINRM-HTTP-In-TCP-PUBLIC -RemoteAddress Any
   ```

1. 在“PowerShell 脚本”部分中，将在上一步中输入的文本替换为以下命令，然后选择“运行”以启用 PowerShell 远程处理 ：

   ```powershell
   Enable-PSRemoting -Force -SkipNetworkProfileCheck
   ```

#### 任务 3：使用 Azure VM 中运行的 Windows Admin Center 网关连接到 Azure VM

1. 在 SEA-ADM1 上，在显示在 Azure VM az800l04-vmwac 上运行的 Windows Admin Center 网关界面的 Microsoft Edge 窗口中，选择“Windows Admin Center”  。
1. 在“所有连接”页面上，选择“+ 添加” 。
1. 在“添加或创建资源”页上的“服务器”部分中，选择“添加”  。
1. 在“服务器名称”文本框中，输入“az800l04-vm0” 。
1. 选择“**为此连接使用另一个帐户**”选项，提供讲师提供的凭据，然后选择“**使用凭据添加**”。
1. 在连接列表中，选择“az800l04-vm0”
1. 成功连接到该 Azure VM 后，在 Windows Admin Center 中查看 Azure VM az800l04-vmwac 的“概述”窗格。

## 练习 5：取消预配 Azure 环境

#### 任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口。
1. 在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格。

#### 任务 2：标识实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L040*'
   ```

1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L040*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >注意：该命令异步执行（由 -AsJob 参数决定），因此，虽然你可以随后立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。
