---
lab:
  title: 实验室：在混合场景中使用 Windows Admin Center
  module: 'Module 4: Facilitating hybrid management'
---

# <a name="lab-using-windows-admin-center-in-hybrid-scenarios"></a>实验室：在混合场景中使用 Windows Admin Center

## <a name="scenario"></a>场景

若要解决与一致操作和管理模型有关的问题，无论托管系统的位置如何，你都将在混合环境中测试 Windows Admin Center 的功能，混合环境中包含在本地和 Microsoft Azure 虚拟机 (VM) 中运行的不同版本的 Windows Server 操作系统。

你的目标是确认 Windows Admin Center 可通过一致的方式得到使用，而不考虑托管系统的位置。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 使用 Azure 网络适配器测试混合连接。
- 在 Azure 中部署 Windows Admin Center 网关。
- 在 Azure 中验证 Windows Admin Center 网关的功能。

## <a name="estimated-time-90-minutes"></a>估计时间：90 分钟

## <a name="lab-setup"></a>实验室设置

Virtual machines: <bpt id="p1">**</bpt>AZ-800T00A-SEA-DC1<ept id="p1">**</ept> and <bpt id="p2">**</bpt>AZ-800T00A-ADM1<ept id="p2">**</ept> must be running. Other VMs can be running, but they aren't required for this lab.

> 注意：AZ-800T00A-SEA-DC1 和 AZ-800T00A-SEA-ADM1 VM 托管 SEA-DC1 和 SEA-ADM1 的安装    

1. 选择“SEA-ADM1”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

For this lab, you'll use the available VM environment and an Azure subscription. Before you begin the lab, ensure that you have an Azure subscription and a user account with the Owner or Contributor role in that subscription, as well as with the Global Administrator role in the Azure Active Directory (Azure AD) tenant associated with that subscription.

## <a name="exercise-1-provisioning-azure-vms-running-windows-server"></a>练习 1：预配运行 Windows Server 的 Azure VM

### <a name="scenario"></a>场景

You need to verify that you can establish hybrid connectivity between an on-premises server and an Azure virtual network. To start, you'll provision Azure VMs running Windows Server by using an Azure Resource Manager template.

此练习的主要任务如下：

1. 使用 Azure 资源管理器模板创建 Azure 资源组。
1. 使用 Azure 资源管理器模板创建 Azure VM。

#### <a name="task-1-create-an-azure-resource-group-by-using-an-azure-resource-manager-template"></a>任务 1：使用 Azure 资源管理器模板创建 Azure 资源组

1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到 Azure 门户，并使用 Azure 凭据进行身份验证。
1. 在 Azure 门户的 Cloud Shell 窗格中打开一个 PowerShell 会话。
1. 将 C:\\Labfiles\\Lab04\\L04-sub_template.json 文件上传到 Cloud Shell 主目录。
1. From the Cloud Shell pane, run the following commands to create a resource group that will contain resources you provision in this lab. (Replace the <ph id="ph1">`&lt;Azure region&gt;`</ph> placeholder with the name of an Azure region into which you can deploy Azure virtual machines, such as <bpt id="p1">**</bpt>eastus<ept id="p1">**</ept>.)

   ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: This lab has been tested and verified using East US, so you should use that region. In general, to identify Azure regions where you can provision Azure VMs, refer to <bpt id="p1">[</bpt>Find Azure credit offers in your region<ept id="p1">](https://aka.ms/regions-offers)</ept>.

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

#### <a name="task-2-create-an-azure-vm-by-using-an-azure-resource-manager-template"></a>任务 2：使用 Azure 资源管理器模板创建 Azure VM

1. 从 Cloud Shell 窗格中，上传 Azure 资源管理器模板 C:\\Labfiles\\Lab04\\L04-rg_template.json 和相应的 Azure 资源管理器参数文件 C:\\Labfiles\\Lab04\\L04-rg_template.parameters.json 。
1. 从 Cloud Shell 窗格中，运行以下命令以部署运行 Windows Server 的 Azure VM，你将在此实验室中使用它：

   ```powershell
   New-AzResourceGroupDeployment `
     -Name az800l04rgDeployment `
     -ResourceGroupName $rgName `
     -TemplateFile $HOME/L04-rg_template.json `
     -TemplateParameterFile $HOME/L04-rg_template.parameters.json
   ```

   ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait for the deployment to complete before you proceed to the next exercise. The deployment should take about 5 minutes.

1. 在 Azure 门户中，关闭 Cloud Shell 窗格。
1. 在 Azure 门户中，将 IP 地址范围为 10.4.3.224/27 的 GatewaySubnet 添加到 az800l04-vnet 虚拟网络  。

## <a name="exercise-2-implementing-hybrid-connectivity-by-using-the-azure-network-adapter"></a>练习 2：使用 Azure 网络适配器实现混合连接

### <a name="scenario"></a>场景

You need to verify that you can establish hybrid connectivity between an on-premises server and the Azure VM you provisioned in the previous exercise. You'll use the Azure Network Adapter feature of Windows Admin Center for this purpose.

此练习的主要任务如下：

1. 向 Azure 注册 Windows Admin Center。
1. 创建 Azure 网络适配器。

#### <a name="task-1-register-windows-admin-center-with-azure"></a>任务 1：向 Azure 注册 Windows Admin Center

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，运行以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait until the installation completes. This should take about 2 minutes.

1. 在 SEA-ADM1 上，启动 Microsoft Edge 并连接到 Windows Admin Center 的本地实例 (`https://SEA-ADM1.contoso.com`)。 

   ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: If the link does not work, on <bpt id="p2">**</bpt>SEA-ADM1<ept id="p2">**</ept>, browse to the <bpt id="p3">**</bpt>WindowsAdminCenter.msi<ept id="p3">**</ept> file, open the context menu for it, and then select <bpt id="p4">**</bpt>Repair<ept id="p4">**</ept>. After the repair completes, refresh Microsoft Edge. 

1. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 从 Windows Admin Center 页中，尝试添加 Azure 网络适配器。
1. 出现提示时，将 Windows Admin Center 注册到你在上一练习中使用的 Azure 订阅。

#### <a name="task-2-create-an-azure-network-adapter"></a>任务 2：创建 Azure 网络适配器

1. 在 SEA-ADM1 上，在显示 Windows Admin Center 的 Microsoft Edge 窗口中，尝试再次创建 Azure 网络适配器。
1. 创建具有以下设置的 Azure 网络适配器：

   |设置|值|
   |---|---|
   |订阅|你在此实验室中使用的 Azure 订阅的名称|
   |位置|eastus|
   |虚拟网络|az800l04-vnet|
   |网关子网|10.4.3.224/27|
   |网关 SKU|VpnGw1|
   |客户端地址空间|192.168.0.0/24|
   |身份验证证书|自动生成的自签名根证书和客户端证书|

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，并确认将预配名称以 WAC-Created-vpngw- 开头的新虚拟网络网关 。

   ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The provisioning of the Azure virtual network gateway can take up to 45 minutes. Do not wait for the provisioning to complete but instead proceed to the next exercise.

## <a name="exercise-3-deploying-windows-admin-center-gateway-in-azure"></a>练习 3：在 Azure 中部署 Windows Admin Center 网关

### <a name="scenario"></a>场景

You need to evaluate the ability to manage Azure VMs running Windows Server OS by using Windows Admin Center. To accomplish this, you'll first install a Windows Admin Center gateway in the Azure virtual network you implemented in the first exercise of this lab.

此练习的主要任务如下：

1. 在 Azure 中安装 Windows Admin Center 网关。
1. 查看脚本预配的结果。

#### <a name="task-1-install-windows-admin-center-gateway-in-azure"></a>任务 1：在 Azure 中安装 Windows Admin Center 网关

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的浏览器窗口。
1. 在 Azure 门户的 Cloud Shell 窗格中启动一个 PowerShell 会话。
1. 从 Cloud Shell 窗格中，将 C:\\Labfiles\\Lab04\\Deploy-WACAzVM.ps1 文件上传到 Cloud Shell 主目录中。
1. 从 Cloud Shell 窗格中，运行以下命令，启用 Windows Admin Center 预配脚本使用的 AzureRm PowerShell cmdlet 的兼容性：

   ```powershell
   Enable-AzureRmAlias -Scope Process
   ```
1. 运行以下命令，设置运行 Windows Admin Center 预配脚本必需的变量值（将 `<Azure region>` 占位符替换为你之前在此实验室中向其部署资源的 Azure 区域的名称，例如 eastus）：

   ```powershell
   $rgName = 'AZ800-L0401-RG'
   $vnetName = 'az800l04-vnet'
   $nsgName = 'az800l04-web-nsg'
   $subnetName = 'subnet1'
   $location = '<Azure region>'
   $pipName = 'wac-public-ip'
   $size = 'Standard_D2s_v3'
   ```
1. 运行以下命令以设置脚本参数变量：

   ```powershell
   $scriptParams = @{
     ResourceGroupName = $rgName
     Name = 'az800l04-vmwac'
     VirtualNetworkName = $vnetName
     SubnetName = $subnetName
     GenerateSslCert = $true
     size = $size
     PublicIPAddressName = $pipname
   }
   ```
1. 运行以下命令以禁用 PowerShell 远程处理的证书验证（在第一个命令后出现提示时，输入 A 并按 Enter 键）：

   ```powershell
   install-module pswsman
   Disable-WSManCertVerification -All
   ```
1. 运行以下命令以启动预配脚本：

   ```powershell
   ./Deploy-WACAzVM.ps1 @scriptParams
   ```
1. 系统提示提供本地管理员帐户的名称时，请输入“Student”。
1. 系统提示提供本地管理员帐户的密码时，请输入“Pa55w.rd1234”。

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait for the provisioning script to complete. This might take about 5 minutes.

1. 确认脚本已成功完成，并注意最后的消息提供的 URL 包含托管 Windows Admin Center 安装的 Azure VM 的完全限定名称。

   ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Record the fully qualified name of the Azure VM. You will need it later in this lab.

1. 关闭 Cloud Shell 窗格。

#### <a name="task-2-review-results-of-the-script-provisioning"></a>任务 2：查看脚本预配的结果

1. 在 Azure 门户中，浏览到 AZ800-L0401-RG 资源组的页面。
1. 在 AZ800-L0401-RG 页的“概述”页上，查看资源列表，其中包括 Azure VM az800l04-vmwac  。
1. 在“az800l04-vmwac”|“网络”页上的“入站端口规则”选项卡上，注意表示允许 TCP 端口 5986 上的连接的入站端口规则和允许 TCP 端口 443 上的连接的入站规则的条目 。

## <a name="exercise-4-verifying-functionality-of-the-windows-admin-center-gateway-in-azure"></a>练习 4：在 Azure 中验证 Windows Admin Center 网关的功能

### <a name="scenario"></a>场景

准备好所有必需的组件后，测试面向部署到你在本实验室的第一个练习中预配的 Azure 虚拟网络中的 Azure VM 的 WAC 功能。

此练习的主要任务如下：

1. 连接到在 Azure VM 中运行的 Windows Admin Center 网关。
1. 在 Azure VM 上启用 PowerShell 远程处理。
1. 使用 Azure VM 中运行的 Windows Admin Center 网关连接到 Azure VM。

#### <a name="task-1-connect-to-the-windows-admin-center-gateway-running-in-azure-vm"></a>任务 1：连接到在 Azure VM 中运行的 Windows Admin Center 网关

1. 在 SEA-ADM1 上，启动 Microsoft Edge 并连接在上一练习中确定的 Azure VM 中运行的 Windows Admin Center 网关。
1. 出现提示时，使用“Student”用户名和“Pa55w.rd1234”密码登录 。
1. 在 Windows Admin Center 的“所有连接”窗格中，选择“az800l04-vmwac [网关]”。
1. 检查 Windows Admin Center 的“概述”窗格。

#### <a name="task-2-enable-powershell-remoting-on-an-azure-vm"></a>任务 2：在 Azure VM 上启用 PowerShell 远程处理

1. 在 SEA-ADM1 上显示 Azure 门户的 Microsoft Edge 窗口中，浏览到 Azure VM“az800l04-vm0”的页面 。
1. 从“az800l04-vm0”页中，使用“运行命令”功能运行以下命令，以启用 Windows 远程管理（若已禁用它） ：

   ```powershell
   winrm quickconfig -quiet
   ```

1. 使用“运行命令”功能运行以下命令，以打开 Windows 远程管理入站端口：

   ```powershell
   Set-NetFirewallRule -Name WINRM-HTTP-In-TCP-PUBLIC -RemoteAddress Any
   ```

1. 使用“运行命令”功能运行以下命令，以启用 PowerShell 远程处理：

   ```powershell
   Enable-PSRemoting -Force -SkipNetworkProfileCheck
   ```

#### <a name="task-3-connect-to-an-azure-vm-by-using-the-windows-admin-center-gateway-running-in-azure-vm"></a>任务 3：使用 Azure VM 中运行的 Windows Admin Center 网关连接到 Azure VM

1. 在 SEA-ADM1 上，在显示在 Azure VM az800l04-vmwac 上运行的 Windows Admin Center 网关界面的 Microsoft Edge 窗口中，通过使用 Azure VM az800l04-vm0 的名称来添加与其的连接  。
1. 出现提示时，使用“Student”用户名和“Pa55w.rd1234”密码进行身份验证 。
1. 成功连接到该 VM 后，在 Windows Admin Center 中检查 Azure VM az800l04-vmwac 的“概述”窗格。

## <a name="exercise-5-deprovisioning-the-azure-environment"></a>练习 5：取消预配 Azure 环境

### <a name="scenario"></a>场景

为了最大限度地减少与 Azure 相关的费用，要取消预配整个实验室中预配的 Azure 资源。

此练习的主要任务如下：

1. 在 Cloud Shell 中启动一个 PowerShell 会话。
1. 标识实验室中预配的所有 Azure 资源。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口。
1. 从 Azure 门户的 Cloud Shell 窗格中打开一个 PowerShell 会话。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>任务 2：标识实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出在此实验室过程中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'az800l04*'
   ```

1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'az800l04*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >虚拟机：AZ-800T00A-SEA-DC1 和 AZ-800T00A-ADM1 必须正在运行 。

### <a name="results"></a>结果

完成此实验室后，你已经以满足 Contoso 的可管理性和安全性要求的方式部署并配置了运行 Windows Server 的 Azure VM。

### <a name="prepare-for-the-next-module"></a>为下一个模块做准备

完成了下一个模块的准备时，结束本实验室。
