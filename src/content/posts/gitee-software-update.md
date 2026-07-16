---
title: 使用Gitee实现免费的软件更新服务
published: 2026-07-06
description: '记录将Gitee用作软件自动更新的完整方案，包括GitHub Actions部署、版本控制、文件下载等实现细节'
image: ''
tags: [Gitee, GitHub Actions, 软件更新, CI/CD]
category: 'Share'
draft: false
---
在开发桌面应用时，软件更新是一个必不可少的功能。本文记录了一套将 Gitee 作为软件更新服务的完整方案，利用 GitHub Actions 实现自动部署，通过 Gitee API 实现版本检测和文件下载。
:::warning
`本文只提供思路，代码均由AI生成，仅供参考。`
:::

## 整体架构

整个更新流程分为三个主要部分：

1. **GitHub Actions 构建部署**：编译项目，将产物上传到专门的文件仓库，并创建对应版本的 release
2. **Gitee 仓库同步**：同步文件仓库到 Gitee，利用标签来追踪版本
3. **客户端更新检测**：通过 Gitee API 获取最新版本信息，对比本地版本，下载并安装更新

```
GitHub 源码仓库
    |
    v
GitHub Actions (编译)
    |
    v
GitHub 文件仓库 (release)
    |
    v
Gitee 文件仓库 (同步)
    |
    v
客户端检测更新 → 下载 → 安装
```

## GitHub Actions 配置

首先需要在 GitHub 上创建一个专门用于存放编译产物的仓库（如 `my-app-releases`），然后在源码仓库配置 Actions 工作流。

```yaml
name: Deploy Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Build application
        run: python build.py

      - name: Upload to release repo
        uses: EndBug/add-and-commit@v9
        with:
          args: '-m "chore: release ${{ github.ref_name }}"'
          add: 'dist/my-app-v${{ github.ref_name }}.7z'
          cwd: '../my-app-releases'

      - name: Create tag
        uses: mathieudutour/github-tag-action@v6.1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          tag: ${{ github.ref_name }}
          repo: username/my-app-releases
```

**关键要点：**

- 文件名必须包含版本号，如 `my-app-v1.0.0.7z`，方便客户端匹配
- 使用 7z 压缩可以显著减小文件体积
- 通过 tag 触发部署，确保版本号与 tag 一致
- 文件直接上传到仓库而非 release，这样 Gitee 同步时可以包含文件
- 就算要下载全部文件，也不要直接下载仓库zip包，安全审查更严格部分ip会被限制下载，可以把所有文件压缩上传到仓库，然后通过raw链接下载

## 仓库清理机制

Gitee 仓库有大小限制，随着版本迭代，编译文件会不断累积，导致仓库体积过大。因此需要在文件仓库中配置一个清理 Actions，定期清理旧版本文件，只保留最新的几个版本。

### 清理策略

- **触发条件**：当仓库中的文件数量超过 1 个时触发清理
- **清理方式**：只保留最新的 1 个版本文件和对应的 tag，删除所有旧版本文件和 tag
- **实现方式**：新建 git 仓库，只保留需要的文件，然后强制覆盖远程仓库

### 清理 Actions 配置

在文件仓库（`my-app-releases`）中创建清理工作流：

```yaml
name: Cleanup Old Releases

on:
  push:
    branches:
      - main

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: List files
        id: list_files
        run: |
          files=$(ls *.7z 2>/dev/null | wc -l)
          echo "file_count=$files" >> "$GITHUB_OUTPUT"

      - name: Cleanup if needed
        if: steps.list_files.outputs.file_count > 1
        run: |
          git checkout --orphan new_main
          git rm -rf .
          
          latest_file=$(ls *.7z 2>/dev/null | sort -V | tail -1)
          git add "$latest_file"
          
          git commit -m "Cleanup: keep only latest version"
          git push origin new_main --force
          
          for tag in $(git tag -l); do
            file="my-app-${tag}.7z"
            if [ ! -f "$file" ]; then
              git push origin :refs/tags/"$tag"
              git tag -d "$tag"
            fi
          done

      - name: Reset main branch
        if: steps.list_files.outputs.file_count > 1
        run: |
          git checkout main
          git reset --hard new_main
          git push origin main --force
```

**清理流程详解：**

1. **统计文件数量**：检查仓库中 `.7z` 文件的数量
2. **创建新分支**：使用 `git checkout --orphan` 创建一个没有历史记录的新分支
3. **清空文件**：删除所有文件，只保留最新的 1 个版本文件
4. **提交并强推**：提交新分支并强制推送到远程仓库
5. **清理旧标签**：删除那些没有对应文件的旧标签
6. **重置主分支**：将主分支重置到新分支并强制推送

**注意事项：**

- 使用 `--force` 参数会强制覆盖远程仓库，确保清理生效
- 只删除没有对应文件的标签，保留与最新版本对应的标签
- Gitee 同步会自动同步清理后的仓库状态，但需要注意同步延迟

## 文件大小优化

Gitee 的仓库文件和 release 都限制 100MB，因此需要对编译产物进行大小优化：

### 编译时排除不必要的导入

在编译前分析项目依赖，只导入实际使用的模块，避免打包未使用的库：

```python title="build.py"
import sys
import os
from pathlib import Path

def analyze_imports(source_dir):
    used_modules = set()
    
    for py_file in Path(source_dir).rglob('*.py'):
        with open(py_file, 'r', encoding='utf-8') as f:
            content = f.read()
            for line in content.split('\n'):
                if line.startswith('import ') or line.startswith('from '):
                    parts = line.split()
                    if len(parts) >= 2:
                        module = parts[1].split('.')[0]
                        used_modules.add(module)
    
    return used_modules

def filter_requirements(requirements_path, used_modules):
    with open(requirements_path, 'r') as f:
        lines = f.readlines()
    
    filtered = []
    for line in lines:
        line = line.strip()
        if not line or line.startswith('#'):
            filtered.append(line)
            continue
        
        pkg_name = line.split('=')[0].split('>')[0].split('<')[0].strip()
        if pkg_name.lower() in [m.lower() for m in used_modules]:
            filtered.append(line)
    
    with open(requirements_path, 'w') as f:
        f.write('\n'.join(filtered) + '\n')

if __name__ == '__main__':
    used_modules = analyze_imports('src')
    filter_requirements('requirements.txt', used_modules)
```

**注意：** 这种方式需要谨慎使用，因为有些模块可能是动态导入的，分析工具无法检测到。建议在测试环境中验证后再使用。

### 编译后清理不必要的运行库文件

以 PyInstaller 为例，可以通过配置文件排除不必要的运行库：

```python title="build.spec"
block_cipher = None

a = Analysis(
    ['main.py'],
    pathex=['src'],
    binaries=[],
    datas=[],
    hiddenimports=[],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[
        'tkinter',
        'matplotlib',
        'numpy',
        'scipy',
        'pandas',
        'PyQt5.QtWebEngine',
        'PyQt5.QtWebKit',
    ],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    cipher=block_cipher,
    noarchive=False,
)

pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

exe = EXE(
    pyz,
    a.scripts,
    [],
    exclude_binaries=True,
    name='my-app',
    debug=False,
    bootloader_ignore_signals=False,
    strip=True,
    upx=True,
    console=False,
)

coll = COLLECT(
    exe,
    a.binaries,
    a.zipfiles,
    a.datas,
    strip=True,
    upx=True,
    upx_exclude=[],
    name='my-app',
)
```

**关键优化项：**

- `excludes`：排除不需要的模块，如 `tkinter`、`matplotlib` 等
- `strip=True`：去除二进制文件中的符号表，减小体积
- `upx=True`：使用 UPX 压缩可执行文件

### 清理运行库文件

编译完成后，可以手动清理一些不必要的文件：

```python title="cleanup.py"
import os
import shutil

def cleanup_dist(dist_dir):
    remove_patterns = [
        '*.pdb',
        '*.pyc',
        '*.pyo',
        '__pycache__',
        '*.dll',
        '*.so',
    ]
    
    for root, dirs, files in os.walk(dist_dir):
        for file in files:
            file_path = os.path.join(root, file)
            for pattern in remove_patterns:
                if file.endswith(pattern.strip('*')):
                    os.remove(file_path)
                    break
        
        for dir_name in dirs:
            if dir_name == '__pycache__':
                dir_path = os.path.join(root, dir_name)
                shutil.rmtree(dir_path)

if __name__ == '__main__':
    cleanup_dist('dist/my-app')
```

**注意：** 需要根据实际使用的库来决定删除哪些文件，避免删除程序运行必需的 DLL 或 SO 文件。

### 压缩为 7z 文件

```python
import subprocess
import os

def compress_with_7z(source_dir, output_path):
    cmd = [
        '7z', 'a', '-t7z', '-mx=9', '-mfb=64', '-md=32m', '-ms=on',
        output_path, source_dir
    ]
    subprocess.run(cmd, check=True)

if __name__ == '__main__':
    compress_with_7z('dist/app', 'dist/my-app-v1.0.0.7z')
```

### 分离资源文件

将资源文件（如图片、配置文件等）放到单独的 Gitee 仓库，在软件启动时检查资源更新：

```python
import requests
import hashlib
import os

RESOURCE_REPO = "https://gitee.com/username/my-app-resources/raw/main/"

def check_resource_update(local_path, resource_name):
    remote_url = f"{RESOURCE_REPO}{resource_name}"
  
    local_hash = md5_file(local_path)
    remote_hash = get_remote_file_hash(remote_url)
  
    if local_hash != remote_hash:
        download_resource(remote_url, local_path)

def md5_file(file_path):
    md5 = hashlib.md5()
    with open(file_path, 'rb') as f:
        for chunk in iter(lambda: f.read(8192), b''):
            md5.update(chunk)
    return md5.hexdigest()

def get_remote_file_hash(url):
    response = requests.head(url)
    return response.headers.get('ETag', '').strip('"')
```

这种方式还可以实现资源的增量更新，只下载变化的文件。

## Gitee 仓库同步

在 Gitee 上创建一个与 GitHub 文件仓库同名的仓库，然后配置同步：

1. 在 Gitee 仓库设置中找到「仓库同步」
2. 添加 GitHub 仓库地址和访问 token
3. 设置同步方向为「从 GitHub 同步到 Gitee」

**注意事项：**

- Gitee 的同步功能**不会同步 release**，只会同步代码和标签（tag）
- 需要通过添加或删除 release 来控制版本更新的发布时机
- **自动同步 API 可能已关闭**：目前 Gitee 的自动同步功能疑似出现问题，可能无法自动触发同步，需要手动到仓库页面点击「同步」按钮

## 独立更新程序设计

建议将更新功能独立编写为一个单独的程序（如 `updater.exe`），由主程序调用执行更新。这样可以避免在更新过程中文件被占用，同时也便于维护和调试。

### 更新程序架构

```
主程序 (main.exe)
    |
    v
检查更新 → 发现新版本 → 启动更新程序 (updater.exe) → 退出主程序
                                    |
                                    v
                            下载更新文件 → 解压 → 替换文件 → 重启主程序
```

### 版本检测（使用 Gitee API + 私钥）

Gitee API 调用需要使用私钥（access_token）进行身份验证，且有调用频率限制。对于小文件（如版本信息）可以直接用 raw URL 获取，大文件下载需要使用 API：

```python title="updater.py"
import requests
import re
import time

class GiteeUpdater:
    def __init__(self, repo_owner, repo_name, access_token=None):
        self.repo_owner = repo_owner
        self.repo_name = repo_name
        self.api_base = "https://gitee.com/api/v5"
        self.access_token = access_token
        self.headers = {}
        if access_token:
            self.headers['Authorization'] = f"token {access_token}"
    
    def _make_request(self, url, method='GET', **kwargs):
        max_retries = 3
        retry_delay = 5
        
        for attempt in range(max_retries):
            try:
                response = requests.request(method, url, headers=self.headers, **kwargs)
                if response.status_code == 429:
                    wait_time = int(response.headers.get('Retry-After', retry_delay))
                    time.sleep(wait_time)
                    continue
                response.raise_for_status()
                return response.json()
            except requests.exceptions.RequestException as e:
                if attempt < max_retries - 1:
                    time.sleep(retry_delay * (attempt + 1))
                    continue
                raise
    
    def get_latest_version(self):
        url = f"{self.api_base}/repos/{self.repo_owner}/{self.repo_name}/tags"
        tags = self._make_request(url)
        
        if not tags:
            return None
        
        version_pattern = re.compile(r'^v(\d+\.\d+\.\d+)$')
        versions = []
        
        for tag in tags:
            match = version_pattern.match(tag['name'])
            if match:
                versions.append((tag['name'], match.group(1)))
        
        if not versions:
            return None
        
        versions.sort(key=lambda x: tuple(map(int, x[1].split('.'))), reverse=True)
        return versions[0][0]
    
    def compare_versions(self, current_version, latest_version):
        current = tuple(map(int, current_version.lstrip('v').split('.')))
        latest = tuple(map(int, latest_version.lstrip('v').split('.')))
        return latest > current
    
    def get_file_download_url(self, version, filename):
        url = f"{self.api_base}/repos/{self.repo_owner}/{self.repo_name}/contents/{filename}"
        params = {'ref': version}
        data = self._make_request(url, params=params)
        
        if isinstance(data, dict) and 'download_url' in data:
            return data['download_url']
        return None
```

**API 安全注意事项：**

- **私钥管理**：不要将 `access_token` 硬编码在代码中，建议通过配置文件或环境变量传入
- **调用限制**：Gitee API 有调用频率限制（普通用户每小时 60 次），需要实现重试机制
- **权限范围**：创建 access_token 时只授予必要的权限（如 `projects:read`）

### 文件下载与安装

```python title="updater.py"
import shutil
import tempfile
import subprocess
import sys

def download_and_install(self, version):
    filename = f"my-app-{version}.7z"
    download_url = self.get_file_download_url(version, filename)
    
    if not download_url:
        raise Exception(f"无法获取文件下载链接: {filename}")
    
    with tempfile.TemporaryDirectory() as temp_dir:
        download_path = os.path.join(temp_dir, filename)
        self.download_file(download_url, download_path)
        self.extract_7z(download_path, temp_dir)
        self.install_update(temp_dir)

def download_file(self, url, save_path):
    max_retries = 3
    retry_delay = 10
    
    for attempt in range(max_retries):
        try:
            response = requests.get(url, headers=self.headers, stream=True)
            if response.status_code == 429:
                wait_time = int(response.headers.get('Retry-After', retry_delay))
                time.sleep(wait_time)
                continue
            response.raise_for_status()
            
            with open(save_path, 'wb') as f:
                for chunk in response.iter_content(chunk_size=8192):
                    f.write(chunk)
            return
        except requests.exceptions.RequestException as e:
            if attempt < max_retries - 1:
                time.sleep(retry_delay * (attempt + 1))
                continue
            raise

def extract_7z(self, file_path, dest_dir):
    import py7zr
    with py7zr.SevenZipFile(file_path, mode='r') as z:
        z.extractall(dest_dir)

def install_update(self, source_dir):
    app_dir = os.path.dirname(os.path.abspath(sys.argv[0]))
    
    for item in os.listdir(source_dir):
        s = os.path.join(source_dir, item)
        d = os.path.join(app_dir, item)
        
        if os.path.isdir(s):
            if os.path.exists(d):
                shutil.rmtree(d)
            shutil.move(s, d)
        else:
            if os.path.exists(d):
                os.remove(d)
            shutil.move(s, d)

def restart_app(self):
    app_path = os.path.join(os.path.dirname(os.path.abspath(sys.argv[0])), 'main.exe')
    subprocess.Popen([app_path])

if __name__ == '__main__':
    updater = GiteeUpdater(
        repo_owner="username",
        repo_name="my-app-releases",
        access_token="your-access-token"
    )
    
    current_version = "v1.0.0"
    latest_version = updater.get_latest_version()
    
    if updater.compare_versions(current_version, latest_version):
        updater.download_and_install(latest_version)
        updater.restart_app()
```

### 主程序调用更新程序

```python title="main.py"
import subprocess
import sys
import os

def check_and_update():
    updater_path = os.path.join(os.path.dirname(sys.argv[0]), 'updater.exe')
    
    if os.path.exists(updater_path):
        subprocess.Popen([updater_path])
        sys.exit(0)

if __name__ == '__main__':
    check_and_update()
    run_main_app()
```

**调用流程：**

1. 主程序启动时检测更新程序是否存在
2. 如果存在，启动更新程序并退出自身
3. 更新程序完成更新后重启主程序

### Raw URL vs API 下载

| 方式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Raw URL** | 无需认证，简单直接 | 有下载限制，大文件可能失败 | 小文件（如版本信息） |
| **API 下载** | 稳定可靠，支持大文件 | 需要认证，有调用限制 | 大文件（如更新包） |

**建议策略：**

- 使用 raw URL 获取版本信息（小文件，无需认证）
- 使用 API 下载更新包（大文件，需要认证）
- 实现重试机制和限流处理

## 为什么这样设计

### 核心设计理由：文件走仓库，版本走 release

这是整个方案的核心思路：

- **文件存储**：编译后的文件直接上传到 GitHub 文件仓库，通过 Gitee 同步功能同步到 Gitee 仓库
- **文件下载**：通过 Gitee API 获取文件下载链接进行下载（需使用 access_token 认证）
- **版本控制**：通过 Gitee release 和标签（tag）来标记和检测版本

**为什么这样设计：**

1. **上传文件到仓库容易**：通过 GitHub Actions 将文件推送到 GitHub 文件仓库，再通过 Gitee 同步功能自动同步到 Gitee，无需手动操作
2. **下载不限速**：通过 Gitee API 获取文件下载链接进行下载，下载速度稳定，不受 release 限速影响
3. **获取下载链接容易**：通过 Gitee API `/repos/{owner}/{repo}/contents/{filename}` 可以直接获取文件下载链接，只需一次 API 调用
4. **同步不包含 release**：Gitee 的仓库同步功能只同步代码和标签（tag），不会同步 release。因此文件必须放在仓库中才能被同步
5. **大文件不支持 raw 下载**：Gitee 的 raw URL 对大文件有限制，无法直接下载几十兆的文件，必须通过 API 获取下载链接

### 为什么不直接在 Actions 上传文件到 Gitee

1. **上传速度慢**：GitHub Actions 的网络环境上传文件到 Gitee 非常慢，几十兆的文件可能需要半小时甚至更久
2. **Gitee Release 限速**：通过 API 上传文件到 Gitee release 有严格的限速限制

### 为什么选择 Gitee

1. **国内访问快**：对于国内用户来说，Gitee 的下载速度远快于 GitHub
2. **API 稳定**：Gitee API 响应稳定，适合作为客户端更新服务
3. **免费额度高**：免费仓库有足够的空间存放编译产物
4. **仓库同步功能**：Gitee 提供了从 GitHub 同步仓库的功能，可以自动同步代码和标签

## 使用流程总结

1. **开发阶段**：在 GitHub 源码仓库开发，提交代码
2. **发布版本**：创建新的 tag（如 `v1.0.0`），触发 GitHub Actions
3. **构建部署**：Actions 编译项目，压缩文件，上传到文件仓库的 release
4. **自动同步**：Gitee 自动同步文件仓库的代码和标签
5. **创建 Release**：在 Gitee 文件仓库手动或通过 API 创建对应版本的 release
6. **客户端检测**：软件启动时调用 Gitee API 获取最新版本，对比本地版本
7. **下载更新**：检测到更新后下载文件，解压并替换现有文件
8. **完成更新**：重启软件完成更新

这套方案充分利用了 GitHub Actions 的构建能力和 Gitee 的国内访问优势，实现了一套完整的软件自动更新流程。
