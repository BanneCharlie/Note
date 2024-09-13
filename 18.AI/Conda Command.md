# Conda 常用指令

`conda` 是 Anaconda 和 Miniconda 中使用的包管理器和环境管理器工具。它用于管理 Python 和其他语言的包、依赖项以及创建和维护隔离的环境。以下是一些常用的 `conda` 命令及其用途：

## 常用的 `conda` 命令

### 1. 环境管理

- **创建新环境**：
    
    conda create --name myenv
    
    创建一个名为 `myenv` 的新环境。
    
- **指定 Python 版本创建环境**：
    
    conda create --name myenv python=3.8
    
    创建一个名为 `myenv` 的新环境，并指定 Python 版本为 3.8。
    
- **激活环境**：
    
    conda activate myenv
    
    激活 `myenv` 环境。
    
- **关闭环境**：
    
    conda deactivate
    
    关闭当前激活的环境。
    
- **列出所有环境**：
    
    conda env list
    
    或者简写：
    
    conda info --envs
    
    列出系统中所有已创建的 conda 环境。
    
- **删除环境**：
    
    conda remove --name myenv --all
    
    删除名为 `myenv` 的环境及其所有包。
    

### 2. 包管理

- **安装包**：
    
    conda install package_name
    
    在当前激活的环境中安装 `package_name` 包。
    
- **指定版本安装包**：
    
    conda install package_name=1.0.4
    
    安装指定版本的包。
    
- **更新包**：
    
    conda update package_name
    
    更新 `package_name` 包到最新版本。
    
- **更新 conda**：
    
    conda update conda
    
    将 `conda` 自身更新到最新版本。
    
- **删除包**：
    
    conda remove package_name
    
    从当前环境中删除 `package_name` 包。
    
- **列出环境中已安装的包**：
    
    conda list
    
    列出当前激活环境中已安装的所有包。
    

### 3. 环境的导出与导入

- **导出环境到文件**：
    
    conda env export > environment.yml
    
    将当前环境导出到 `environment.yml` 文件中。
    
- **从文件中创建环境**：
    
    conda env create -f environment.yml
    
    根据 `environment.yml` 文件创建一个新环境。
    

### 4. 搜索包

- **查找包**：
    
    conda search package_name
    
    搜索并显示 `conda` 仓库中 `package_name` 包的信息。
    

### 5. 查看帮助

- **查看帮助信息**：
    
    conda --help
    
    显示 `conda` 命令的帮助信息和用法。
    

### 6. 清理缓存

- **清理已下载的包缓存**：
    
    conda clean --tarballs
    
    清理 conda 已下载的包的缓存。
    
- **清理未使用的包**：
    
    conda clean --packages
    
    清理未使用的包。
    
- **清理所有缓存**：
    
    conda clean --all
    
    清理所有缓存，包括未使用的包、包缓存、索引缓存等。
    

## 组合与示例

例如，如果你想创建一个新的环境 `myenv`，使用 Python 3.8 并安装 `numpy` 和 `pandas`，可以执行：

conda create --name myenv python=3.8 numpy pandas

然后激活该环境：

conda activate myenv

这将启动 `myenv` 环境并准备好使用你指定的包。

这些常用命令可以帮助你更有效地管理 conda 环境和包。你可以根据需求组合使用这些命令来完成各种任务。

