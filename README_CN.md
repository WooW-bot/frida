# Frida 中文指南

Frida 是面向开发者、逆向工程师和安全研究员的动态插桩工具箱。它允许你将 JavaScript 脚本注入到原生 App 中，实时监控、修改函数调用。

了解更多信息，请访问 [frida.re](https://frida.re/)。

---

## 目录
1. [快速安装](#1-快速安装推荐)
2. [从源码编译（小白入门）](#2-从源码编译)
3. [编译产物说明](#3-编译产物说明)
4. [安装指南（以-android-为例）](#4-安装指南以-android-为例)
5. [如何使用-frida](#5-如何使用-frida)

---

## 1. 快速安装（推荐）

这是最简单的方法，直接安装官方预编译好的工具：

```bash
pip install frida-tools # 安装命令行工具 (如 frida, frida-ps)
pip install frida       # 安装 Python 绑定
npm install frida       # 安装 Node.js 绑定
```

你也可以从 GitHub 的 [Releases](https://github.com/frida/frida/releases) 页面手动下载对应系统的二进制文件。

---

## 2. 从源码编译

如果你想通过修改 Frida 源码来定制功能，就需要手动编译。

### 2.1 环境准备
在开始之前，请确保你的电脑已安装以下工具：
- **GNU Make**: 编译控制工具。
- **Python 3.x**: 脚本支持。
- **Node.js**: Frida 的部分组件需要它。
- **GCC / Clang**: C++ 编译器（MacOS 用户请安装 Xcode 命令行工具）。

### 2.2 编译命令
在当前目录下运行：

```bash
make
```

如果你想为特定平台编译，可以使用：
- `make core-macos-apple_silicon` (MacOS ARM芯片)
- `make core-android-arm64` (安卓 64位)
- `make core-linux-x86_64` (Linux 64位)

编译过程可能需要较长时间，因为它会拉取相关的子项目。

---

## 3. 编译产物说明

编译完成后，你会在 `build/` 目录下找到不同的产物。以下是它们的主要作用：

*   **`frida-server`**: 这是最重要的文件。它是一个可执行程序，需要运行在**目标设备**（如你的手机）上。它作为桥梁，接收你电脑发出的指令。
*   **`frida-gadget`**: 一个共享库（.so 或 .dylib）。在没有 Root 的手机上，或者你只想针对特定的 App 进行插桩时，可以将它打包进 App 内部。
*   **`frida-agent`**: Frida 的核心库，在插桩时会被自动注入到目标进程中。
*   **`frida-helper`**: 一个辅助工具，用于处理一些权限较高或特定的注入操作。

---

## 4. 安装指南（以 Android 为例）

如果你编译或下载了 `frida-server`，按照以下步骤将其安装到安卓手机：

1.  **连接手机**：确保开启了“USB 调试”。
2.  **上传文件**：
    ```bash
    adb push build/frida-android-arm64/bin/frida-server /data/local/tmp/
    ```
3.  **修改权限**：
    ```bash
    adb shell "chmod +x /data/local/tmp/frida-server"
    ```
4.  **运行服务端**：
    ```bash
    adb shell "/data/local/tmp/frida-server &"
    ```

> [!TIP]
> 运行后，`frida-server` 会在手机后台静默运行，不会有输出界面。

---

## 5. 如何使用 Frida

一旦手机上的 `frida-server` 跑起来了，你就可以在你的电脑（主机）上使用 Frida 工具了。

### 5.1 查看进程列表
确定连接是否成功，列出手机上正在运行的进程：
```bash
frida-ps -U
```
（`-U` 代表通过 USB 连接）

### 5.2 追踪函数调用
比如，你想看看某个 App 调用了哪些加密相关的函数：
```bash
frida-trace -U -i "SecKey*" com.example.app
```

### 5.3 注入自定义脚本
编写一个 `script.js`，然后运行：
```bash
frida -U -f com.example.app -l script.js
```
`-f` 参数表示强制重启该 App 并附加（Attach）上去。

---

## 更多学习资源
- [官方文档 (英文)](https://frida.re/docs/home/)
- [Frida 的 JavaScript API 指南](https://frida.re/docs/javascript-api/)
