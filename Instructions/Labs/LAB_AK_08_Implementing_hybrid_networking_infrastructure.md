---
lab:
  title: 实验室：实现 Windows Server IaaS VM 网络
  type: Answer Key
  module: 'Module 8: Implementing Windows Server IaaS VM networking'
ms.openlocfilehash: 2cdfd4c09ee1b0db6ba40dd32c0fc2e349dedcd7
ms.sourcegitcommit: 32ed048f66b9810d6ee6f680c4b9637b28ffa316
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/28/2022
ms.locfileid: "141184694"
---
# <a name="lab-implementing-hybrid-networking-infrastructure"></a>实验室：实现混合网络基础结构

### <a name="exercise-1-implement-virtual-network-routing-in-azure"></a>练习 1：在 Azure 中实现虚拟网络路由

#### <a name="task-1-provision-lab-infrastructure-resources"></a>任务 1：预配实验室基础结构资源

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\\Administrator 的身份使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，启动 Microsoft Edge，转到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录。 
1. 在 Azure 门户中，通过选择搜索文本框旁边的工具栏图标打开“Cloud Shell”窗格。
1. 如果系统提示选择 Bash 或 PowerShell，请选择 PowerShell  。

   >备注：如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择要在此实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中，选择“上传”，将文件 **C:\\Labfiles\\Lab08\\L08-rg_template.json** 和 **C:\\Labfiles\\Lab08\\L08-rg_template.parameters.json** 上传到 Cloud Shell 主目录。
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

    >**注意**：在继续下一步之前，请等待部署完成。 这大约需要 3 分钟。

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

    >**注意**：请不要等待部署完成，而是继续执行下一步操作。 安装网络观察程序扩展大约需要 5 分钟。

#### <a name="task-2-configure-the-hub-and-spoke-network-topology"></a>任务 2：配置中心辐射型网络拓扑

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开另一个选项卡，浏览到 [Azure 门户](https://portal.azure.com) 。
1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“虚拟网络” 。
1. 在虚拟网络列表中，选择“az800l08-vnet0”。
1. 在 az800l08-vnet0 虚拟网络页上的“设置”部分中，选择“对等互连”，然后选择“+ 添加”   。
1. 指定以下设置（将其他设置保留为默认值），然后选择“添加”：

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

1. 在 az800l08-vnet0 虚拟网络页上的“设置”部分中，选择“对等互连”，然后选择“+ 添加”   。
1. 指定以下设置（将其他设置保留为默认值），然后选择“添加”：

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

    >**注意**：此步骤将建立两个对等互连：一个从 az800l08-vnet0 到 az800l08-vnet2，另一个从 az800l08-vnet2 到 az800l08-vnet0   。 至此就完成了中心辐射型拓扑的设置（其中 az800l08 vnet0 虚拟网络充当中心，而 az800l08 vnet1 和 az800l08-vnet2 充当分支）  。

#### <a name="task-3-test-transitivity-of-virtual-network-peering"></a>任务 3：测试虚拟网络对等互连的传递性

>**注意**：在开始此任务之前，请确保在此练习的第一个任务中调用的脚本已成功完成。

1. 在 Azure 门户中，搜索并选择“网络观察程序”。
1. 在“网络观察程序”页上，浏览到“连接故障排除” 。
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

    > **注意**：10.81.0.4 表示 az800l08-vm1 的专用 IP 地址 。 此测试使用 TCP 端口 3389，因为在 Azure 虚拟机上默认启用了远程桌面，且虚拟网络内部以及虚拟网络之间可以相互访问 。

1. 选择“检查”并等到返回连接检查结果。 验证状态是否为可访问。 查看网络路径，请注意，连接是直接的，VM 之间没有中间跃点。

    > **注意**：这是预期行为，因为中心虚拟网络直接与第一个分支虚拟网络进行对等互连。

1. 在“网络观察程序 - 连接故障排除”页面上，使用以下设置启动检查（将其他设置保留为默认值）：

    | 设置 | 值 |
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

1. 选择“检查”并等到返回连接检查结果。 验证状态是否为可访问。 查看网络路径，请注意，连接是直接的，VM 之间没有中间跃点。

    > **注意**：这是预期行为，因为中心虚拟网络直接与第二个分支虚拟网络进行对等互连。

1. 在“网络观察程序 - 连接故障排除”页面上，指定以下设置（将其他设置保留为默认值）并启动另一项检查：

    > **注意**：可能需要刷新虚拟机 az800l08-vm1 的浏览器页，才能在“虚拟机”下拉列表中显示 。

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 源类型 | **虚拟机** |
    | 虚拟机 | **az800l08-vm1** |
    | 目标 | **手动指定** |
    | URI、FQDN 或 IPv4 | **10.82.0.4** |
    | 协议 | **TCP** |
    | Destination Port | **3389** |

1. 选择“检查”并等到返回连接检查结果。 请注意，状态为“不可访问”。

    > **注意**：这是预期行为，因为两个分支虚拟网络彼此之间并不是对等互连的，并且虚拟网络的对等互连性是不可传递的。

#### <a name="task-4-configure-routing-in-the-hub-and-spoke-topology"></a>任务 4：在中心辐射型拓扑中配置路由

1. 在 Azure 门户中，搜索并选择“虚拟机”。
1. 在“虚拟机”页上的虚拟机列表中，选择“az800l08-vm0” 。
1. 在 az800l08-vm0 虚拟机页上的“设置”部分，选择“网络”  。
1. 选择“网络接口”标签旁边的“az800l08-nic0”链接，然后在“az800l08-nic0”网络接口页上的“设置”部分中，选择“IP 配置”    。
1. 将“IP 转发”设置为“已启用”，然后选择“保存”以保存更改  。

   > **注意**：az800l08-vm0 需要此设置才能充当路由器，以路由两个分支虚拟网络之间的流量。

   > **注意**：现在你需要配置 az800l08-vm0 虚拟机的操作系统，以支持路由。

1. 在 Azure 门户中，浏览回 az800l08-vm0 Azure 虚拟机页。
1. 在“az800l08-vm0”页的“操作”部分，选择“运行命令”，然后在命令列表中选择“RunPowerShellScript”   。
1. 在“运行命令脚本”页上，输入以下命令，然后选择“运行”以安装远程访问 Windows Server 角色。 

   ```powershell
   Install-WindowsFeature RemoteAccess -IncludeManagementTools
   ```

   > **注意**：请等到显示命令已成功完成的确认信息。

1. 在“运行命令脚本”页上的“PowerShell 脚本”部分中，将前面输入的命令替换为以下命令，然后选择“运行”以安装路由角色服务  。

   ```powershell
   Install-WindowsFeature -Name Routing -IncludeManagementTools -IncludeAllSubFeature
   Install-WindowsFeature -Name "RSAT-RemoteAccess-Powershell"
   Install-RemoteAccess -VpnType RoutingOnly
   Get-NetAdapter | Set-NetIPInterface -Forwarding Enabled
   ```

   > **注意**：请等到显示命令已成功完成的确认信息。

   > **注意**：现在需要在分支虚拟网络上创建和配置用户定义的路由。

1. 在 Azure 门户工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“路由表”，然后在“路由表”页面上，选择“+ 创建”   。
1. 使用以下设置（将其他设置保留为默认值）创建路由表：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 位置 | 创建虚拟网络的 Azure 区域的名称 |
    | 名称 | **az800l08-rt12** |
    | 传播网关路由 | **是** |

1. 选择“查看 + 创建”，然后选择“创建”。

   > **注意**：请等到路由表完成创建。 这大约需要 1 分钟。

1. 选择“转到资源”。 
1. 在 az800l08-rt12 路由表页上的“设置”部分中，选择“路由”，然后选择“+ 添加”   。
1. 添加具有以下设置的新路由：

    | 设置 | 值 |
    | --- | --- |
    | 路由名称 | **az800l08-route-vnet1-to-vnet2** |
    | 地址前缀 | **10.82.0.0/20** |
    | 下一跃点类型 | **虚拟设备** |
    | 下一跃点地址 | **10.80.0.4** |

    > **注意**：10.80.0.4 表示 az800l08-vm0 的专用 IP 地址 。 

1. 选择“确定”。
1. 在 az800l08-rt12 路由表页上的“设置”部分中，选择“子网”，然后选择“+ 关联”   。
1. 将路由表 az800l08-rt12 与以下子网关联：

    | 设置 | 值 |
    | --- | --- |
    | 虚拟网络 | **az800l08-vnet1** |
    | 子网 | subnet0 |

1. 选择“确定”。
1. 浏览回“路由表”页并选择“+ 创建” 。
1. 使用以下设置（将其他设置保留为默认值）创建路由表：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 区域 | 创建虚拟网络的 Azure 区域的名称 |
    | 名称 | **az800l08-rt21** |
    | 传播网关路由 | **是** |

1. 选择“查看 + 创建”，然后选择“创建”。

   > **注意**：请等到路由表完成创建。 这大约需要 3 分钟。

1. 选择“转到资源”。 
1. 在 az800l08-rt21 路由表页上的“设置”部分中，选择“路由”，然后选择“+ 添加”   。
1. 添加具有以下设置的新路由：

    | 设置 | 值 |
    | --- | --- |
    | 路由名称 | **az800l08-route-vnet2-to-vnet1** |
    | 地址前缀 | **10.81.0.0/20** |
    | 下一跃点类型 | **虚拟设备** |
    | 下一跃点地址 | **10.80.0.4** |

1. 选择“确定”。
1. 在 az800l08-rt21 路由表页上的“设置”部分中，选择“子网”，然后选择“+ 关联”   。
1. 将路由表 az800l08-rt21 与以下子网关联：

    | 设置 | 值 |
    | --- | --- |
    | 虚拟网络 | **az800l08-vnet2** |
    | 子网 | subnet0 |

1. 选择“确定”。
1. 在 Azure 门户中，浏览回“网络观察程序 - 连接故障排除”页。
1. 在“网络观察程序 - 连接故障排除”页面上，使用以下设置启动检查（将其他设置保留为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0801-RG** |
    | 源类型 | **虚拟机** |
    | 虚拟机 | **az800l08-vm1** |
    | 目标 | **手动指定** |
    | URI、FQDN 或 IPv4 | **10.82.0.4** |
    | 协议 | **TCP** |
    | Destination Port | **3389** |

1. 选择“检查”并等到返回连接检查结果。 验证状态是否为可访问。 查看网络路径，请注意，流量是通过 10.80.0.4 路由的，并且分配给 az800l08-nic0 网络适配器 。 

    > **注意**：这是预期行为，因为分支虚拟网络之间的流量现在通过中心虚拟网络中的虚拟机（充当路由器）进行路由。


### <a name="exercise-2-implement-dns-name-resolution-in-azure"></a>练习 2：在 Azure 中实现 DNS 名称解析

#### <a name="task-1-configure-azure-private-dns-name-resolution"></a>任务 1：配置 Azure 专用 DNS 名称解析

1. 在显示 Azure 门户的 Microsoft Edge 窗口的“SEA-ADM1”上，在工具栏的“搜索资源、服务和文档”文本框中，搜索并选择“专用 DNS 区域”，然后在“专用 DNS 区域”页上，选择“+ 创建”。    
1. 使用以下设置创建专用 DNS 区域：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组 AZ800-L0802-RG 的名称 |
    | 名称 | **contoso.org** |
    | 资源组位置 | 此实验室上一练习中部署资源的同一 Azure 区域 |

1. 选择“查看并创建”，然后选择“创建” 。

    >**注意**：请等到专用 DNS 区域创建完成。 这大约需要 2 分钟。

1. 选择“转到资源”，打开 contoso.org DNS 专用区域页 。
1. 在 contoso.org 专用 DNS 区域页上的“设置”部分中，选择“虚拟网络链接”  。
1. 在 contoso.org 虚拟网络链接页上，选择“+ 添加”，指定以下设置（将其他设置保留为其默认值），然后选择“确定”以创建在上一个练习中创建的第一个虚拟网络的虚拟网络链接 **\|**  ：

    | 设置 | 值 |
    | --- | --- |
    | 链接名称 | **az800l08-vnet0-link** |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az800l08-vnet0 (AZ800-L0801-RG)** |
    | 启用自动注册 | 已选定 |

    >**注意：** 请等到虚拟网络链接创建完成。 此过程应会在 1 分钟内完成。

1. 重复前面的步骤，为虚拟网络 az800l08-vnet1 和 az800l08-vnet2 分别创建名为 az800l08-vnet1-link 和 az800l08-vnet2-link 的虚拟网络链接（并启用自动注册）   。
1. 在 contoso.org 专用 DNS 区域页的左侧垂直菜单中，选择“概述” 。
1. 在 contoso.org 专用 DNS 区域页面的“概述”部分中，查看 DNS 记录集列表，并验证 az800l08-vm0、az800l08-vm1 和 az800l08-vm2 的 A 记录是否在列表中显示为“自动注册”。      

    >**注意：** 如果未列出记录集，你可能需要等待几分钟并刷新页面。

#### <a name="task-2-validate-azure-private-dns-name-resolution"></a>任务 2：验证 Azure 专用 DNS 名称解析

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“网络观察程序 - 连接故障排除”页面 。
1. 在“网络观察程序 - 连接故障排除”页面上，使用以下设置启动检查（将其他设置保留为默认值）：

    | 设置 | 值 |
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

1. 选择“检查”并等到返回连接检查结果。 验证状态是否为可访问。 

    > **注意**：这是预期行为，因为目标完全限定的域名 (FQDN) 可通过 Azure 专用 DNS 区域解析。 

#### <a name="task-3-configure-azure-public-dns-name-resolution"></a>任务 3：配置 Azure 公用 DNS 名称解析

1. 在 SEA-ADM1 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开一个新选项卡，浏览到 https://www.godaddy.com/domains/domain-name-search 。
1. 使用域名搜索标识当前未使用的域名。
1. 在“SEA-ADM1”上，切换到显示 Azure 门户的 Microsoft Edge 选项卡，在工具栏的“搜索资源、服务和文档”文本框中，搜索并选择“DNS 区域”，然后在“DNS 区域”页上，选择“+ 创建”。    
1. 在“创建 DNS 区域”页上，指定以下设置（其他设置保留默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | **AZ800-L0802-RG** |
    | 名称 | 之前在此任务中标识的 DNS 域名 |

1. 选择“查看 + 创建”，然后选择“创建”。

    >**注意**：请等到 DNS 区域创建完成。 这大约需要 1 分钟。

1. 选择“转到资源”以打开新创建的 DNS 区域的页面。
1. 在“DNS 区域”页上，选择“+ 记录集”。
1. 在“添加记录集”窗格中，指定以下设置（保留其他设置为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 名称 | **www** |
    | 类型 | **A** |
    | 别名记录集 | **是** |
    | TTL | **1** |
    | TTL 单位 | **小时数** |
    | IP 地址 | 20.30.40.50 |

    >**注意**：IP 地址和相应的名称完全是任意的。 它们旨在提供一个非常简单的示例来说明如何实现公共 DNS 记录，而不是模拟需要从 DNS 注册商处购买命名空间的真实场景。 

1. 选择“确定”
1. 在“DNS 区域”页上，标识“名称服务器 1”的全名。

    >**注意**：请记下“名称服务器 1”的全名。 稍后在下一个任务中将用到它。

#### <a name="task-4-validate-azure-public-dns-name-resolution"></a>任务 4：验证 Azure 公用 DNS 名称解析

1. 在 SEA-ADM1 上，在“开始”菜单中选择“Windows PowerShell”  。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter，测试新创建的 DNS 区域中 www DNS 记录集的外部名称解析（将占位符 `<Name server 1>` 替换为之前记下的名称服务器 1 的名称，并将 `<domain name>` 占位符替换为之前在此任务中创建的 DNS 域的名称）  ：

   ```powershell
   nslookup www.<domain name> <Name server 1>
   ```

1. 验证该命令的输出是否包含 20.30.40.50 的公共 IP 地址。

    >**注意**：名称解析按预期方式运行，因为 nslookup 命令允许你指定要查询的 DNS 服务器的 IP 地址（在此示例中为 `<Name server 1>`）。 要在查询任何可公开访问的 DNS 服务器时使用名称解析，需要向 DNS 注册机构注册域名，并将 Azure 门户中的“公共 DNS 区域”页上列出的名称服务器配置为与该域对应的命名空间的权威服务器。

## <a name="exercise-3-deprovisioning-the-azure-environment"></a>练习 3：取消预配 Azure 环境

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口。
1. 在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>任务 2：标识实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 页面中，运行以下命令，列出在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L08*'
   ```

1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ800-L08*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**注意**：该命令异步执行（由 -AsJob 参数决定），因此，虽然你可以随后立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。
