---
lab:
  title: 实验室：管理 Windows Server
  module: 'Module 3: Windows Server administration'
---

# 实验室：管理 Windows Server

## 场景

Contoso, Ltd. 想要在环境中实现多个新服务器，他们决定使用 Server Core。 他们还想要实现 Windows Admin Center，以远程管理这些服务器和组织中的其他服务器。

## 目标

- 实现和配置 Windows Admin Center

## 估计时间：45 分钟

## 实验室设置

虚拟机：AZ-800T00A-SEA-DC1 和 AZ-800T00A-ADM1 必须正在运行 。 其他 VM 可以运行，但此实验室不需要这些 VM。

> 注意：AZ-800T00A-SEA-DC1 和 AZ-800T00A-SEA-ADM1 虚拟机托管 SEA-DC1 和 SEA-ADM1 的安装    。

1. 选择“SEA-ADM1”。
1. 使用讲师提供的凭据进行登录。

对于本实验室，你将使用可用的 VM 环境和 Microsoft Entra 租户。

## 练习 1：实现和使用远程服务器管理

### 场景 

部署 Server Core 服务器后，需要实现 Windows Admin Center 以进行远程管理。

此练习的主要任务如下：

1. 安装 Windows Admin Center。
1. 添加用于远程管理的服务器。
1. 配置 Windows Admin Center 扩展。
1. 验证远程管理。
1. 使用远程 PowerShell 管理服务器。

#### 任务 1：安装 Windows Admin Center

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 。
1. 从 Windows PowerShell 控制台中，运行以下命令，下载最新版本的 Windows Admin Center：
    
   ```powershell
   $parameters = @{
         Source = "https://aka.ms/WACdownload"
         Destination = ".\WindowsAdminCenter.exe"
    }
    Start-BitsTransfer @parameters
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
    Start-Process -FilePath '.\WindowsAdminCenter.exe' -ArgumentList '/VERYSILENT' -Wait
   ```
   > 注意：请等待安装完成。 这大约需要 2 分钟。

   > **注意**：安装完成后，可能会遇到错误消息“ERR_Connection_Refused”。 如果发生这种情况，请重启 SEA-ADM1 以更正此问题。

#### 任务 2：添加用于远程管理的服务器

1. 在 SEA-ADM1 上，启动 Microsoft Edge，然后转到 `https://SEA-ADM1.contoso.com`。 
1. 出现提示时，请使用讲师提供的凭据登录。
1. 查看“所有连接”页，注意它包含 sea-adm1.contoso.com 条目 。 
1. 在“所有连接”窗格中，添加与 `sea-dc1.contoso.com` 的连接。
1. 出现提示时，请使用讲师提供的凭据登录。

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派。

#### 任务 3：配置 Windows Admin Center 扩展

1. 在 SEA-ADM1 的右上角，选择“设置”图标（齿轮） 。
1. 查看可用的扩展。
1. 安装“DNS”**** 扩展。 该扩展将安装，Windows Admin Center 将刷新。

1. 确认已安装的扩展列表中包含 DNS 扩展。
1. 在顶部菜单的“设置”旁边，选择下拉箭头，然后选择“服务器管理器” 。
1. 在 Windows Admin Center 中，连接到 `sea-dc1.contoso.com`，然后根据需要使用讲师提供的凭据进行登录。
1. 连接到 `sea-dc1.contoso.com` 上的 DNS 服务器并安装 DNS PowerShell 工具。
1. 选择“Contoso.com”区域并查看其 DNS 记录的列表。

#### 任务 4：验证远程管理

1. 在 SEA-ADM1 上，在 Windows Admin Center 中，连接到 `sea-dc1.contoso.com` 时，查看“概述”窗格。 请注意，Windows Admin Center 的“详细信息”窗格显示基本的服务器信息和性能监视。
1. 在 Windows Admin Center 中，使用“角色和功能”工具在 `sea-dc1.contoso.com` 上安装 Telnet 客户端。 
1. 在 Windows Admin Center 中，使用“设置”界面在 `sea-dc1.contoso.com` 上启用远程桌面。
1. 在 Windows Admin Center 中，通过远程桌面连接到 `sea-dc1.contoso.com`。
1. 断开与远程桌面会话的连接。 
1. 关闭 Microsoft Edge 窗口。

#### 任务 5：使用远程 PowerShell 管理服务器

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台 。
1. 在 Windows PowerShell 控制台中，运行以下命令以启动与 SEA-DC1 的 PowerShell 远程处理会话 ：

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1
   ```
1. 从 [SEA-DC1] 提示符中，运行以下命令以显示应用程序标识服务 (AppIDSvc) 的状态：

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > 注意：确认服务当前已停止。

1. 从 [SEA-DC1] 提示符中，运行以下命令以启动应用程序标识服务：

   ```powershell
   Start-Service -Name AppIDSvc
   ```
1. 从 [SEA-DC1] 提示符中，运行以下命令以显示应用程序标识服务 (AppIDSvc) 的状态：

   ```powershell
   Get-Service -Name AppIDSvc
   ```

   > 注意：确认服务当前正在运行。

### 结果

完成本练习后，你已安装 Windows Admin Center 并将其连接到实验室环境中的服务器。 你执行了许多远程管理任务，包括安装功能以及启用和测试远程桌面连接。 最后，你使用了 PowerShell 远程处理来检查服务的状态，然后启动它。
