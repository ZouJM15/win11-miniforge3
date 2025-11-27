# win11 配置 miniforge3
<a id="windows11-config-miniforge3"></a>
本文详细介绍了在Windows11系统上配置Miniforge3的完整流程。主要内容包括：1)从清华镜像站或GitHub下载安装Miniforge3；2)配置.condarc文件设置镜像源和环境目录；3)提供全面的Miniforge/Mamba指令集，涵盖环境创建、包管理、环境导出迁移等操作。

## 目录

- [windows11 配置 miniforge3](#windows11-config-miniforge3)
  - [一. miniforge 下载及安装](#miniforge-download-install)
    - [1. 下载](#miniforge-download)
    - [2. 安装](#miniforge-install)
  - [二. .condarc 文件配置](#condarc-file-config)
  - [三. 创建环境](#create-env)
  - [四. 初始化 Shell （One-Time Setup）](#init-shell)
    - [1. 初始化 cmd.exe](#init-shell-cmd)
    - [2. 初始化 PowerShell](#init-shell-PowerShell)
  - [五. miniforge指令总结](#miniforge-commands)
    - [🚀 常用指令](#common-commands)
    - [📦 包管理常用指令](#package-management-commands)
    - [🔧 环境配置相关指令](#env-config-commands)
    - [📂 环境导出与迁移](#env-export-migration)
    - [🧠 AI 项目常用 Conda 指令](#ai-project-conda-commands)

<a id="miniforge-download-install"></a>

## 一. miniforge 下载及安装

<a id="miniforge-download"></a>

### 1. 下载

1. 清华大学开源软件镜像站：[清华镜像 - Miniforge](https://mirrors.tuna.tsinghua.edu.cn/github-release/conda-forge/miniforge/LatestRelease/)
2. GitHub: [conda-forge/miniforge](https://github.com/conda-forge/miniforge)

- 下载 `Miniforge3-Windows-x86_64.exe` 后安装

<a id="miniforge-install"></a>

### 2. 安装

几个特殊选项：

- Install for： 选`Just Me` （其实选All Users也没啥区别）
- 选 `create shortcuts` , `add installation to my PATH environment variable` 和 `Register Miniforge3 as my default Python 3.12` 这前三个选项

---

<a id="condarc-file-config"></a>

## 二. .condarc 文件配置

位置：C盘-> user（用户）-> 用户名 -> .condarc
**若没有则在此位置新建**

```yaml
# 告诉 conda 你的包主要从哪里下载。
# 排序很重要：上面越靠前的优先级越高。
channels:
  - conda-forge

# 当从 conda-forge 下载包时，设置去清华镜像找
# 还有别的镜像 https://mirrors.ustc.edu.cn/anaconda/cloud （中科大）
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud

# channel_alias: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
# mirrored_channels:
#   conda-forge:
#     - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
#     - https://conda.anaconda.org/conda-forge
#     - https://prefix.dev/conda-forge

# 环境保存的位置，自己定
envs_dirs:
  - D:\Tools\miniforge3\envs

# 安装包时，显示它来自哪个源
show_channel_urls: true
# 打开终端时，不自动进入 base 环境（推荐）
auto_activate_base: false
# 严格优先级，来自高优先级 channel 的包优先使用，避免冲突
channel_priority: strict
```

保存后运行指令 `mamba clean --all -y` 再创建环境。

**注意：** *若用miniforge专用的mirrored_channels，则需要写全URL。
他不像custom_channels和channel_alias，并没有URL补全功能，如指定conda-forge的URL则需要在最后加上`/conda-forge`，鄙人在此处踩过坑，不加的话会出现404的错误。
正确使用mirrored_channels是这样的：*

```yaml
mirrored_channels: 
  conda-forge: 
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
    - https://conda.anaconda.org/conda-forge 
    - https://prefix.dev/conda-forge
```

*同时使用 channel_alias 和 mirrored_channels 时，mirrored_channels 会覆盖 channel_alias，mirrored_channels 优先级高于 channel_alias，所以如果使用mirrored_channels，一定要写对URL！！*

---

<a id="create-env"></a>

## 三. 创建环境

推荐 mamba，更快。

```bash
mamba create -n myai python=3.10
```

创建一个名为 `myai` 的环境，此名字自定，我在此只是举个例子，下同。

可使用如下指令查看创建的环境：

```bash
mamba env list
```

***在首次激活环境（mamba activate <env>）之前需要进行 `四. 初始化 Shell` 的操作。***

---

<a id="init-shell"></a>

## 四. 初始化 Shell （One-Time Setup）

**为什么需要初始化 shell？**

Mamba / Conda 需要在 shell（cmd.exe 或 PowerShell）中写入一些 **环境变量和 shell 函数**，才能正确使用 `mamba activate <env>` 与 `mamba deactivate` 。
- ***Shell 初始化只需在首次安装 Miniforge 时执行一次，成功后，以后创建环境 `create` 后即可直接 `activate` 。***

如果没有初始化，mamba 只能作为 **子进程运行**，无法修改父进程环境，就会报错：

```cmd
critical libmamba Shell not initialized
'mamba' is running as a subprocess and can't modify the parent shell.
```

初始化就是把这些配置写入你的 shell 配置文件。

---

<a id="init-shell-cmd"></a>

### 1. 初始化 cmd.exe

1. 打开 **Windows 命令提示符（cmd）**
2. 执行初始化命令：

```cmd
mamba shell init --shell cmd.exe
```

3. 你会看到提示，告诉你初始化成功，并且修改了某个用户配置文件（通常在 `%USERPROFILE%\_conda\_init.bat` ）
4. 关闭当前 cmd 窗口
5. 重新打开一个 cmd 窗口
6. 测试激活环境：

```cmd
mamba activate myai
```

如果成功，你会看到 `(myai)` 出现在命令提示符前面。

---

<a id="init-shell-PowerShell"></a>

### 2. 初始化 PowerShell

1. 打开 **PowerShell**（不是 cmd，空白处右击“在终端中打开”）
2. 永久允许当前用户执行脚本

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

- `CurrentUser` → 只修改当前用户，不影响系统其他用户
- `RemoteSigned` → 允许本地脚本运行，但从网上下载的脚本需要签名
- 当然也有临时的，可执行指令 `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` 适合一次性激活 mamba shell 或运行脚本，`-Scope Process` 表示只在当前 PowerShell 会话有效，关闭窗口就恢复原策略。
- 修改后可以用指令 `Get-ExecutionPolicy -Scope CurrentUser` 确认。若显示为 `RemoteSigned` 则说明修改成功，RemoteSigned 的安全性很高，只允许本地脚本执行，外部脚本需要签名。永远不要把策略设为 `Unrestricted`，不安全。

**注意：** 若不执行第2步，PowerShell 默认策略会阻止 mamba 写入的 profile 脚本运行，则在执行初始化命令后会出现：***无法加载文件 文档\WindowsPowerShell\profile.ps1，因为在此系统上禁止运行脚本。***

3. 执行初始化命令：

```powershell
mamba shell init --shell powershell
```

4. 初始化完成后，PowerShell 会修改 **用户配置文件**（通常是 `文档\WindowsPowerShell\profile.ps1` ）
5. 关闭当前 PowerShell 窗口
6. 重新打开 PowerShell
7. 测试激活环境：

```powershell
mamba activate myai
```

如果成功，你会看到 `(myai)` 出现在提示符前。

---

**注意事项：**

1. **每个 shell 都需要单独初始化**

   * cmd.exe 和 PowerShell 是两个不同的 shell，必须分别初始化

2. **如果之前初始化失败或报错**，可以用 `reinit` 重新初始化：

```cmd
# cmd.exe
mamba shell reinit --shell cmd.exe

# PowerShell
mamba shell reinit --shell powershell
```

---

<a id="miniforge-commands"></a>

## 五. miniforge指令

<a id="common-commands"></a>

### 🚀 常用指令

```bash
# ① 创建环境（推荐 mamba，更快）
mamba create -n <env> python=<python_version>
# 示例：mamba create -n myai python=3.10

# ② 激活环境
mamba activate <env>
# 或：conda activate <env>

# ③ 退出环境
mamba deactivate
# 或：conda deactivate

# ④ 查看所有虚拟环境
conda env list

# ⑤ 删除环境
mamba env remove -n <env>

# ⑥ 查看信息
mamba --version     # 输出 mamba 版本号
mamba info          # 查看 mamba 环境信息
#或 conda --version
# conda info
```

---

<a id="package-management-commands"></a>

### 📦 包管理常用指令

```bash
# 安装包
mamba install <package>
# 示例：mamba install numpy

# 安装带指定源的包
mamba install <package> -c <channel>

# 删除包
mamba remove <package>

# 更新单个包
mamba update <package>

# 更新所有包
mamba update --all

# 搜索包
mamba search <package>
# 示例：mamba search pytorch

# 查看当前环境已安装的包
conda list
```

---

<a id="env-config-commands"></a>

### 🔧 环境配置相关指令

```bash
# 查看当前 conda/mamba 配置
conda config --show

# 清理缓存和临时文件（非常常用）
mamba clean --all -y

# 添加镜像源（示例：清华镜像）
conda config --add channels <mirror_url>
# 示例：conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
```

---

<a id="env-export-migration"></a>

### 📂 环境导出与迁移

```bash
# 导出环境（复制给别人/备份）
conda env export > <file>.yml
# 示例：conda env export > environment.yml

# 通过 yml 文件创建环境
mamba env create -n <env> -f <file>.yml
# 示例：mamba env create -f environment.yml
```

---

<a id="ai-project-conda-commands"></a>

### 🧠 AI 项目常用 Conda 指令

```bash
# 安装 PyTorch（CPU 版本）
mamba install pytorch torchvision torchaudio -c conda-forge

# 安装 PyTorch（CUDA 12.1）
mamba install pytorch torchvision torchaudio cudatoolkit=12.1 -c conda-forge

# 安装 Transformers
pip install transformers
# Transformers 在 pip 上更新最快，推荐 pip 安装

# 安装 JupyterLab
mamba install jupyterlab

# 安装 OpenAI SDK
pip install openai
```
