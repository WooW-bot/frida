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
- **Android NDK (r29)**: 如果你要为 Android 编译，必须安装并设置环境变量。

**设置 NDK 路径示例：**
```bash
export ANDROID_NDK_ROOT=/你的/ndk/路径/29.x.x
```
建议将其添加到 `~/.zshrc` 或 `~/.bash_profile` 中。

### 2.2 编译步骤
现代版本的 Frida 使用“先配置，后编译”的逻辑：

1.  **清理旧配置**（如果切换平台）：`rm -rf build`
2.  **设置 macOS 签名**（防止报错）：`export MACOS_CERTID=-`
3.  **针对目标平台配置**：
    - `core-macos-apple_silicon` -> `./configure --host=macos-arm64`
    - `core-android-arm64` -> `./configure --host=android-arm64`
4.  **开始编译**：
    ```bash
    make
    ```

---

## 3. 编译产物说明

编译完成后，产物通常位于 `build/subprojects/frida-core/` 目录下的对应子目录。

*   **`frida-server`**: 运行在目标设备（如手机）上的服务端。路径示例：`build/subprojects/frida-core/server/frida-server`。
*   **`frida-gadget`**: 共享库，用于非 Root 注入。
*   **`frida-agent`**: Frida 核心库，注入到进程中。

---

## 4. 安装指南（以 Android 为例）

1.  **推送到手机**：
    ```bash
    adb push build/subprojects/frida-core/server/frida-server /data/local/tmp/
    ```
2.  **设置权限**：
    ```bash
    adb shell "chmod +x /data/local/tmp/frida-server"
    ```
3.  **以 Root 权限运行**：
    ```bash
    adb shell "su -c '/data/local/tmp/frida-server &'"
    ```
    > [!IMPORTANT]
    > 必须使用 `su -c` 运行，否则会因为权限不足（如 SELinux）导致无法注入进程。

---

## 5. 常见问题排查

### 5.1 Address already in use
如果你看到 `Error binding to address: Address already in use`，说明已经有一个 frida-server 在运行。
**解决方法**：
```bash
adb shell "su -c 'pkill -9 frida-server'"
```

### 5.2 缺少 Python 模块 (setuptools/wheel)
如果在编译时提示找不到模块，请安装它们并添加强制标志：
```bash
python3 -m pip install setuptools wheel --break-system-packages
```

---

## 6. 如何使用 Frida

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
