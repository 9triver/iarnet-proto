# 迁移指南：从分散的 proto 到统一的 iarnet-proto

## 重构概述

本次重构将分散在各个仓库中的 proto 文件统一迁移到 `iarnet-proto` 仓库，采用**模式 B**（各业务仓库本地拉取 proto 源码，然后自行生成）进行管理。

## 目录结构

### iarnet-proto 仓库

```
iarnet-proto/
├── proto/                    # Proto 源文件
│   ├── common/              # 通用类型和消息
│   ├── ignis/               # Ignis 执行引擎
│   ├── resource/            # 资源管理
│   ├── application/         # 应用相关
│   └── global/              # 全局服务
├── scripts/                 # 生成脚本
│   ├── gen_go.sh           # Go 代码生成脚本
│   └── gen_python.sh       # Python 代码生成脚本
├── README.md               # 项目说明
├── USAGE.md                # 使用指南
└── .gitignore              # Git 忽略文件
```

### 业务仓库结构

各业务仓库（如 `iarnet`、`iarnet-global`）保持原有的输出目录结构，但 proto 源文件改为从 `iarnet-proto` 获取。

## 迁移步骤

### 1. 初始化 iarnet-proto 仓库

```bash
cd /path/to/iarnet-proto
git init
git add .
git commit -m "初始提交: 统一 proto 定义"
git tag v1.0.0
```

### 2. 在业务仓库中添加依赖

#### iarnet 仓库

```bash
cd /path/to/iarnet
git submodule add <iarnet-proto-repo-url> third_party/iarnet-proto
# 或直接克隆到同级目录
git clone <iarnet-proto-repo-url> ../iarnet-proto
```

#### iarnet-global 仓库

```bash
cd /path/to/iarnet-global
git submodule add <iarnet-proto-repo-url> third_party/iarnet-proto
# 或直接克隆到同级目录
git clone <iarnet-proto-repo-url> ../iarnet-proto
```

### 3. 更新生成脚本

各业务仓库的 `proto/protobuf-gen.sh` 已更新为调用 `iarnet-proto/scripts/` 中的脚本。

### 4. 测试生成

```bash
# 在 iarnet 仓库
cd /path/to/iarnet
./proto/protobuf-gen.sh

# 在 iarnet-global 仓库
cd /path/to/iarnet-global
./proto/protobuf-gen.sh
```

### 5. 清理旧文件（可选）

迁移完成后，可以考虑删除业务仓库中的 proto 源文件（保留生成脚本）：

```bash
# 在 iarnet 仓库
rm -rf proto/common proto/ignis proto/resource proto/application proto/global

# 在 iarnet-global 仓库
rm -rf proto/resource
```

**注意**: 删除前请确保：
1. `iarnet-proto` 仓库已包含所有 proto 文件
2. 生成脚本已测试通过
3. 已提交代码到版本控制

## 版本管理策略

### 版本号规则

- 使用语义化版本号：`v<major>.<minor>.<patch>`
- Major: 不兼容的协议变更
- Minor: 向后兼容的新功能
- Patch: 向后兼容的 bug 修复

### 锁定版本

业务仓库通过 submodule 的 commit 或 tag 锁定版本：

```bash
cd third_party/iarnet-proto
git checkout v1.0.0
cd ../..
git add third_party/iarnet-proto
git commit -m "锁定 iarnet-proto 版本为 v1.0.0"
```

## 协议修改规范

### 向后兼容原则

1. **只允许添加新字段**：新增字段使用新的字段编号
2. **禁止删除字段**：已删除的字段编号不能重用
3. **禁止修改字段类型**：如需修改，应创建新字段
4. **服务方法变更**：新增方法可以，删除或修改需要新版本

### 不兼容变更处理

如果必须进行不兼容变更：

1. 创建新的 major 版本（如 `v2.0.0`）
2. 在 `iarnet-proto` 中创建新目录或使用新的包名
3. 通知所有依赖的业务仓库进行升级

## 常见问题

### Q: 如何确保所有仓库使用相同版本的 proto？

A: 通过 git submodule 的 commit 锁定版本，并在 CI/CD 中检查 submodule 版本一致性。

### Q: 多个仓库需要不同的生成参数怎么办？

A: 各业务仓库的生成脚本可以自定义输出目录和参数，但应调用 `iarnet-proto/scripts/` 中的基础脚本。

### Q: 如何回滚到旧版本？

A: 切换 submodule 到旧版本：

```bash
cd third_party/iarnet-proto
git checkout <old-version-tag>
cd ../..
git add third_party/iarnet-proto
git commit -m "回滚 iarnet-proto 到旧版本"
```

## 后续优化建议

1. **CI/CD 集成**：在 CI 中自动检查 proto 版本一致性
2. **自动化测试**：添加 proto 兼容性测试
3. **文档完善**：为每个 proto 文件添加详细注释
4. **版本发布流程**：建立规范的版本发布流程

