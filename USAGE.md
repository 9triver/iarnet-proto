# iarnet-proto 使用指南

## 在业务仓库中使用 iarnet-proto

### 步骤 1: 添加 iarnet-proto 依赖

有两种方式添加 iarnet-proto 到你的项目：

#### 方式 A: Git Submodule（推荐）

```bash
# 在业务仓库根目录执行
git submodule add <iarnet-proto-repo-url> third_party/iarnet-proto
git submodule update --init --recursive
```

#### 方式 B: 直接克隆

```bash
# 克隆到项目同级目录
git clone <iarnet-proto-repo-url> ../iarnet-proto

# 或克隆到 third_party 目录
git clone <iarnet-proto-repo-url> third_party/iarnet-proto
```

### 步骤 2: 创建生成脚本

在你的业务仓库中创建 `proto/protobuf-gen.sh`，参考以下模板：

```bash
#!/bin/bash
set -e

PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"

# 指定 iarnet-proto 路径（支持多种方式）
IARNET_PROTO_DIR="${IARNET_PROTO_DIR:-${PROJECT_ROOT}/../iarnet-proto}"
if [ ! -d "$IARNET_PROTO_DIR" ]; then
    IARNET_PROTO_DIR="${PROJECT_ROOT}/third_party/iarnet-proto"
fi

IARNET_PROTO_PROTO_DIR="${IARNET_PROTO_DIR}/proto"
IARNET_PROTO_SCRIPTS_DIR="${IARNET_PROTO_DIR}/scripts"

# 生成 Go 代码
GO_OUTPUT="${PROJECT_ROOT}/internal/proto"
bash "$IARNET_PROTO_SCRIPTS_DIR/gen_go.sh" "$IARNET_PROTO_PROTO_DIR" "$GO_OUTPUT"

# 生成 Python 代码（如果需要）
PY_OUTPUT="${PROJECT_ROOT}/proto"
bash "$IARNET_PROTO_SCRIPTS_DIR/gen_python.sh" "$IARNET_PROTO_PROTO_DIR" "$PY_OUTPUT"
```

### 步骤 3: 运行生成脚本

```bash
./proto/protobuf-gen.sh
```

## 版本管理

### 锁定版本

使用 git submodule 时，可以通过 commit 或 tag 锁定版本：

```bash
# 切换到指定版本
cd third_party/iarnet-proto
git checkout v1.0.0
cd ../..
git add third_party/iarnet-proto
git commit -m "锁定 iarnet-proto 版本为 v1.0.0"
```

### 更新版本

```bash
# 更新 submodule 到最新版本
cd third_party/iarnet-proto
git pull origin main
cd ../..
git add third_party/iarnet-proto
git commit -m "更新 iarnet-proto 到最新版本"
```

## 环境变量

可以通过环境变量自定义行为：

- `IARNET_PROTO_DIR`: 指定 iarnet-proto 的路径
- `PROTOC_CMD`: 指定 protoc 命令（默认: `python -m grpc_tools.protoc`）

示例：

```bash
export IARNET_PROTO_DIR=/path/to/custom/iarnet-proto
export PROTOC_CMD=/usr/local/bin/protoc
./proto/protobuf-gen.sh
```

## 常见问题

### Q: 找不到 iarnet-proto 目录？

A: 检查以下几点：
1. 是否已添加 submodule 或克隆了仓库
2. 路径是否正确（默认查找 `../iarnet-proto` 和 `third_party/iarnet-proto`）
3. 可以通过 `IARNET_PROTO_DIR` 环境变量指定路径

### Q: 生成的代码导入路径不对？

A: 检查 proto 文件中的 `go_package` 选项是否正确设置。如果需要修改导入路径，可以在生成后使用 `sed` 命令批量替换。

### Q: 如何只生成部分 proto 文件？

A: 可以修改生成脚本，只调用需要的部分，或者直接使用 `protoc` 命令手动生成。

