---
lab:
  title: 实验室：在 Windows Server 中实现和配置网络基础设施服务
  module: 'Module 7: Network Infrastructure services in Windows Server'
ms.openlocfilehash: d50ffa7e7dc2631ee6c955b27e8b3507479ee8c9
ms.sourcegitcommit: 33fdeedf81ac2a39e09176f7a4b7a72b983a072f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/19/2022
ms.locfileid: "140742068"
---
# <a name="lab-implementing-and-configuring-network-infrastructure-services-in-windows-server"></a>实验室：在 Windows Server 中实现和配置网络基础设施服务

## <a name="scenario"></a>场景

Contoso, Ltd. 是一家大型组织，对网络服务具有复杂要求。 为了帮助满足这些要求，你将部署和配置 DHCP，以确保服务可用性。 你还将设置 DNS，以便 Trey Research（Contoso 内的一个部门）可以在测试区域中拥有自己的 DNS 服务器。 最后，你将提供对 Windows Admin Center 的远程访问权限，并使用 Web 应用程序代理对其进行保护。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 部署和配置 DHCP。
- 部署和配置 DNS。

## <a name="estimated-time-60-minutes"></a>估计时间：60 分钟

## <a name="lab-setup"></a>实验室设置

虚拟机：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1 和 AZ-800T00A-ADM1 必须正在运行  。 其他 VM 可以运行，但本实验室不需要这些 VM。

> 注意：AZ-800T00A-SEA-DC1、AZ-800T00A-SEA-SVR1 和 AZ-800T00A-ADM1 虚拟机承载 SEA-DC1、SEA-SVR1 和 SEA-ADM1 的安装      

1. 选择“SEA-ADM1”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境。

## <a name="exercise-1-deploying-and-configuring-dhcp"></a>练习 1：部署和配置 DHCP

### <a name="scenario"></a>场景

Contoso 的 Trey Research 部门有一个独立的办公室，该办公室只有大约 50 个用户。 他们已在其所有计算机上手动配置 IP 地址，并且想要开始改为使用 DHCP。 你将在 SEA-SVR1 上安装 DHCP，并且为 Trey research 站点创建范围。 此外，你将使用新的 DHCP 服务器来配置 DHCP 故障转移，以便使用 SEA-DC1 实现高可用性。

此练习的主要任务如下：

1. 安装 DHCP 角色。
1. 对 DHCP 服务器授权。
1. 创建范围。
1. 配置 DHCP 故障转移。
1. 验证 DHCP 功能。

### <a name="task-1-install-the-dhcp-role"></a>任务 1：安装 DHCP 角色

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，运行以下命令，下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 运行以下命令以安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 注意：请等待安装完成。 这大约需要 2 分钟。

1. 在 SEA-ADM1 上，启动 Microsoft Edge 并连接到 Windows Admin Center 的本地实例 (`https://SEA-ADM1.contoso.com`)。

   >注意：如果链接不起作用，请在 SEA-ADM1 上浏览到 WindowsAdminCenter.msi 文件，打开其上下文菜单，然后选择“修复”   。 修复完成后，刷新 Microsoft Edge。 
   
1. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在 Windows Admin Center 中，向 sea-svr1.contoso.com 添加一个连接，并以 CONTOSO\\Administrator 身份使用密码 Pa55w.rd 连接  。
1. 在“工具”列表中，使用“角色和功能”在 SEA-SVR1 上安装 DHCP 角色  。
1. 在“工具”列表中，浏览到 DHCP 工具并安装 DHCP PowerShell 工具  。 

   > 注意：如果 sea-svr1.contoso.com 的“工具”窗格中未提供 DHCP 条目，请刷新 Microsoft Edge 页面，然后重试   。

### <a name="task-2-authorize-the-dhcp-server"></a>任务 2：对 DHCP 服务器授权

1. 在 SEA-ADM1 上，打开服务器管理器。
1. 在服务器管理器中，打开“通知”，打开“完成 DHCP 配置”，然后使用默认选项完成 DHCP 安装后配置向导  。

### <a name="task-3-create-a-scope"></a>任务 3：创建范围

1. 在 SEA-ADM1 上，在 Windows Admin Center 中，当连接到 sea-svr1.contoso.com 时，使用 DHCP 工具采用以下设置创建一个新范围  ：

   - 协议：IPv4
   - 名称：ContosoClients
   - 起始 IP 地址：10.100.150.50
   - 结束 IP 地址：10.100.150.254
   - DHCP 客户端子网掩码：255.255.255.0
   - 路由器（默认网关）：10.100.150.1
   - DHCP 客户端的租约持续时间：4 天

1. 在服务器管理器的“工具”菜单中，打开 DHCP 管理控制台 。
1. 在 DHCP 管理控制台中，添加所有授权的服务器（172.16.10.12 和 SEA-DC1.contoso.com）  。
1. 在 DHCP 管理控制台，浏览到 DHCP 服务器 172.16.10.12 节点，然后在 ContosoClients 范围中，添加范围选项“006 DNS 服务器”，其值为 172.16.10.10    。

### <a name="task-4-configure-dhcp-failover"></a>任务 4：配置 DHCP 故障转移

1. 在 SEA-ADM1 上，在 DHCP 管理控制台中，浏览到 DHCP 服务器 172.16.10.12 的 IPv4 节点，使用以下设置将 ContosoClients 范围的故障转移配置为 SEA-DC1.contoso.com     ：

   - 关系名称：SEA-SVR1 到 SEA-DC1
   - 最长客户端提前期：1 小时
   - 模式：热备用
   - 伙伴服务器的角色：备用
   - 为备用服务器保留的地址数：5 %
   - 状态切换间隔：已禁用
   - 启用消息身份验证：已启用
   - 共享机密：DHCP-故障转移

1. 验证 SEA-SVR1 只有一个范围。
1. 验证 SEA-DC1 现在有两个范围。
1. 在 SEA-ADM1 上，在 DHCP 管理控制台中，浏览到 DHCP 服务器 SEA-DC1.contoso.com 的 IPv4 节点，使用现有故障转移关系的设置，将 Contoso 范围的故障转移配置为 172.16.10.12     。
1. 验证两个范围现在都出现在 172.16.10.12 (SEA-SVR1) 的 IPv4 节点内  。

### <a name="task-5-verify-dhcp-functionality"></a>任务 5：验证 DHCP 功能

1. 在 SEA-ADM1 上，将 IP 配置从静态分配更改为动态分配。
1. 检查生成的 IP 配置，并验证 DHCP 租约是否是从 SEA-SVR1 (172.16.10.12) 获取的。
1. 在 SEA-ADM1 上，在 DHCP 管理控制台中，验证这两个 DHCP 服务器是否列出 Contoso 范围内的 SEA-ADM1 的租约   。
1. 在 SEA-ADM1 上，使用 DHCP 管理控制台停止 SEA-SVR1 (172.16.10.12) 上的 DHCP 服务   。
1. 通过在 SEA-ADM1 上禁用并重新启用以太网网络连接来强制续订租约。
1. 在 SEA-ADM1 上，验证是否从 SEA-DC1 (172.16.10.10) 获得相同的 DHCP 租约 。

## <a name="exercise-2-deploying-and-configuring-dns"></a>练习 2：部署和配置 DNS

### <a name="scenario"></a>场景

在 Contoso 内的 Trey Research 位置工作的人员需要拥有自己的 DNS 服务器，以在其测试环境中创建记录。 但是，他们的测试环境仍需要能够解析 Contoso 的 Internet DNS 名称和资源记录。 为了满足这些需求，你要配置到 Internet 服务提供商 (ISP) 的转发，并为 contoso.com 到 SEA-DC1 创建一个条件转发器 。 还有一个测试应用程序，该应用程序需要基于用户位置进行不同 IP 地址解析。 你使用 DNS 策略来配置 testapp.treyresearch.net，以便对总部的用户进行不同的解析。

此练习的主要任务如下：

1. 安装 DNS 角色。
1. 创建 DNS 区域。
1. 配置转发。
1. 配置 DNS 条件转发。
1. 配置 DNS 策略。
1. 验证 DNS 策略功能。

### <a name="task-1-install-the-dns-role"></a>任务 1：安装 DNS 角色

1. 在 SEA-ADM1 上，使用连接到 sea-svr1.contoso.com 的 Windows Admin Center 的“工具”列表中的“角色和功能”，在 SEA-SVR1 上安装 DNS 角色    。
1. 在“工具”列表中，浏览到 DNS 工具并安装 DNS PowerShell 工具  。 

   > 注意：如果 sea-svr1.contoso.com 的“工具”列表中未提供 DNS 条目，请刷新 Microsoft Edge 页面，然后重试    。

### <a name="task-2-create-a-dns-zone"></a>任务 2：创建 DNS 区域

1. 在 SEA-ADM1 上，在 Windows Admin Center 中，当连接到 sea-svr1.contoso.com 时，使用 DNS 工具采用以下设置创建一个新的 DNS 区域  ：

   - 区域类型：主要
   - 区域名称：TreyResearch.net
   - 区域文件：创建新文件
   - 区域文件名：TreyResearch.net.dns
   - 动态更新：不允许动态更新

1. 在 Windows Admin Center 中，当连接到 sea-svr1.contoso.com 时，使用 DNS 工具采用以下设置在 TreyResearch.net 区域创建一个新的 DNS 记录  ：

   - DNS 记录类型：主机 (A)
   - 记录名称：TestApp
   - IP 地址：172.30.99.234
   - 生存时间：600

1. 在 SEA-ADM1 上，在 Windows PowerShell 控制台中，运行以下命令来验证新创建的记录是否正确解析 ：

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
    ```

### <a name="task-3-configure-forwarding"></a>任务 3：配置转发

1. 在 SEA-ADM1 上，从服务器管理器的“工具”菜单，打开 DNS 管理器控制台  。
1. 在 DNS 管理器中，连接到 SEA-SVR1.contoso.com 。
1. 在 SEA-SVR1.contoso.com 的属性中，在“转发器”选项卡上，将 131.107.0.100 配置为转发器  。

### <a name="task-4-configure-conditional-forwarding"></a>任务 4：配置条件转发

1. 在 SEA-ADM1 上，在 DNS 管理器中，当连接到 SEA-SVR1.contoso.com 时，为 Contoso.com 创建一个新的条件转发器，用于将请求转发到 SEA-DC1.contoso.com (172.16.10.10)     。
1. 在 SEA-ADM1 上，在 Windows PowerShell 控制台中，运行以下命令来验证条件转发器是否正常运行 ：

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name sea-dc1.contoso.com
    ```

### <a name="task-5-configure-dns-policies"></a>任务 5：配置 DNS 策略

1. 在 SEA-ADM1 上，在 Windows Admin Center 中，当连接到 sea-svr1.contoso.com 时，使用“工具”列表中的 PowerShell 来建立 PowerShell 远程会话   。
1. 在 Windows PowerShell 提示符下，运行以下命令以创建总部子网：

    ```powershell
    Add-DnsServerClientSubnet -Name "HeadOfficeSubnet" -IPv4Subnet "172.16.10.0/24"
    ```

1. 运行以下命令，为总部创建区域范围：

    ```powershell
    Add-DnsServerZoneScope -ZoneName "TreyResearch.net" -Name "HeadOfficeScope"
    ```

1. 运行以下命令，为总部范围创建新的资源记录：

    ```powershell
    Add-DnsServerResourceRecord -ZoneName "TreyResearch.net" -A -Name "testapp" -IPv4Address "172.30.99.100" -ZoneScope "HeadOfficeScope"
    ```

1. 运行以下命令，创建链接总部子网和区域范围的新策略：

    ```powershell
    Add-DnsServerQueryResolutionPolicy -Name "HeadOfficePolicy" -Action ALLOW -ClientSubnet "eq,HeadOfficeSubnet" -ZoneScope "HeadOfficeScope,1" -ZoneName "TreyResearch.net"
    ```

### <a name="task-6-verify-dns-policy-functionality"></a>任务 6：验证 DNS 策略功能

1. 在 SEA-ADM1 上，在 Windows PowerShell 控制台中，运行 `ipconfig` 验证 SEA-ADM1 是否在 HeadOfficeSubnet (172.16.10.0) 上   。
1. 在 Windows PowerShell 控制台中，运行以下命令来测试 DNS 策略：

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
    ```

   > 注意：验证名称是否解析为在 HeadOfficePolicy 中配置的 IP 地址 172.30.99.100  。

1. 将分配给 SEA-ADM1 的 IP 地址从 172.16.10.11 改为一个不在 HeadOfficeSubnet 的 IP 地址范围内的 IP 地址 (172.16.11.11)   。
1. 在 Windows PowerShell 控制台中，运行以下命令来测试 DNS 策略：

    ```powershell
    Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
    ```
   > 注意：验证名称是否解析为 172.30.99.234 。 这是意料之中的，因为 SEA-ADM1 的 IP 地址已经不在 HeadOfficeSubnet 内 。 从 HeadOfficeSubnet (172.16.10.0/24) 向 `testapp.treyresearch.net` 发起的 DNS 查询解析为 172.30.99.100  。 从此子网外部向 `testapp.treyresearch.net` 发起的 DNS 查询解析为 172.30.99.234。

1. 将 SEA-ADM1 的 IP 地址更改回其原始值。
