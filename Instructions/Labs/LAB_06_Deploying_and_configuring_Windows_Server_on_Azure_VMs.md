---
lab:
  title: 实验室：在 Azure VM 上部署和配置 Windows Server
  module: 'Module 6: Deploying and Configuring Azure VMs'
ms.openlocfilehash: bbf7d0657532d3b162ac8c366cc37da11ae3c32c
ms.sourcegitcommit: d34dce53481b0263d0ff82913b3f49cb173d5c06
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/09/2022
ms.locfileid: "147039431"
---
# <a name="lab-deploying-and-configuring-windows-server-on-azure-vms"></a>实验室：在 Azure VM 上部署和配置 Windows Server

## <a name="scenario"></a>场景

你需要解决有关当前基础结构的问题。 你有一个过时的运营模型（有限使用自动化），信息安全团队关注应该应用于运行基于 Windows Server 的工作负载的 Azure VM 的其他控制措施。 你已决定为运行 Windows Server 的 Azure VM 开发并实现自动部署和配置过程。

此过程将涉及 Azure 资源管理器 (ARM) 模板和通过 Azure VM 扩展的 OS 配置。 它还会纳入其他安全保护机制（不仅仅是已应用于本地系统的机制），例如通过 AppLocker 的应用程序允许列表、文件完整性检查和自适应网络/DDoS 保护。 你还将利用 JIT 功能，将 Azure VM 的管理访问权限限制为与伦敦总部关联的公共 IP 地址范围。

你的目标是以满足可管理性和安全性要求的方式部署和配置运行 Windows Server 的 Azure VM。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 创作用于 Azure VM 部署的 ARM 模板。
- 修改 ARM 模板以包括基于 VM 扩展的配置。
- 使用 ARM 模板部署运行 Windows Server 的 Azure VM。
- 配置对运行 Windows Server 的 Azure VM 的管理访问权限。
- 在 Azure VM 中配置 Windows Server 安全性。
- 取消预配 Azure 环境。

## <a name="estimated-time-90-minutes"></a>估计时间：90 分钟

## <a name="lab-setup"></a>实验室设置

虚拟机：AZ-800T00A-SEA-DC1 和 AZ-800T00A-ADM1 必须正在运行 。 其他 VM 可以运行，但此实验室不需要这些 VM。

> 注意：AZ-800T00A-SEA-DC1 和 AZ-800T00A-SEA-ADM1 虚拟机托管 SEA-DC1 和 SEA-ADM1 的安装    。

1. 选择“SEA-ADM1”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境和 Azure 订阅。 在开始实验室之前，请确保拥有 Azure 订阅以及具有该订阅中“所有者”或“参与者”角色的用户帐户。

## <a name="exercise-1-authoring-arm-templates-for-azure-vm-deployment"></a>练习 1：创作用于 Azure VM 部署的 ARM 模板

### <a name="scenario"></a>场景

为了简化基于 Azure 的操作，你决定开发并实现 Windows Server 到 Azure VM 的自动部署和配置过程。 你的部署需要符合信息安全团队的要求，并遵守 Contoso, Ltd. 所期望的目标运营模型，包括高可用性。

此练习的主要任务如下：

1. 连接到你的 Azure 订阅并启用 Microsoft Defender for Cloud 增强的安全性。
1. 使用 Azure 门户生成 ARM 模板和参数文件。
1. 从 Azure 门户下载 ARM 模板和参数文件。

#### <a name="task-1-connect-to-your-azure-subscription-and-enable-enhanced-security-of-microsoft-defender-for-cloud"></a>任务 1：连接到你的 Azure 订阅并启用 Microsoft Defender for Cloud 增强的安全性

在此任务中，你将连接到你的 Azure 订阅并启用 Microsoft Defender for Cloud 增强的安全性。

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\\Administrator 的身份使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，启动 Microsoft Edge，转到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录。

>备注：如果你的 Azure 订阅中已启用了 Microsoft Defender for Cloud，请跳过此任务中的其余步骤，直接转到下一个任务。

1. 在 Azure 门户中，浏览到“Microsoft Defender for Cloud”页面。
1. 在“Microsoft Defender for Cloud \| 入门”页上，启用 Microsoft Defender for Cloud 增强的安全性，并启用 Microsoft Defender for Cloud 自动代理安装。

#### <a name="task-2-generate-an-arm-template-and-parameters-files-by-using-the-azure-portal"></a>任务 2：使用 Azure 门户生成 ARM 模板和参数文件

1. 在 Azure 门户中，逐步完成使用以下设置创建新的 Azure VM 的过程（将所有其他项保留默认值），但不要部署它：

   |设置|值|
   |---|---|
   |订阅|你将在此实验室中使用的 Azure 订阅的名称。|
   |资源组|新资源组 AZ800-L0601-RG 的名称|
   |虚拟机名称|az800l06-vm0|
   |区域|使用可以在其中预配 Azure 虚拟机的 Azure 区域的名称|
   |可用性选项|没有所需的基础结构冗余|
   |映像|Windows Server 2022 Datacenter：Azure Edition - Gen2|
   |Azure Spot 实例|否|
   |大小|**Standard_D2s_v3**|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|
   |公共入站端口|无|
   |是否要使用现有的 Windows Server 许可证|否|
   |OS 磁盘类型|**标准 HDD**|
   |名称|az800l06-vnet|
   |地址范围|10.60.0.0/20|
   |子网名称|subnet0|
   |子网范围|10.60.0.0/24|
   |公共 IP|无|
   |NIC 网络安全组|无|
   |加速网络|关|
   |是否将此虚拟机置于现有负载均衡解决方案之后？|否|
   |启动诊断|使用托管存储帐户启用（推荐）|

1. 当你到达“创建虚拟机”页的“查看和创建”选项卡时，请继续执行任务 3 。

   >备注：请勿创建虚拟机。 你将为此目的使用自动生成的模板。

#### <a name="task-3-download-the-arm-template-and-parameters-files-from-the-azure-portal"></a>任务 3：从 Azure 门户下载 ARM 模板和参数文件

1. 从“创建虚拟机”页  的“查看和创建”选项卡中，下载自动化模板，并将其复制到实验室 VM 上的 C:\\Labfiles\\Mod06 文件夹。
1. 在 Azure 门户中，关闭“创建虚拟机”页。

## <a name="exercise-2-modifying-arm-templates-to-include-vm-extension-based-configuration"></a>练习 2：修改 ARM 模板以包括基于 VM 扩展的配置

### <a name="scenario"></a>场景

除了自动部署 Azure 资源外，你还需要确保可以自动配置在 Azure VM 中运行的 Windows Server。 若要实现此目的，需要测试 Azure 自定义脚本扩展的使用。

此练习的主要任务如下：

1. 查看用于 Azure VM 部署的 ARM 模板和参数文件。
1. 将 Azure VM 扩展部分添加到现有模板。

#### <a name="task-1-review-the-arm-template-and-parameters-files-for-azure-vm-deployment"></a>任务 1：查看用于 Azure VM 部署的 ARM 模板和参数文件

1. 将已下载的存档的内容提取到 **C:\\Labfiles\\Mod06** 文件夹。
1. 在记事本中打开 template.json 文件，并查看内容。 使记事本窗口保持打开状态。
1. 在记事本中打开 **C:\\Labfiles\\Mod06\\parameters.json** 文件，查看该文件，然后关闭记事本窗口。

#### <a name="task-2-add-an-azure-vm-extension-section-to-the-existing-template"></a>任务 2：将 Azure VM 扩展部分添加到现有模板

1. 在实验室 VM 上，在显示 template.json 文件内容的记事本窗口中，在 `    "resources": [` 行下直接插入以下代码：

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
1. 保存更改并关闭文件。

## <a name="exercise-3-deploying-azure-vms-running-windows-server-by-using-arm-templates"></a>练习 3：使用 ARM 模板部署运行 Windows Server 的 Azure VM

### <a name="scenario"></a>场景

配置 ARM 模板后，你将通过在概念证明 Azure 订阅中执行部署来验证其功能。

此练习的主要任务如下：

1. 使用 ARM 模板部署 Azure VM。
1. 查看 Azure VM 部署的结果。

#### <a name="task-1-deploy-an-azure-vm-by-using-an-arm-template"></a>任务 1：使用 ARM 模板部署 Azure VM

1. 在 Azure 门户的“SEA-ADM1”中，浏览到“自定义部署”页，然后选择“在编辑器中生成自己的模板”选项  。
1. 将模板文件和参数文件加载到“自定义部署”页。
1. 部署具有以下设置的模板，并将所有其他设置保留为其默认值：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|AZ800-L0601-RG|
   |区域|可以在其中预配 Azure VM 的 Azure 区域的名称|
   |管理员密码|**Pa55w.rd1234**|

   >备注：部署可能需要大约 10 分钟的时间。

1. 验证部署是否已成功完成。

#### <a name="task-2-review-results-of-the-azure-vm-deployment"></a>任务 2：查看 Azure VM 部署的结果

1. 在 Azure 门户中，浏览到 AZ800-L0601-RG 资源组页并查看其资源列表，尤其是 Azure VM az800l06-vm0 。
1. 浏览到 az800l06-vm0 Azure VM 页面，并验证是否已成功预配 customScriptExtension 。
1. 浏览回 AZ800-L0601-RG 资源组页面，查看其部署，并查看用于部署它的 Microsoft.Template 以确认它与用于部署的模板匹配 。

## <a name="exercise-4-configuring-administrative-access-to-azure-vms-running-windows-server"></a>练习 4：配置对运行 Windows Server 的 Azure VM 的管理访问权限

### <a name="scenario"></a>场景

运行 Windows Server 的 Azure VM 到位后，你需要测试是否能够从本地管理工作站远程管理它们。

此练习的主要任务如下：

1. 验证 Azure Microsoft Defender for Cloud 的状态。
1. 查看实时 VM 访问设置。

#### <a name="task-1-verify-the-status-of-azure-microsoft-defender-for-cloud"></a>任务 1：验证 Azure Microsoft Defender for Cloud 的状态

1. 在 Azure 门户中，浏览到“Microsoft Defender for Cloud”页面。
1. 验证是否已启用 Microsoft Defender for Cloud 的增强安全功能。

#### <a name="task-2-review-the-just-in-time-vm-access-settings"></a>任务 2：查看实时 VM 访问设置

1. 在 Azure 门户中，浏览到 **"Microsoft Defender for Cloud \| 工作负载保护"** 页面，并查看 **“实时 VM 访问”** 设置。
1. 在“实时 VM 访问”页面上，查看“已配置”、“未配置”和“不支持”选项卡   。

   >备注：新部署的 VM 可能需要长达 24 小时才能出现在“不支持”选项卡上。在此期间，与其等待，不如继续下一个练习 。

## <a name="exercise-5-configuring-windows-server-security-in-azure-vms"></a>练习 5：在 Azure VM 中配置 Windows Server 安全性

### <a name="scenario"></a>场景

运行 Windows Server 的 Azure VM 到位后，你需要测试是否能够从本地管理工作站远程管理它们。

此练习的主要任务如下：

1. 创建和配置 NSG。
1. 配置对 Azure VM 的入站 HTTP 访问。
1. 触发对 Azure VM 的 JIT 状态的重新评估。
1. 通过 JIT VM 访问连接到 Azure VM。

#### <a name="task-1-create-and-configure-an-nsg"></a>任务 1：创建和配置 NSG

1. 在 Azure 门户中，使用以下设置创建 NSG，并将所有其他设置保留为其默认值：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |资源组|AZ800-L0601-RG|
   |名称|az800l06-vm0-nsg1|
   |区域|在其中预配了 Azure VM az800l06-vm0 的 Azure 区域的名称|

1. 使用以下设置将入站安全规则添加到新创建的网络安全组，并将所有其他设置保留为其默认值：

   |设置|值|
   |---|---|
   |源|**任意**|
   |源端口范围|*|
   |目标|**任意**|
   |服务|**HTTP**|
   |操作|**允许**|
   |优先级|**300**|
   |名称|AllowHTTPInBound|

#### <a name="task-2-configure-inbound-http-access-to-an-azure-vm"></a>任务 2：配置对 Azure VM 的入站 HTTP 访问

1. 在 Azure 门户中，浏览到连接到 az800l06-vm0 Azure VM 的网络接口的页面，并将其与在上一任务中创建的网络安全组关联。
1. 在 Azure 门户中，浏览到连接到 az800l06-vm0 Azure VM 的网络接口的 IP 配置，并将其与具有以下设置的新的公共 IP 地址相关联，并将所有其他设置保留为其默认值：

   |设置|值|
   |---|---|
   |名称|az800l06-vm0-pip1|
   |SKU|**标准**|

1. 从实验室 VM，打开浏览器选项卡，浏览到新创建的公共 IP 地址，并验证该页是否显示以下消息“Hello World from az800l06-vm0”。
1. 从实验室 VM，尝试建立与同一 IP 地址的远程桌面连接，并验证连接尝试失败。

   >备注：这是预期的结果，因为当前无法通过 TCP 端口 3389 从 Internet 访问 Azure VM。 只能通过 TCP 端口 80 访问它。

#### <a name="task-3-trigger-re-evaluation-of-the-jit-status-of-an-azure-vm"></a>任务 3：触发对 Azure VM 的 JIT 状态的重新评估

>备注：若要触发对 Azure VM 的 JIT 状态的重新评估，必须执行此任务。 默认情况下，这可能需要长达 24 小时的时间。

1. 在 Azure 门户中，浏览回 az800l06-vm0 页。
1. 在“az800l06-vm0”页上，选择“配置” 。 
1. 在“az800l06-vm0 \| 配置”页上，选择“启用实时 VM 访问”，然后选择“打开 Azure 安全中心”链接  。
1. 在“实时 VM 访问”页面上，验证代表 az800l06-vm0 Azure VM 的条目是否出现在“已配置”选项卡上  。

#### <a name="task-4-connect-to-the-azure-vm-via-jit-vm-access"></a>任务 4：通过 JIT VM 访问连接到 Azure VM

1. 在 Azure 门户的 az800l06-vm0 页面中，请求 JIT VM 访问权限。
1. 请求批准后，启动与目标 Azure VM 的远程桌面会话。
1. 当系统提示输入凭据时，请指定以下值：
   
   |设置|值|
   |---|---|
   |用户名|**学生**|
   |密码|**Pa55w.rd1234**|

1. 验证是否可以通过远程桌面成功访问在 Azure VM 中运行的操作系统，并关闭远程桌面会话。

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>练习 6：取消预配 Azure 环境

### <a name="scenario"></a>场景

为了最大限度地减少与 Azure 相关的费用，需要取消预配整个实验室中预配的 Azure 资源。

此练习的主要任务如下：

1. 在 Cloud Shell 中启动一个 PowerShell 会话。
1. 标识实验室中预配的所有 Azure 资源。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 从 Azure 门户的 Azure Cloud Shell 窗格中打开一个 PowerShell 会话。
1. 如果这是第一次启动 Cloud Shell，请接受其默认设置。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>任务 2：标识实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出在此实验室过程中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*'
   ```
1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L06*' | Remove-AzResourceGroup -Force -AsJob
   ```

## <a name="results"></a>结果

完成此实验室后，你已经以满足 Contoso, Ltd. 的可管理性和安全性要求的方式部署并配置了运行 Windows Server 的 Azure VM。
