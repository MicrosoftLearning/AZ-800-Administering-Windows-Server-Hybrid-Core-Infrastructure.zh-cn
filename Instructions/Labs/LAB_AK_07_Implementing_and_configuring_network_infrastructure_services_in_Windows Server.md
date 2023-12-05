---
lab:
  title: 实验室：在 Windows Server 中实现和配置网络基础设施服务
  type: Answer Key
  module: 'Module 7: Network Infrastructure services in Windows Server'
---

# 实验室解答：在 Windows Server 中实现和配置网络基础设施服务

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-800%20Lab%20Simulation%20-%20Implementing%20and%20configuring%20network%20infrastructure%20services%20in%20Windows%20Server)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## 练习 1：部署和配置 DHCP

#### 任务 1：安装 DHCP 角色

1. 连接到 SEA-ADM1，然后根据需要，以 CONTOSO\Administrator 的身份使用密码 Pa55w.rd 登录  。
1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 备注：请等待安装完成。 这大约需要 2 分钟。

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后浏览到 `https://SEA-ADM1.contoso.com`。
 
   >注意：如果链接不起作用，请在 SEA-ADM1 上打开文件资源管理器，选择“下载”文件夹，在“下载”文件夹中选择 WindowsAdminCenter.msi 文件并手动安装。 安装完成后，刷新 Microsoft Edge。

   >注意：如果收到 NET::ERR_CERT_DATE_INVALID 错误，请在 Microsoft Edge 浏览器页上选择“高级”，在页面底部选择“继续访问 sea-adm1-contoso.com (不安全)” 。

2. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\Administrator
   - 密码：Pa55w.rd

3. 在“所有连接”窗格中，选择“+ 添加”。
4. 在“添加或创建资源”窗格中的“服务器”磁贴上，选择“添加” 。
5. 在“服务器名称”文本框中，输入“sea-svr1.contoso.com” 。 
6. 确保已选中“为此连接使用另一个帐户”选项，输入以下凭据，然后选择“使用凭据添加” ：

   - 用户名：CONTOSO\Administrator
   - 密码：Pa55w.rd

   > 备注：若要执行单一登录，你需要设置 Kerberos 约束委派。

7. 在 sea-svr1.contoso.com 页面上的“工具”列表中，选择“角色和功能”  。
8. 在“角色和功能”窗格中，选中“DHCP 服务器”复选框，然后选择“+ 安装” 。
9. 在“安装角色和功能”窗格中，选择“是”。

   > 备注：请等待指示已安装 DHCP 角色的通知。 如有必要，请选择“通知”图标以验证当前状态。

10. 刷新 Microsoft Edge 页面，返回到 sea-svr1.contoso.com 页面上，在“工具”列表中，选择“DHCP”，然后在详细信息窗格中，选择“安装”以安装 DHCP PowerShell 工具    。 

   > 备注：如果 sea-svr1.contoso.com 的“工具”列表中未提供“DHCP”条目，请刷新 Microsoft Edge 页面，然后重试    。
   根据网络性能，DHCP 服务器最长可能需要 5 分钟才能显示。

11. 等待指示已安装 DHCP PowerShell 工具的通知。 如有必要，请选择“通知”图标以验证当前状态。

#### 任务 2：对 DHCP 服务器授权

1. 在 SEA-ADM1 上，选择“开始”，然后选择“服务器管理器”  。
1. 在“服务器管理器”中，选择菜单中的“通知”，然后选择“完成 DHCP 配置”  。
1. 在“DHCP 安装后配置向导”窗口中的“说明”屏幕上，选择“下一步”  。
1. 在“授权”屏幕上，确保已选中“CONTOSO\Administrator”选项，然后选择“提交”  。
1. 完成这两项任务后，选择“关闭”。

#### 任务 3：创建范围

1. 在 SEA-ADM1 上，在 Microsoft Edge 窗口中切换到 Windows Admin Center，其中显示了 SEA-SVR1 上的“DHCP”设置  。

   > **注意**：DHCP 选项可能需要几分钟时间才会出现在菜单中。 如有必要，请刷新与 sea-svr1 的连接。 如果系统提示安装 DHCP Powershell 工具，请选择“安装”。

1. 在“DHCP”页面上，选择“+ 新建范围” 。
1. 在创建新范围的窗格中，指定以下设置，然后选择“创建”。

   - 协议：IPv4
   - 名称：ContosoClients
   - 起始 IP 地址：10.100.150.50
   - 结束 IP 地址：10.100.150.254
   - DHCP 客户端子网掩码：255.255.255.0
   - 路由器（默认网关）：10.100.150.1
   - DHCP 客户端的租约持续时间：4 天

1. 在 SEA-ADM1 上，切换到“服务器管理器”，在“服务器管理器”中，选择“工具”，然后选择“DHCP”    。
1. 在“DHCP”窗口中的“操作”窗格中，选择“更多操作”，然后选择“管理已授权的服务器”  。
1. 在“管理已授权的服务器”窗口中，选择“刷新”，确保 sea-svr1.contoso.com 已出现在已授权的 DHCP 服务器列表中，然后关闭“管理已授权的服务器”窗口   。
1. 在“DHCP”窗口中的“操作”窗格中，选择“更多操作”，然后选择“添加服务器”  。
1. 在“添加服务器”对话框中，选择“此已授权的 DHCP 服务器”，选择“sea-svr1.contoso.com”，然后选择“确定”   。
1. 在“DHCP”窗口中，依次展开“sea-svr1”、“IPv4”和“范围 [10.100.150.0] ContosoClients”，然后选择“范围选项”   。
1. 在“操作”窗格中，选择“更多操作”，然后选择“配置选项” 。
1. 在“范围选项”对话框中，选中“006 DNS 服务器”复选框 。
1. 在“服务器名称”文本框中，输入 sea-dc1.contoso.com，选择“解析”，确认名称已解析为 172.16.10.10，选择“添加”，然后选择“确定”     。

#### 任务 4：配置 DHCP 故障转移

1. 在 SEA-ADM1 上，在“DHCP”窗口中选择“IPv4”，在“操作”窗格中选择“更多操作”，然后选择“配置故障转移”    。
1. 在“配置故障转移”窗口中，确认已选中“全部选择”复选框，然后选择“下一步”  。
1. 在“指定要用于故障转移的伙伴服务器”屏幕上，选择“添加服务器” 。
1. 在“添加服务器”对话框中，选择“此已授权的 DHCP 服务器”，选择“sea-dc1.contoso.com”，然后选择“确定”   。
1. 返回到“指定要用于故障转移的伙伴服务器”屏幕上，确保“sea-dc1”出现在“伙伴服务器”下拉列表中，然后选择“下一步”   。
1. 在“创建新的故障转移关系”屏幕上，输入以下信息，然后选择“下一步” 。

   - 关系名称：SEA-SVR1 到 SEA-DC1
   - 最长客户端提前期：1 小时
   - 模式：热备用
   - 伙伴服务器的角色：备用
   - 为备用服务器保留的地址数：5 %
   - 状态切换间隔：已禁用
   - 启用消息身份验证：已启用
   - 共享机密：DHCP-故障转移

1. 选择“完成”。
1. 在“配置故障转移”对话框中，选择“关闭” 。
1. 在“DHCP”窗口中的“操作”窗格中，选择“更多操作”，然后选择“添加服务器”  。
1. 在“添加服务器”对话框中，选择“此已授权的 DHCP 服务器”，选择“sea-dc1.contoso.com”，然后选择“确定”   。
1. 在 SEA-ADM1 上，在“DHCP”窗口中展开“sea-dc1”节点，选择“IPv4”，然后验证是否列出了两个范围   。
1. 选择“范围 [172.16.0.0] Contoso”，在“操作”窗格中，选择“更多操作”，然后选择“配置故障转移”  。
1. 在“配置故障转移”窗口中，选择“下一步” 。
1. 在“指定要用于故障转移的伙伴服务器”屏幕上，在“伙伴服务器”框中，输入“172.16.10.12”，选中“重用使用此服务器配置的现有故障转移关系(如果存在)”复选框，然后选择“下一步”    。
1. 在“从已在此服务器上配置的故障转移关系中选择”屏幕上，选择“下一步”，然后选择“完成”  。
1. 在“配置故障转移”对话框中，选择“关闭” 。
1. 在“sea-svr1”下，选择“IPv4”，然后验证是否已列出了两个范围。 如有必要，请按 F5 键刷新。

#### 任务 5：验证 DHCP 功能

1. 在 SEA-ADM1 上，选择“开始”，然后选择“设置”  。
1. 在“设置”窗口中，选择“网络和 Internet”，然后选择“网络和共享中心”  。
1. 在“网络和共享中心”中，选择“以太网”，然后选择“属性”  。
1. 在“以太网属性”对话框中，选择“Internet 协议版本 4 (TCP/IPv4)”，然后选择“属性”  。
1. 在“Internet 协议版本 4 (TCP/IPv4) 属性”对话框中，选择“自动获取 IP 地址”，选择“自动获取 DNS 服务器地址”，然后选择“确定”   。
1. 选择“关闭”，然后在“以太网状态”窗口中，选择“详细信息”  。
1. 在“网络连接详细信息”对话框中，验证是否已启用 DHCP、是否已获取 IP 地址，以及“sea-svr1”DHCP 服务器是否发出了租约。
1. 选择“关闭”以回到“以太网状态”窗口 。
1. 在 SEA-ADM1 上，在“DHCP”窗口中，依次展开“172.16.10.12”节点、“IPv4”节点和“范围 [172.16.0.0] Contoso”节点，然后选择“地址租约”     。
1. 验证是否存在代表 SEA-ADM1.contoso.com 租约的条目。
1. 在 SEA-ADM1 上，在“DHCP”窗口中，依次展开“sea-dc1”节点、“IPv4”节点和“范围 [172.16.0.0] Contoso”节点，然后选择“地址租约”     。
1. 验证此处是否也存在代表 SEA-ADM1.contoso.com 租约的条目。
1. 选择“sea-svr1”，在“操作”窗格中，选择“更多操作”，选择“所有任务”，然后选择“停止”  。
1. 在 SEA-ADM1 上，切换回“以太网状态”窗口，然后选择“禁用”  。
1. 返回到“网络和共享中心”窗口中，选择“更改适配器设置”，选择“以太网”，然后选择“启用此网络设备”   。
1. 双击已启用的“以太网”连接以显示其状态窗口。
1. 在“以太网状态”窗口中，选择“详细信息” 。
1. 在“网络连接详细信息”对话框中，验证 DHCP 是否已启用、是否已获取 IP 地址，以及“SEA-DC1 (172.16.10.10)”DHCP 服务器是否发出了租约 。
1. 选择“关闭”以回到“以太网状态”窗口 。
1. 在“以太网状态”窗口中，选择“属性” 。
1. 在“以太网属性”对话框中，选择“Internet 协议版本 4 (TCP/IPv4)”，然后选择“属性”  。
1. 在“Internet 协议版本 4 (TCP/IPv4) 属性”对话框中，选择“使用以下 IP 地址”，并指定以下设置 ：

   - IP 地址：172.16.10.11
   - 子网掩码：255.255.0.0
   - 默认网关：172.16.10.1

1. 在“Internet 协议版本 4 (TCP/IPv4) 属性”对话框中，选择“使用以下 DNS 服务器地址”，将“首选 DNS 服务器”设置为“172.16.10.10”，然后选择“确定”    。

   > 备注：使“以太网状态”窗口保持打开状态 。 本实验室中稍后会用到它。 

## 练习 2：部署和配置 DNS

#### 任务 1：安装 DNS 角色

1. 在 SEA-ADM1 上，切换回 Microsoft Edge 窗口，其中显示了到 Windows Admin Center 中 sea-svr1.contoso.com 的连接 。 
1. 在“工具”列表中，选择“角色和功能” 。
1. 在“角色和功能”窗格中，选中“DNS 服务器”复选框，然后选择“+ 安装” 。
1. 在“安装角色和功能”窗格中，选择“是”。

   > 备注：请等待指示已安装 DNS 角色的通知。 如有必要，请选择“通知”图标以验证当前状态。

1. 刷新 Microsoft Edge 页面，返回到 sea-svr1.contoso.com 页面上，在“工具”列表中，选择“DNS”，然后在详细信息窗格中，选择“安装”以安装 DNS PowerShell 工具    。 

   > 备注：如果 sea-svr1.contoso.com 的“工具”列表中未提供“DNS”条目，请刷新 Microsoft Edge 页面，然后重试    。

1. 等待出现指示已安装 DNS PowerShell 工具的通知。 如有必要，请选择“通知”图标以验证当前状态。

#### 任务 2：创建 DNS 区域

1. 在 SEA-ADM1 上，在 Windows Admin Center 中的 DNS 窗格中，选择“操作”，然后在“操作”菜单上，选择“+ 创建新的 DNS 区域”   。
1. 在“创建新的 DNS 区域”对话框中，指定以下设置，然后选择“创建” ：

   - 区域类型：主要
   - 区域名称：TreyResearch.net
   - 区域文件：创建新文件
   - 区域文件名：TreyResearch.net.dns
   - 动态更新：不允许动态更新

1. 返回到 DNS 窗格中，选择“TreyResearch.net”，然后选择“+ 创建新的 DNS 记录” 。
1. 在“创建新 DNS 记录”窗格中，指定以下设置，然后选择“创建” ：

   - DNS 记录类型：主机 (A)
   - 记录名称：TestApp
   - IP 地址：172.30.99.234
   - 生存时间：600

1. 在 SEA-ADM1 上，选择“开始”，然后选择“Windows PowerShell”  。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 键以验证新 DNS 记录是否提供了名称解析：

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
   ```

#### 任务 3：配置转发

1. 在 SEA-ADM1 上，切换到“服务器管理器”。
1. 在“服务器管理器”中，选择“工具”，然后选择“DNS” 。
1. 在“DNS 管理器”中，选择“DNS”，显示其上下文相关菜单，然后在菜单中，选择“连接到 DNS 服务器”  。
1. 在“连接到 DNS 服务器”对话框中，选择“以下计算机”，输入“SEA-SVR1.contoso.com”，然后选择“确定”   。
1. 在“DNS 管理器”中，选择“SEA-SVR1.contoso.com”，显示其上下文相关菜单，然后选择“属性”  。
1. 在“SEA-SVR1.contoso.com 属性”对话框中，选择“转发器”选项卡，然后选择“编辑”  。
1. 在“编辑转发器”对话框中，在“转发服务器的 IP 地址”框中，输入“131.107.0.100”，然后选择“确定”   。
1. 在“SEA-SVR1.contoso.com 属性”对话框中，选择“确定” 。

#### 任务 4：配置条件转发

1. 在 SEA-ADM1 上，在“DNS 管理器”中，展开“SEA-SVR1.contoso.com”，然后选择“条件转发器”   。
1. 选择“条件转发器”，显示其上下文相关菜单，然后在菜单中，选择“新建条件转发器” 。
1. 在“新建条件转发器”对话框中，在“DNS 域”框中，输入“Contoso.com”  。
1. 在“主服务器的 IP 地址”框中，输入“172.16.10.10”，然后选择 Enter  。

   > 备注：忽略“新建条件转发器”对话框内验证列中“出现未知错误”的消息  。

1. 选择“确定”。
1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台 。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 以验证条件转发：

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name sea-dc1.contoso.com
   ```

#### 任务 5：配置 DNS 策略

1. 在 SEA-ADM1 上，切换回 Microsoft Edge 窗口，其中显示了到 Windows Admin Center 中 sea-svr1.contoso.com 的连接 。
1. 在“工具”列表中，选择“PowerShell”，当出现提示时，以 CONTOSO\Administrator 用户身份使用密码 Pa55w.rd 登录   。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 以创建总部子网：

   ```powershell
   Add-DnsServerClientSubnet -Name "HeadOfficeSubnet" -IPv4Subnet '172.16.10.0/24'
   ```
1. 输入以下命令，然后按 Enter 为总部创建区域范围：

   ```powershell
   Add-DnsServerZoneScope -ZoneName 'TreyResearch.net' -Name 'HeadOfficeScope'
   ```
1. 输入以下命令，然后按 Enter 为总部范围添加新资源记录：

   ```powershell
   Add-DnsServerResourceRecord -ZoneName 'TreyResearch.net' -A -Name 'testapp' -IPv4Address '172.30.99.100' -ZoneScope 'HeadOfficeScope'
   ```
1. 输入以下命令，然后按 Enter 创建链接总部子网和区域范围的新策略：

   ```powershell
   Add-DnsServerQueryResolutionPolicy -Name 'HeadOfficePolicy' -Action ALLOW -ClientSubnet 'eq,HeadOfficeSubnet' -ZoneScope 'HeadOfficeScope,1' -ZoneName 'TreyResearch.net'
   ```

#### 任务 6：验证 DNS 策略功能

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台 。
1. 在 Windows PowerShell 控制台中，输入 `ipconfig`，然后按 Enter 显示其当前的 IP 配置。

   > 备注：请注意，以太网适配器有一个 IP 地址，此地址是策略中配置的 HeadOfficeSubnet 的一部分 。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 测试 `testapp.treyresearch.net` DNS 记录的解析：

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
   ```

   > 备注：验证名称是否解析为在 HeadOfficePolicy 中配置的 IP 地址 172.30.99.100  。

1. 在 SEA-ADM1 上，切换回“以太网状态”窗口 。
1. 在“以太网状态”窗口中，选择“属性” 。
1. 在“以太网属性”对话框中，选择“Internet 协议版本 4 (TCP/IPv4)”，然后选择“属性”  。
1. 在“Internet 协议版本 4 (TCP/IPv4) 属性”对话框中，将当前分配的 IP 地址 (172.16.10.11) 更改为不在 HeadOfficeSubnet 的 IP 地址范围内的一个 IP 地址 172.16.11.11，然后选择“确定”    。
1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台 。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 测试 `testapp.treyresearch.net` DNS 记录的解析：

   ```powershell
   Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
   ```

   > 备注：验证名称是否解析为 172.30.99.234 。 这是意料之中的，因为 SEA-ADM1 的 IP 地址已经不在 HeadOfficeSubnet 内 。 从 HeadOfficeSubnet (172.16.10.0/24) 向 `testapp.treyresearch.net` 发起的 DNS 查询解析为 172.30.99.100  。 从此子网外部向 `testapp.treyresearch.net` 发起的 DNS 查询解析为 172.30.99.234。

   > 备注：现在将 SEA-ADM1 的 IP 地址更改回其原始值 。

1. 切换回“以太网状态”窗口，然后选择“属性” 。
1. 在“以太网属性”对话框中，选择“Internet 协议版本 4 (TCP/IPv4)”，然后选择“属性”  。
1. 在“Internet 协议版本 4 (TCP/IPv4) 属性”对话框中，将当前分配的 IP 地址 (172.16.11.11) 更改为其原始值 (172.16.10.11)，然后选择“确定”   。
1. 选择两次“关闭”。
1. 关闭所有打开的窗口。
