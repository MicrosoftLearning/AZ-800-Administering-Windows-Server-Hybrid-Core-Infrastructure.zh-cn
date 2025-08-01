---
lab:
  title: 实验室：在 Azure VM 上部署和配置 Windows Server
  type: Answer Key
  module: 'Module 6: Deploying and Configuring Azure VMs'
---

# 实验室解答：在 Azure VM 上部署和配置 Windows Server

## 练习 1：创作用于 Azure VM 部署的 Azure 资源管理器 (ARM) 模板

#### 任务 1：连接到你的 Azure 订阅并启用 Microsoft Defender for Cloud 增强的安全性

在此任务中，你将连接到你的 Azure 订阅并启用 Microsoft Defender for Cloud 增强的安全性。

1. 连接到 **SEA-ADM1**，然后根据需要使用讲师提供的凭据登录。
1. 在 **SEA-ADM1** 上，启动 Microsoft Edge，转到 Azure 门户`https://portal.azure.com`，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录。

>备注：如果你的 Azure 订阅中已启用了 Microsoft Defender for Cloud，请跳过此任务中的其余步骤，直接转到下一个任务。

1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择 Microsoft Defender for Cloud 。
1. 在“Microsoft Defender for Cloud \| 开始”页上，选择“升级”，然后选择“安装代理”  。

#### 任务 2：使用 Azure 门户生成 ARM 模板和参数文件

在此任务中，你将使用 Azure 门户创建资源组，并在资源组中创建一个磁盘。

1. 在 SEA-ADM1 上，在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“虚拟机”  。 在“虚拟机”页中，选择“+ 创建”，然后选择“Azure 虚拟机”  。
1. 在“创建虚拟机”页面中的“基本信息”选项卡上，指定以下设置并将所有其他设置保留为默认值，但不部署它 ：

   |设置|值|
   |---|---|
   |订阅|你将在此实验室中使用的 Azure 订阅的名称。|
   |资源组|新资源组 AZ800-L0601-RG 的名称|
   |虚拟机名称|**az800L06-vm0**|
   |区域|使用可以在其中预配 Azure 虚拟机的 Azure 区域的名称|
   |可用性选项|不需要基础结构冗余|
   |映像|Windows Server 2022 Datacenter：Azure Edition - Gen2|
   |使用 Azure Spot 折扣运行|否|
   |大小|**Standard_D2s_v3**|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|
   |公共入站端口|无|
   |是否要使用现有的 Windows Server 许可证|关闭|

1. 选择“下一步: 磁盘 >”，然后在“创建虚拟机”页面中的“磁盘”选项卡上，指定以下设置，同时将所有其他设置保留为默认值  ：

   |设置|Value|
   |---|---|
   |OS 磁盘类型|**标准 HDD**|

1. 选择“下一页:网络 >”，然后在“创建虚拟机”页中的“网络”选项卡上，选择“新建”超链接，此链接位于“虚拟网络”文本框之后   。
1. 在“创建虚拟网络”页面上，指定以下设置，同时将所有其他设置保留为默认值，然后选择“确定” ：

   |设置|值|
   |---|---|
   |名称|**az800L06-vnet**|
   |地址范围|10.60.0.0/20|
   |子网名称|subnet0|
   |子网范围|10.60.0.0/24|

1. 回到“创建虚拟机”页面上，在“网络”选项卡上，指定以下设置，同时将所有其他设置保留为默认值 ：

   |设置|值|
   |---|---|
   |公共 IP|无|
   |NIC 网络安全组|无|
   |启用加速网络|关闭|
   |负载均衡选项|无|

1. 选择“下一步: 管理 >”，保留所有设置的默认值。
   
1. 选择“下一步: 监视 >”，然后在“创建虚拟机”页面中的“监视”选项卡上，指定以下设置，同时将所有其他设置保留为默认值  ：

   |设置|值|
   |---|---|
   |启动诊断|使用托管存储帐户启用（推荐）|
   
1. 选择“下一步: 高级 >”，在“创建虚拟机”页的“高级”选项卡上，查看可用的设置，而不修改任何设置，然后选择“查看 + 创建”   。

   >备注：请勿创建虚拟机。 你将为此目的使用自动生成的模板。

#### 任务 3：从 Azure 门户下载 ARM 模板和参数文件

1. 在 Azure 门户中的“创建虚拟机”页面上，选择“下载自动化模板” 。
1. 在“模板”页面上，选择“下载” 。
1. 选择 template.zip 旁的省略号按钮，然后在弹出菜单中，选择“在文件夹中显示” 。 这将自动打开文件资源管理器，其中将显示 Downloads 文件夹的内容。
1. 在文件资源管理器中，复制 **template.zip**，转到目录 **C:\\** 并新建名为 **Lab06files** 的文件夹，打开文件夹并粘贴 **template.zip** 文件。
1. 在“模板”页面中，浏览回“创建虚拟机”页面，在不完成部署的情况下关闭它 。

## 练习 2：修改 ARM 模板以包括基于 VM 扩展的配置

#### 任务 1：查看用于 Azure VM 部署的 ARM 模板和参数文件

1. 在 **SEA-ADM1** 上，启动文件资源管理器，然后浏览到 **C:\\Lab06files** 文件夹。
1. 将 template.zip 文件的内容提取到同一文件夹中。
1. 在记事本中打开 template.json 文件，并查看其内容。 使记事本窗口保持打开状态。
1. 从文件资源管理器的记事本中，打开 **C:\\Lab06files\\parameters.json** 文件并查看其内容。
1. 关闭显示 parameters.json 文件的记事本窗口。

#### 任务 2：将 Azure VM 扩展部分添加到现有模板

1. 在 SEA-ADM1 上，在显示 template.json 文件内容的记事本窗口中，在 `    "resources": [` 行后直接插入以下代码 ：
   >**注意**：如果使用逐行粘贴代码的工具，IntelliSense 可能会添加多余的括号，从而导致验证错误。 可能需要先将代码粘贴到记事本，再将其粘贴到 JSON 文件中。

   ```json
      {
         "type": "Microsoft.Compute/virtualMachines/extensions",
         "name": "[concat(parameters('virtualMachineName'), '/customScriptExtension')]",
         "apiVersion": "2018-06-01",
         "location": "[resourceGroup().location]",
         "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', parameters('virtualMachineName'))]"
         ],
         "properties": {
            "publisher": "Microsoft.Compute",
            "type": "CustomScriptExtension",
            "typeHandlerVersion": "1.7",
            "autoUpgradeMinorVersion": true,
            "settings": {
                  "commandToExecute": "powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)"
            }
         }
      },
   ```
   >备注：请确保代码片段与脚本的其余部分保持一致。

1. 保存更改并关闭文件。

## 练习 3：使用 ARM 模板部署运行 Windows Server 的 Azure VM

#### 任务 1：使用 ARM 模板部署 Azure VM

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的浏览器窗口。
1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“部署自定义模板” 。
1. 在“自定义部署”页面中，选择“在编辑器中生成自己的模板” 。
1. 在“编辑模板”页面上，选择“上传文件”、上传在上一练习中编辑的模板文件 template.json，然后选择“保存”   。
1. 在“自定义部署”页面上，选择“编辑参数” 。
1. 在“编辑参数”页面上，选择“上传文件”、上传在上一练习中查看的参数文件 parameters.json，然后选择“保存”   。
1. 返回到“自定义部署”页，指定以下设置，并将其他设置保留为默认值：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|AZ800-L0601-RG|
   |区域|可以在其中预配 Azure VM 的 Azure 区域的名称|
   |管理员密码|**Pa55w.rd1234**|

1. 选择“查看 + 创建”，然后选择“创建”。

   >备注：部署可能需要大约 10 分钟的时间。

1. 验证部署是否已成功完成。

#### 任务 2：查看 Azure VM 部署的结果

1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“资源组” 。
1. 在“资源组”页面上，选择“AZ800-L0601-RG”条目 。
1. 在“AZ800-L0601-RG”页的“概述”页上，查看资源列表，其中包括 Azure VM“az800L06-vm0”  。
1. 在资源列表中，选择 Azure VM“az800L06-vm0”条目。 
1. 在“**az800L06-vm0**”页的“设置”选项卡下，选择“**扩展 + 应用程序**”，然后在扩展列表中验证是否已成功预配 **customScriptExtension**。
1. 浏览回“AZ800-L0601-RG”页面，然后在“设置”部分中，选择“部署”  。
1. 在“AZ800-L0601-RG \| 部署”页面上，选择“Microsoft.Template”链接 。
1. 在“Microsoft.Template \| 概述”页面上，选择“模板”，你会注意到这是用于部署的同一模板 。

## 练习 4：配置对运行 Windows Server 的 Azure VM 的管理访问权限

#### 任务 1：验证 Azure Microsoft Defender for Cloud 的状态

1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择 Microsoft Defender for Cloud 。
1. 在 Microsoft Defender for Cloud 的“概述”页面上，在左侧垂直菜单的“管理”部分中，选择“环境设置”  。 
1. 在“环境设置”页上，选择代表你的 Azure 订阅的条目。
1. 在“设置”\|“Defender 计划”页面上，确认已选中“启用所有 Microsoft Defender for Cloud 计划”磁贴********。 出现提示时跳过 Defender for API 计划****。 在顶部，选择“设置和监视”****。
1. 在“设置和监视”页面上，查看其他可用选项****。
1. 在“设置 \| Defender 计划”页上，选择“保存” 。

#### 任务 2：查看实时 VM 访问设置

1. 浏览回到 Microsoft Defender for Cloud 的“概述”页，然后在“云安全”部分中，选择“工作负载保护”  。
1. 在“Microsoft Defender for Cloud \| 工作负载保护”页上，选择“实时 VM 访问” 。
1. 在“实时 VM 访问”页面上，查看“已配置”、“未配置”和“不支持”选项卡   。

   >**注意**：新部署的 VM 可能需要最长 24 小时才能出现在“不支持”选项卡上****。此时不支持此 VM 的原因是实时 VM 访问需要网络安全组。 请继续进行下一个练习，不要等待。

## 练习 5：在 Azure VM 中配置 Windows Server 安全性

#### 任务 1：创建和配置 NSG

1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“网络安全组” 。
1. 在“网络安全组”页上，选择“+ 创建”。
1. 在“创建网络安全组”页的“基本信息”选项卡上，指定以下设置（将其他设置保留为默认值） ：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|AZ800-L0601-RG|
   |名称|**az800L06-vm0-nsg1**|
   |区域|在其中预配了 Azure VM **az800L06-vm0** 的 Azure 区域的名称|

1. 在“创建网络安全组”页的“基本信息”选项卡上，选择“查看 + 创建”，然后选择“创建”   。
1. 在 Azure 门户中，浏览回“AZ800-L0601-RG”页面，然后在资源列表中，选择代表新创建的网络安全组 az800L06-vm0-nsg1 的条目 。
1. 在“az800L06-vm0-nsg1”页面上，查看默认入站和出站安全规则的列表，然后在“设置”部分中，选择“入站安全规则”  。
1. 在“az800L06-vm0-nsg1 \| 入站安全规则”页上，选择“+ 添加” 。
1. 在“添加入站安全规则”页面上，指定以下设置，同时将所有其他设置保留为默认值，然后选择“添加” ：

   |设置|值|
   |---|---|
   |源|**任意**|
   |源端口范围|*|
   |目标|**任意**|
   |服务|**HTTP**|
   |操作|**允许**|
   |优先级|**300**|
   |名称|AllowHTTPInBound|

#### 任务 2：配置对 Azure VM 的入站 HTTP 访问

1. 在 Azure 门户的工具栏上，在“**搜索资源、服务和文档**”文本框中，搜索并选择“**公共 IP 地址**”。
1. 在“公共 IP 地址”**** 页上，选择“+ 创建”****。
1. 在“创建公共 IP 地址”**** 页的“基本信息”**** 选项卡上，请指定以下设置（将其他设置保留为默认值）：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|AZ800-L0601-RG|
   |名称|az800l06-vm0-pip1|
   |区域|在其中预配了 Azure VM az800l06-vm0 的 Azure 区域的名称|
   |SKU|**标准**|

1. 在“创建公共 IP 地址”**** 页的“基本信息”**** 选项卡上，选择“查看 + 创建”****，然后选择“创建”****。
1. 验证部署是否已成功完成。
1. 在 Azure 门户中，浏览回“AZ800-L0601-RG”页面，然后在资源列表中，选择代表 Azure VM az800l06-vm0 的条目 。
1. 在“**az800l06-vm0**”页上的“**网络**”菜单下，选择“**网络设置**”。
1. 在“**az800l06-vm0”\|“网络设置**”页上，选择用于指定附加到 **az800l06-vm0** 的网络接口链接。
1. 在显示网络接口属性的页面上，在左侧垂直菜单的“设置”部分中，选择“网络安全组” 。 
1. 在“网络安全组”页面的下拉列表中，选择“az800l06-vm0-nsg1”，然后选择“保存”  。
1. 返回到显示网络接口属性的页面上，选择“IP 配置”，然后选择“ipconfig1”条目 。
1. 在“**ipconfig1**”页的“**公共 IP 地址**”部分中，选择“**关联公共 IP 地址**”框，然后从“**公共 IP 地址**”下拉列表中，选择“**az800l06-vm0-pip1**”。
1. 在“ipconfig1”**** 页上，选择“保存”****。
1. 浏览回到显示网络接口属性的页面，选择“概述”。 请注意分配给接口的公共 IP 地址的值。
1. 打开另一个浏览器选项卡，浏览到该 IP 地址，验证网页是否已打开，其中是否显示了“来自 az800L06-vm0 的 Hello World”。
1. 从实验室计算机中，启动远程桌面应用，并尝试连接到同一 IP 地址。 验证连接是否失败。

   >备注：这是预期的结果，因为当前无法通过 TCP 端口 3389 从 Internet 访问 Azure VM。 只能通过 TCP 端口 80 访问它。

#### 任务 3：触发对 Azure VM 的 JIT 状态的重新评估

>备注：若要触发对 Azure VM 的 JIT 状态的重新评估，必须执行此任务。 默认情况下，这可能需要长达 24 小时的时间。

1. 在 Azure 门户中，浏览回“AZ800-L0601-RG”页面，然后在资源列表中，选择代表 Azure VM az800L06-vm0 的条目 。
1. 在“**az800L06-vm0**”页的“设置”部分下，选择“**配置**”。 
1. 在“az800L06-vm0 \| 配置”页上，选择“启用实时 VM 访问”，然后选择“打开 Microsoft Defender for Cloud”链接  。
1. 在“实时 VM 访问”页面上，验证代表 az800L06-vm0 Azure VM 的条目是否出现在“已配置”选项卡上  。

#### 任务 4：通过 JIT VM 访问连接到 Azure VM

1. 浏览回到“az800L06-vm0”页面，选择“连接”，然后在下拉列表中，选择“RDP”  。 
1. 在“az800L06-vm0 \| 连接”页面上的“源 IP”部分中，选择“我的 IP”，然后选择“请求访问权限”   。
1. 等待指示你的请求已获批准的通知，选择“下载 RDP 文件”，并按照提示连接到目标 Azure VM。
1. 当系统提示输入凭据时，请指定以下值，然后选择“确定”：

   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 验证是否可以通过远程桌面成功访问在 Azure VM 中运行的操作系统，并关闭远程桌面会话。

## 练习 6：取消预配 Azure 环境

#### 任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-ADM1 上显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Azure Cloud Shell 窗格。
1. 如果系统提示选择 Bash 或 PowerShell，请选择 PowerShell  。

   >备注：如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择在此实验室中使用的订阅，然后选择“创建存储”  。

#### 任务 2：标识实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出在此实验室过程中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*'
   ```
1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*' | Remove-AzResourceGroup -Force -AsJob
   ```
   >备注：该命令异步执行（由 -AsJob 参数确定）。 因此，尽管可以在同一 PowerShell 会话中立即运行另一个 PowerShell 命令，但在实际删除资源组之前将需要几分钟的时间。
