# unirtos_vfs_demos

本仓库推荐通过 unirtos-cli 的 demo 工作流使用，以保证创建、环境拉取和编译流程一致。

## 功能描述

本 Demo 展示虚拟文件系统（VFS）相关能力的基础使用方式，适合作为文件读写类应用开发参考。

- 演示文件创建、写入、读取、定位、删除等基础流程
- 提供目录创建、遍历、统计、递归删除的完整样例
- 便于扩展日志落盘、配置持久化与存储健康检查能力

## 快速上手

### 1. 安装 UniRTOS 工具链

- [开发准备](https://www.quectel.com.cn/unirtos/docs?docs_page=快速上手/开发准备/开发准备.html)
- [安装交叉编译工具链](https://www.quectel.com.cn/unirtos/docs?docs_page=快速上手/环境搭建/环境搭建.html)
- [安装 Python3](https://www.python.org/downloads/)
- [安装 git](https://git-scm.com)
- 安装 `unirtos-cli`：`pip install unirtos-cli`

以上工具安装完成后，确认以下命令可用：

```bash
python --version # Python3
git --version
unirtos --version # 1.0.5 及以上版本
unirtos-cli version # 1.0.11 及以上版本
```

### 2. 使用 unirtos-cli 拉取 demo

先查看可用 demo 与版本：

```bash
unirtos-cli ls-demos
```

创建本 demo 工程：

```bash
unirtos-cli new -r unirtos_vfs_demos
```

如需指定版本：

```bash
unirtos-cli new -r unirtos_vfs_demos -v 1.0.0
```

### 3. 进入工程并编译

```bash
cd unirtos_vfs_demos-1.0.0
unirtos-cli env-setup
unirtos-cli build
```

## 常用命令

```bash
# 打开 SDK 菜单配置
unirtos-cli menuconfig

# 清理构建产物
unirtos-cli clean
```

## 代码概览

#### 项目结构

```
unirtos_vfs_demos/
├── CMakeLists.txt        # CMake 构建配置
├── env_config.json       # 构建模块与环境配置
├── menuconfig/           # SDK 菜单配置目录
├── vfs_file_demo.c       # VFS 文件与目录操作示例
└── README.md             # 本文件
```

#### 示例工作流程

```
程序启动
	↓
调用 unir_vfs_demo_init()
	↓
创建名为 "vfs_demo" 的任务
	↓
任务延时约 10 秒后进入 unir_vfs_task_handler()
	↓
执行文件测试 unir_vfs_demo_file_test()
  - open/create -> write -> fstat -> lseek -> read -> close -> unlink
	↓
执行目录测试 unir_vfs_demo_dir_test()
  - opendir/mkdir -> creat 文件 -> mkdir 子目录 -> readdir 列表
  - closedir -> rmdir(空目录限制演示) -> rmdir_recursive 清理
```

#### 主要 API 接口

##### unir_vfs_demo_init
VFS demo 初始化函数
- 创建任务并设置栈大小、优先级
- 启动 VFS 文件和目录联合测试流程

##### unir_vfs_task_handler
任务处理函数
- 延时后依次执行文件测试与目录测试
- 便于系统启动后再进行存储读写操作

##### unir_vfs_demo_file_test
文件操作测试函数
- 通过 qosa_vfs_open 创建并打开测试文件
- 写入固定内容并通过 qosa_vfs_fstat 获取文件大小
- 使用 qosa_vfs_lseek 回到起始位置后读取并打印
- 测试结束后关闭并删除测试文件

##### unir_vfs_demo_dir_test
目录操作测试函数
- 打开目录，不存在时自动创建
- 创建测试文件与子目录
- 调用目录遍历函数输出目录项信息
- 演示 qosa_vfs_rmdir 与 qosa_vfs_rmdir_recursive 的差异

##### unir_vfs_demo_dir_list
目录遍历与统计函数
- 遍历目录项并区分普通文件与子目录
- 输出文件名、大小、权限等信息
- 统计子目录总大小并输出

#### 日志展示

运行后可看到类似输出：

```
[V/LOG] test file
[D/LOG] write ret=10
[D/LOG] size=10
[D/LOG] read ret=10,data=[1234567890]
[V/LOG] test dir
[V/LOG] File name: vfs_test1.txt
[D/LOG] remove dir ret=...
```

目录删除阶段会先调用 qosa_vfs_rmdir 演示空目录限制，再通过 qosa_vfs_rmdir_recursive 完成递归清理。

## 配置说明

关键配置集中在 vfs_file_demo.c：

- UNIR_VFS_DEMO_TASK_STACK_SIZE：示例任务栈大小，默认 4096
- 测试文件路径：./vfs_test.txt
- 测试目录路径：./testdir
- 测试子文件：./testdir/vfs_test1.txt、./testdir/vfs_test2.txt
- 测试子目录：./testdir/subdir

说明：
- 若文件或目录创建失败，可结合 qosa_get_errno 输出排查存储挂载与权限问题。
- 演示代码默认在测试末尾删除文件与目录，如需保留结果可注释对应 unlink/rmdir 语句。
- 不同平台文件系统实现可能对路径、权限和容量上限有差异，建议根据目标模块实际能力调整。

## 技术社区

技术社区：https://forumschinese.quectel.com/c/66-category/66

## 贡献指南

欢迎参与共建，建议按以下方式提交：
- 提交前先执行一次基础验证：env-setup、build、clean。
- 使用清晰的提交说明，描述改动目的、影响范围和验证结果。
- 新增功能或行为变化时，同步更新 README 与相关文档。
- 通过 Issue 或 Pull Request 提交问题修复与功能改进。
