---
lab:
  title: 实验室：实现混合网络基础结构
  module: 'Module 8: Implementing Windows Server IaaS VM networking'
---

# <a name="lab-implementing-hybrid-networking-infrastructure"></a>实验室：实现混合网络基础结构

## <a name="scenario"></a>场景

You were tasked with building a test environment in Azure, consisting of Microsoft Azure virtual machines deployed into separate virtual networks configured in the hub and spoke topology. This testing must include implementing connectivity between spokes by using user-defined routes that force traffic to flow via the hub. You also need to implement DNS name resolution for Azure virtual machines between virtual networks by using Azure private DNS zones and evaluate the use of Azure DNS zones for external name resolution.

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 在 Azure 中实现虚拟网络路由
- 在 Azure 中实现 DNS 名称解析
- 取消预配 Azure 环境

## <a name="estimated-time-60-minutes"></a>估计时间：60 分钟

## <a name="lab-setup"></a>实验室设置

Virtual machines: <bpt id="p1">**</bpt>AZ-800T00A-SEA-DC1<ept id="p1">**</ept> and <bpt id="p2">**</bpt>AZ-800T00A-ADM1<ept id="p2">**</ept> must be running. Other VMs can be running, but they aren't required for this lab.

> **注意**：AZ-800T00A-SEA-DC1 和 AZ-800T00A-SEA-ADM1 虚拟机托管 SEA-DC1 和 SEA-ADM1 的安装   

1. 选择“SEA-ADM1”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

For this lab, you'll use the available VM environment and an Azure subscription. Before you begin the lab, ensure that you have an Azure subscription and a user account with the Owner or Contributor role in that subscription.

><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: This lab, by default, requires a total of 6 vCPUs available in the Standard_Dsv3 series in the region you choose for deployment because it involves deployment of three Azure VMs of Standard_D2s_v3 SKU. If you are using a free Azure account, with the limit of 4 vCPUs, you can use a VM size that requires only one vCPU (such as Standard_B1s).

### <a name="exercise-1-implement-virtual-network-routing-in-azure"></a>练习 1：在 Azure 中实现虚拟网络路由

### <a name="scenario"></a>场景

首先使用 Azure 资源管理器模板部署核心网络基础结构，在该基础结构中配置自定义路由，并验证其功能。 

此练习的主要任务如下：

1. 预配实验室基础结构资源
1. 配置中心辐射型网络拓扑
1. 测试虚拟网络对等互连的传递性
1. 在中心辐射型拓扑中配置路由

#### <a name="task-1-provision-lab-infrastructure-resources"></a>任务 1：预配实验室基础结构资源

你的任务是在 Azure 中构建一个测试环境，其中包括部署到在中心辐射型拓扑中配置的单独虚拟网络中的 Microsoft Azure 虚拟机。

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，启动 Microsoft Edge，浏览到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录 。
1. 在 Azure 门户的 Cloud Shell 窗格中打开一个 PowerShell 会话。
1. 将 C:\\Labfiles\\Lab08\\L08-rg_template.json 和 C:\\Labfiles\\Lab08\\L08-rg_template.parameters.json 文件加载到 Cloud Shell 主目录 。
1. 在 Cloud Shell 窗格中，运行以下命令以创建将托管实验室环境的第一个资源组（将占位符 `<Azure_region>` 替换为要用于部署的 Azure 区域的名称）：

   >**注意**：可运行 (Get-AzLocation).Location 命令列出可用 Azure 区域的名称：

    ```powershell 
    $location = '<Azure_region>'
    $rgName = 'AZ800-L0801-RG'
    New-AzResourceGroup -Name $rgName -Location $location
    ```
1. 在 Cloud Shell 窗格中，运行以下命令，通过使用上传的模板和参数文件，在其中创建三个虚拟网络和四个 Azure VM：

   ```powershell
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/L08-rg_template.json `
      -TemplateParameterFile $HOME/L08-rg_template.parameters.json
   ```

    >此测试必须包括通过使用强制流量通过中心流动的用户定义的路由来实现分支之间的连接。

1. 在 Cloud Shell 窗格中，运行以下命令，在上一步中部署的 Azure VM 上安装网络观察程序扩展：

   ```powershell
   $rgName = 'AZ800-L0801-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $rgName).location
   $vmNames = (Get-AzVM -ResourceGroupName $rgName).Name

   foreach ($vmName in $vmNames) {
     Set-AzVMExtension `
     -ResourceGroupName $rgName `
     -Location $location `
     -VMName $vmName `
     -Name 'networkWatcherAgent' `
     -Publisher 'Microsoft.Azure.NetworkWatcher' `
     -Type 'NetworkWatcherAgentWindows' `
     -TypeHandlerVersion '1.4'
   }
   ```

    >你还需要使用 Azure 专用 DNS 区域为虚拟网络之间的 Azure 虚拟机实现 DNS 名称解析，并对使用 Azure DNS 区域进行外部名称解析作出评估。

#### <a name="task-2-configure-the-hub-and-spoke-network-topology"></a>任务 2：配置中心辐射型网络拓扑

此任务将在之前任务中部署的虚拟网络之间配置本地对等互连，以创建中心辐射型网络拓扑。

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开另一个选项卡，浏览到 [Azure 门户](https://portal.azure.com) 。
1. 在 Azure 门户中，浏览到 az800l08-vnet0 虚拟网络页面。
1. 在 az800l08-vnet0 虚拟网络页面上，使用以下设置创建一个对等互连（保留其余设置为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 本虚拟网络: 对等互连链接名称 | **az800l08-vnet0_to_az800l08-vnet1** |
    | 到远程虚拟网络的流量 | **允许（默认）** |
    | 从远程虚拟网络转接的流量 | **允许（默认）** |
    | 虚拟网络网关或路由服务器 | **“无”（默认）** |
    | 远程虚拟网络: 对等互连链接名称 | **az800l08-vnet1_to_az800l08-vnet0** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 远程虚拟网络：虚拟网络 | **az800l08-vnet1** |
    | 到远程虚拟网络的流量 | **允许（默认）** |
    | 从远程虚拟网络转接的流量 | **允许（默认）** |
    | 虚拟网络网关 | **“无”（默认）** |

    >**注意**：请等待操作完成。

    >**注意**：此步骤将建立两个对等互连：一个从 az800l08-vnet0 到 az800l08-vnet1，另一个从 az800l08-vnet1 到 az800l08-vnet0   。

    >**注意**：需要启用“允许转发的流量”，以便于在此实验室稍后将实现的分支虚拟网络之间进行路由。

1. 重复上述步骤，添加具有以下设置的对等互连（将其余设置保留为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 本虚拟网络: 对等互连链接名称 | **az800l08-vnet0_to_az800l08-vnet2** |
    | 到远程虚拟网络的流量 | **允许（默认）** |
    | 从远程虚拟网络转接的流量 | **允许（默认）** |
    | 虚拟网络网关 | **“无”（默认）** |
    | 远程虚拟网络: 对等互连链接名称 | **az800l08-vnet2_to_az800l08-vnet0** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 远程虚拟网络：虚拟网络 | **az800l08-vnet2** |
    | 到远程虚拟网络的流量 | **允许（默认）** |
    | 从远程虚拟网络转接的流量 | **允许（默认）** |
    | 虚拟网络网关 | **“无”（默认）** |

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: This step establishes two peerings - one from <bpt id="p2">**</bpt>az800l08-vnet0<ept id="p2">**</ept> to <bpt id="p3">**</bpt>az800l08-vnet2<ept id="p3">**</ept> and the other from <bpt id="p4">**</bpt>az800l08-vnet2<ept id="p4">**</ept> to <bpt id="p5">**</bpt>az800l08-vnet0<ept id="p5">**</ept>. This completes setting up the hub and spoke topology (with the <bpt id="p1">**</bpt>az800l08-vnet0<ept id="p1">**</ept> virtual network serving the role of the hub, while <bpt id="p2">**</bpt>az800l08-vnet1<ept id="p2">**</ept> and <bpt id="p3">**</bpt>az800l08-vnet2<ept id="p3">**</ept> are its spokes).

#### <a name="task-3-test-transitivity-of-virtual-network-peering"></a>任务 3：测试虚拟网络对等互连的传递性

此任务将通过使用网络观察程序跨虚拟网络对等互连测试连接。

>**注意**：在开始此任务之前，请确保在此练习的第一个任务中调用的脚本已成功完成。

1. 在 Azure 门户中，浏览到“网络观察程序”页面。
1. 在“网络观察程序 - 连接故障排除”页面上，使用以下设置启动检查（将其他设置保留为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 源类型 | **虚拟机** |
    | 虚拟机 | **az800l08-vm0** |
    | 目标 | **手动指定** |
    | URI、FQDN 或 IPv4 | **10.81.0.4** |
    | 协议 | **TCP** |
    | Destination Port | **3389** |

    > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: <bpt id="p2">**</bpt>10.81.0.4<ept id="p2">**</ept> represents the private IP address of <bpt id="p3">**</bpt>az800l08-vm1<ept id="p3">**</ept>. The test uses the <bpt id="p1">**</bpt>TCP<ept id="p1">**</ept> port <bpt id="p2">**</bpt>3389<ept id="p2">**</ept> because Remote Desktop is by default enabled on Azure virtual machines and accessible within and between virtual networks.

1. Wait until results of the connectivity check are returned. Verify that the status is <bpt id="p1">**</bpt>Reachable<ept id="p1">**</ept>. Review the network path and note that the connection was direct, with no intermediate hops in between the VMs.

    > **注意**：这是预期行为，因为中心虚拟网络直接与第一个分支虚拟网络进行对等互连。

1. 在“网络观察程序 - 连接故障排除”页面上，使用以下设置启动另一项检查（将其他设置保留为默认值）：

    | 设置 | “值” |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 源类型 | **虚拟机** |
    | 虚拟机 | **az800l08-vm0** |
    | 目标 | **手动指定** |
    | URI、FQDN 或 IPv4 | **10.82.0.4** |
    | 协议 | **TCP** |
    | Destination Port | **3389** |

    > **注意**：10.82.0.4 表示 az800l08-vm2 的专用 IP 地址 。 

1. Select <bpt id="p1">**</bpt>Check<ept id="p1">**</ept> and wait until results of the connectivity check are returned. Verify that the status is <bpt id="p1">**</bpt>Reachable<ept id="p1">**</ept>. Review the network path and note that the connection was direct, with no intermediate hops in between the VMs.

    > **注意**：这是预期行为，因为中心虚拟网络直接与第二个分支虚拟网络进行对等互连。

1. 在“网络观察程序 - 连接故障排除”页面上，使用以下设置启动另一项检查（将其他设置保留为默认值），再启动另一项检查：

    > **注意**：可能需要刷新虚拟机 az800l08-vm1 的浏览器页，才能在“虚拟机”下拉列表中显示 。

    | 设置 | Value |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 源类型 | **虚拟机** |
    | 虚拟机 | **az800l08-vm1** |
    | 目标 | **手动指定** |
    | URI、FQDN 或 IPv4 | **10.82.0.4** |
    | 协议 | **TCP** |
    | Destination Port | **3389** |

1. Wait until results of the connectivity check are returned. Note that the status is <bpt id="p1">**</bpt>Unreachable<ept id="p1">**</ept>.

    > **注意**：这是预期行为，因为两个分支虚拟网络彼此之间并不是对等互连的，并且虚拟网络的对等互连性是不可传递的。

#### <a name="task-4-configure-routing-in-the-hub-and-spoke-topology"></a>任务 4：在中心辐射型拓扑中配置路由

此任务将通过在 az800l08-vm1 虚拟机的网络接口上启用 IP 转发，在其操作系统内启用路由，并在分支虚拟网络上配置用户定义的路由，来配置和测试两个分支虚拟网络之间的路由。

1. 在 Azure 门户中，浏览到 az800l08-vm0 虚拟机页面。
1. 在 az800l08-vm0 虚拟机页上，浏览到显示其网络接口 az800l08-nic0 的设置的页面 。
1. 在 az800l08-nic0 网络接口页上，显示其“IP 配置”页，并启用 IP 转发  。

   > **注意**：az800l08-vm0 需要此设置才能充当路由器，以路由两个分支虚拟网络之间的流量。

   > **注意**：现在你需要配置 az800l08-vm0 虚拟机的操作系统，以支持路由。

1. 在 Azure 门户中，浏览到 az800l08-vm0 虚拟机页面，并使用“运行命令”操作列表中的 RunPowerShellScript 运行以下命令，以安装远程访问 Windows Server 角色  。

   ```powershell
   Install-WindowsFeature RemoteAccess -IncludeManagementTools
   ```

   > **注意**：请等到显示命令已成功完成的确认信息。

1. 通过运行以下命令，再次使用 RunPowerShellScript 安装路由角色服务：

   ```powershell
   Install-WindowsFeature -Name Routing -IncludeManagementTools -IncludeAllSubFeature
   Install-WindowsFeature -Name "RSAT-RemoteAccess-Powershell"
   Install-RemoteAccess -VpnType RoutingOnly
   Get-NetAdapter | Set-NetIPInterface -Forwarding Enabled
   ```

   > **注意**：请等到显示命令已成功完成的确认信息。

   > **注意**：现在需要在分支虚拟网络上创建和配置用户定义的路由。

1. 在 Azure 门户中，浏览到“路由表”页面，并创建具有以下设置的路由表（其他设置保留其默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 位置 | 创建虚拟网络的 Azure 区域的名称 |
    | 名称 | **az800l08-rt12** |
    | 传播网关路由 | **是** |

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait for the route table to be created. This should take about 1 minute.

1. 浏览到新创建的路由表 az800l08-rt12 的页面，并添加具有以下设置的路由：

    | 设置 | Value |
    | --- | --- |
    | 路由名称 | **az800l08-route-vnet1-to-vnet2** |
    | 地址前缀 | **10.82.0.0/20** |
    | 下一跃点类型 | **虚拟设备** |
    | 下一跃点地址 | **10.80.0.4** |

    > **注意**：10.80.0.4 表示 az800l08-vm0 的专用 IP 地址 。 

1. 在 az800l08-rt12 路由表页面上，将路由表 az800l08-rt12 与以下子网进行关联 ：

    | 设置 | Value |
    | --- | --- |
    | 虚拟网络 | **az800l08-vnet1** |
    | 子网 | **subnet0** |

1. 转到“路由表”页面并创建具有以下设置的另一个路由表（将其他设置保留为其默认值）：

    | 设置 | “值” |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 区域 | 创建虚拟网络的 Azure 区域的名称 |
    | 名称 | **az800l08-rt21** |
    | 传播网关路由 | **是** |

   > <bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait for the route table to be created. This should take about 3 minutes.

1. 浏览到 az800l08-rt21 路由表页并添加具有以下设置的路由：

    | 设置 | Value |
    | --- | --- |
    | 路由名称 | **az800l08-route-vnet2-to-vnet1** |
    | 地址前缀 | **10.81.0.0/20** |
    | 下一跃点类型 | **虚拟设备** |
    | 下一跃点地址 | **10.80.0.4** |

1. 在 az800l08-rt21 路由表页面上，将路由表 az800l08-rt21 与以下子网进行关联 ：

    | 设置 | Value |
    | --- | --- |
    | 虚拟网络 | **az800l08-vnet2** |
    | 子网 | **subnet0** |

1. 在 Azure 门户中，浏览到“网络观察程序 - 连接故障排除”页面，使用以下设置启动检查（将其他设置保留为默认值）：

    | 设置 | Value |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 源类型 | **虚拟机** |
    | 虚拟机 | **az800l08-vm1** |
    | 目标 | **手动指定** |
    | URI、FQDN 或 IPv4 | **10.82.0.4** |
    | 协议 | **TCP** |
    | Destination Port | **3389** |

1. Wait until the results of the connectivity check are returned. Verify that the status is <bpt id="p1">**</bpt>Reachable<ept id="p1">**</ept>. Review the network path and note that the traffic was routed via <bpt id="p1">**</bpt>10.80.0.4<ept id="p1">**</ept>, assigned to the <bpt id="p2">**</bpt>az800l08-nic0<ept id="p2">**</ept> network adapter. 

    > **注意**：这是预期行为，因为分支虚拟网络之间的流量现在通过中心虚拟网络中的虚拟机（充当路由器）进行路由。


### <a name="exercise-2-implement-dns-name-resolution-in-azure"></a>练习 2：在 Azure 中实现 DNS 名称解析

### <a name="scenario"></a>场景

成功测试自定义路由后，现在可以使用 Azure 专用 DNS 区域为虚拟网络之间的 Azure 虚拟机实现 DNS 名称解析，并对使用 Azure DNS 区域进行外部名称解析作出评估。

此练习的主要任务如下：

1. 配置 Azure 专用 DNS 名称解析
1. 验证 Azure 专用 DNS 名称解析
1. 配置 Azure 公用 DNS 名称解析
1. 验证 Azure 公用 DNS 名称解析

#### <a name="task-1-configure-azure-private-dns-name-resolution"></a>任务 1：配置 Azure 专用 DNS 名称解析

此任务将使用 Azure 专用 DNS 区域配置虚拟网络之间的 DNS 名称解析。

1. 在 ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“专用 DNS 区域”页面 。 
1. 使用以下设置创建专用 DNS 区域：

    | 设置 | “值” |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组 AZ800-L0802-RG 的名称 |
    | 名称 | **contoso.org** |
    | 资源组位置 | 此实验室上一练习中部署资源的同一 Azure 区域 |

    >虚拟机：AZ-800T00A-SEA-DC1 和 AZ-800T00A-ADM1 必须正在运行 。

1. 浏览到 Contoso.org DNS 专用区域页面。
1. 在 contoso.org 专用 DNS 区域页面上，将虚拟网络链接添加到在上一练习中使用以下设置（将其他设置保留为默认值）创建的第一个虚拟网络：

    | 设置 | 值 |
    | --- | --- |
    | 链接名称 | **az800l08-vnet0-link** |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az800l08-vnet0 (AZ800-L0801-RG)** |
    | 启用自动注册 | 已选定 |

    >其他 VM 可以运行，但此实验室不需要这些 VM。

1. 重复前面的步骤，为虚拟网络 az800l08-vnet1 和 az800l08-vnet2 分别创建名为 az800l08-vnet1-link 和 az800l08-vnet2-link 的虚拟网络链接（并启用自动注册）   。
1. 在 contoso.org 专用 DNS 区域页面上，浏览到 contoso.org 专用 DNS 区域页的“概述”窗格，查看 DNS 记录集列表，并验证 az800l08-vm0、az800l08-vm1 和 az800l08-vm2 的 A 记录是否在列表中显示为“自动注册”       。

    >**注意：** 如果未列出记录集，你可能需要等待几分钟并刷新页面。

#### <a name="task-2-validate-azure-private-dns-name-resolution"></a>任务 2：验证 Azure 专用 DNS 名称解析

此任务将验证 Azure 专用 DNS 名称解析。

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“网络观察程序 - 连接故障排除”页面 。
1. 使用以下设置启动检查（将其他设置保留为默认值）：

    | 设置 | Value |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 源类型 | **虚拟机** |
    | 虚拟机 | **az800l08-vm1** |
    | 目标 | **手动指定** |
    | URI、FQDN 或 IPv4 | **az800l08-vm2.contoso.org** |
    | 首选 IP 版本 | **IPv4** |
    | 协议 | **TCP** |
    | Destination Port | **3389** |

1. Wait until the results of the connectivity check are returned. Verify that the status is <bpt id="p1">**</bpt>Reachable<ept id="p1">**</ept>. 

    > **注意**：这是预期行为，因为目标完全限定的域名 (FQDN) 可通过 Azure 专用 DNS 区域解析。 

#### <a name="task-3-configure-azure-public-dns-name-resolution"></a>任务 3：配置 Azure 公用 DNS 名称解析

此任务将使用 Azure 公用 DNS 区域配置外部 DNS 名称解析。

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开一个新选项卡，浏览到 https://www.godaddy.com/domains/domain-name-search 。
1. 使用域名搜索标识当前未使用的域名。
1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 选项卡，并浏览到“DNS 区域”页面 。
1. 使用以下设置（将其他设置保留为默认值）创建 DNS 区域：

    | 设置 | “值” |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0802-RG** |
    | 名称 | 之前在此任务中标识的 DNS 域名 |

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Wait for the DNS zone to be created. This should take about 1 minute.

1. 浏览到新创建的 DNS 区域的页面。
1. 添加具有以下设置的记录集（保留其他设置为默认值）：

    | 设置 | Value |
    | --- | --- |
    | 名称 | **www** |
    | 类型 | **A** |
    | 别名记录集 | **是** |
    | TTL | **1** |
    | TTL 单位 | **小时数** |
    | IP 地址 | 20.30.40.50 |

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The IP address and the corresponding name are entirely arbitrary. They are meant to provide a very simple example illustrating implementing public DNS records, rather than emulate a real world scenario, which would require purchasing a namespace from a DNS registrar. 

1. 在“DNS 区域”页上，标识“名称服务器 1”的全名。

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Record the full name of <bpt id="p2">**</bpt>Name server 1<ept id="p2">**</ept>. You will need it in the next task.

#### <a name="task-4-validate-azure-public-dns-name-resolution"></a>任务 4：验证 Azure 公用 DNS 名称解析

此任务将使用 Azure 公用 DNS 区域验证外部 DNS 名称解析。

1. 在 SEA-ADM1 上，在“开始”菜单中选择 Windows PowerShell  。
1. 在 Windows PowerShell 控制台中，运行以下命令以测试新创建的 DNS 区域中 www DNS 记录集的外部名称解析（将占位符 `<Name server 1>` 替换为之前记下的名称服务器 1 的名称，并将 `<domain name>` 占位符替换为之前在此任务中创建的 DNS 域的名称）  ：

   ```powershell
   nslookup www.<domain name> <Name server 1>
   ```

1. 验证该命令的输出是否包含 20.30.40.50 的公共 IP 地址。

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The name resolution works as expected because the <bpt id="p2">**</bpt>nslookup<ept id="p2">**</ept> command allows you to specify the IP address of the DNS server to query for a record (which, in this case, is <ph id="ph1">`&lt;Name server 1&gt;`</ph>. For the name resolution to work when querying any publicly accessible DNS server, you would need to register the domain name with a DNS registrar and configure the name servers listed on the public DNS zone page in the Azure portal as authoritative for the namespace corresponding to that domain.

## <a name="exercise-3-deprovisioning-the-azure-environment"></a>练习 3：取消预配 Azure 环境

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口。
1. 从 Azure 门户的 Cloud Shell 窗格中打开一个 PowerShell 会话。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>任务 2：标识实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出在此实验室过程中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L08*'
   ```

1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L08*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**注意**：该命令异步执行（由 -AsJob 参数决定），因此，虽然你可以随后立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。

### <a name="results"></a>结果

完成本实验室后，你将能够：

- 预配实验室环境。
- 配置中心辐射型网络拓扑。
- 测试虚拟网络对等互连的传递性。
- 在中心辐射型拓扑中配置路由。
- 配置 Azure DNS 进行内部名称解析。
- 配置 Azure DNS 进行外部名称解析。