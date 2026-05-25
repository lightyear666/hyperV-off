 这是一个彻底禁用 Windows Hyper-V
  及虚拟化安全功能的工具，源自一个中文模拟器项目（PDB路径：H:\trunk_模拟器cn_需求\simulator\bin\dishyperv.pdb）。

  ---
  主程序流程 (_wmain @ 0x4022C0)

  1. 创建日志文件 log\hyperV_disable.log
  2. 调用核心函数 sub_4018A0 执行所有禁用操作
  3. 退出清理

  ---
  核心功能 sub_4018A0 (closeHyperV) — 分三大类操作：

  A. DISM 禁用 Windows 功能

  调用 dism.exe /Online /NoRestart /Disable-Feature: 依次禁用以下 10 个功能：

  ┌─────────────────────────────────────────┐
  │                 功能名                  │
  ├─────────────────────────────────────────┤
  │ HypervisorPlatform                      │
  ├─────────────────────────────────────────┤
  │ Microsoft-Hyper-V                       │
  ├─────────────────────────────────────────┤
  │ Microsoft-Hyper-V-All                   │
  ├─────────────────────────────────────────┤
  │ Microsoft-Hyper-V-Tools-All             │
  ├─────────────────────────────────────────┤
  │ Microsoft-Hyper-V-Management-PowerShell │
  ├─────────────────────────────────────────┤
  │ Microsoft-Hyper-V-Hypervisor            │
  ├─────────────────────────────────────────┤
  │ Microsoft-Hyper-V-Services              │
  ├─────────────────────────────────────────┤
  │ Microsoft-Hyper-V-Management-Clients    │
  ├─────────────────────────────────────────┤
  │ VirtualMachinePlatform                  │
  ├─────────────────────────────────────────┤
  │ Containers-DisposableClientVM           │
  └─────────────────────────────────────────┘

  B. BCD 引导配置

  - bcdedit.exe /set hypervisorlaunchtype off — 在启动层面关闭 Hypervisor

  C. 注册表修改（关闭虚拟化安全相关功能）

  通过操作以下注册表项来禁用安全特性：
  - SystemGuard
  - HypervisorEnforcedCodeIntegrity
  - CredentialGuard
  - KernelShadowStacks
  - EnableVirtualizationBasedSecurity
  - WindowsHello
  - SecureBiometrics

  D. 服务配置 (sub_401680 — closeHyperService)

  - 查询 HvHost 服务，若运行中则设启动类型为 demand（手动）
  - 查询 vmms（Hyper-V 虚拟机管理服务），若运行中则设启动类型为 demand

  ---
  其他特征

  - 隐藏窗口: 创建一个 HWND_MESSAGE 类型的仅消息窗口（"STATIC" 类），用于接收系统消息
  - 并发库: 使用 ConCRT (Concurrency Runtime) 异步执行
  - 反调试: 导入 IsDebuggerPresent
  - C++ 标准库: 大量使用 MSVC STL（std::basic_filebuf, std::ctype 等）

  ---
  用途判断

  这是典型的 Android 模拟器辅助工具。许多 Android 模拟器（如网易MuMu、腾讯手游助手等）依赖 Intel HAXM 或 KVM
  进行硬件虚拟化加速，而这与 Hyper-V 的 VT-x 独占冲突。该工具的目的就是在运行模拟器之前，一键彻底清除 Hyper-V
  及其关联的虚拟化安全功能，确保模拟器能正常使用硬件加速。
