---
layout: post
title: "Python 虚拟环境 (venv) 完全指南"
date: 2026-02-07 20:02:05 +0800
categories: development python
permalink: /python-venv-guide/
---

在进行 AI 或 Python 开发时，使用虚拟环境（Virtual Environment）是保证项目稳定性的**核心最佳实践**。

---

## 1. 什么是虚拟环境？
虚拟环境是一个**自包含**的目录，其中包含了一个特定版本的 Python 解释器以及一系列安装的包。
*   **物理形式**：通常表现为项目下的一个文件夹（如 `venv/`）。
*   **作用**：它像一个“沙盒”，你在里面安装的所有库都不会影响系统其他部分。

## 2. 为什么需要它？
*   **防止冲突**：项目 A 需要 `torch 1.0`，项目 B 需要 `torch 2.5`。虚拟环境让你能同时拥有这两个版本。
*   **环境一致性**：如果你把项目发给别人，他们可以根据你的配置创建一个一模一样的运行环境。
*   **无需 Root 权限**：普通用户也可以在自己的项目文件夹内随意安装库。

---

## 3. 工作原理（底层机制）
很多人好奇，为什么 `source venv/bin/activate` 之后，系统就能“神奇”地找到正确的 Python？

1.  **修改环境变量 ($PATH)**：当你激活环境时，它会将 `venv/bin` (或 Windows 的 `venv/Scripts`) 路径临时插入到你系统环境变量 `$PATH` 的最前面。
2.  **优先级覆盖**：由于它在最前面，当你输入 `python` 或 `pip` 时，系统会优先使用该目录下的程序，而不是系统的全局程序。
3.  **配置文件 (pyvenv.cfg)**：在 `venv` 目录下有一个 `pyvenv.cfg` 文件，它记录了基础 Python 的位置。当你运行虚拟环境里的 Python 时，它会参考这个文件来确定标准库和站点包（site-packages）的搜索路径。

---

## 4. 核心命令手册

### A. 创建环境
在项目根目录下运行（只需运行一次）：
```bash
# MacOS/Linux/Windows
python3 -m venv venv
```
*注：最后的 `venv` 是文件夹的名字，你可以自定义，但通常都叫 venv。*

### B. 激活环境 (Activate)
让终端“进入”这个沙盒。
*   **MacOS / Linux**:
    ```bash
    source venv/bin/activate
    ```
*   **Windows (PowerShell)**:
    ```powershell
    .\venv\Scripts\Activate.ps1
    ```
**激活成功后**，你的终端提示符前会出现 `(venv)` 字样。

### C. 退出环境 (Deactivate)
当你完成工作，想回到系统默认环境时：
```bash
deactivate
```

---

## 4. 管理依赖包

### 安装库
激活环境后，使用 `pip` 正常安装即可：
```bash
pip install torch jupyter matplotlib
```

### 冻结与恢复 (分享项目必备)
1.  **导出当前环境的所有库列表**：
    ```bash
    pip freeze > requirements.txt
    ```
2.  **在新机器上根据列表一键重装**：
    ```bash
    pip install -r requirements.txt
    ```

---

## 5. 在 Jupyter Notebook 中使用
如果你在虚拟环境中安装了库，但 Jupyter 找不到它们，通常是因为内核（Kernel）没选对。

1.  **方法一：在 VS Code 中选内核**
    *   点击 Notebook 右上角的 "Select Kernel"。
    *   在列表中寻找 `./venv/bin/python`。
2.  **方法二：手动注册内核 (如果上面的方法不行)**
    ```bash
    # 激活环境后运行
    pip install ipykernel
    python -m ipykernel install --user --name=python_venv --display-name "Python (venv)"
    ```

---

## 6. 注意事项
*   **不要把 `venv` 文件夹提交到 Git**：你应该在 `.gitignore` 文件中添加 `venv/`。别人应该通过 `requirements.txt` 自己创建环境，而不是拷贝你的环境文件夹。
*   **删除环境**：如果环境搞乱了，直接把 `venv` 文件夹删了，重新执行“创建”和“安装”步骤即可，非常安全。

---
*文档更新于：2026-02-07*
