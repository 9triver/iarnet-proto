# iarnet-proto

统一的 Protobuf 协议定义仓库，供多个模块和仓库共享使用。

## 目录结构

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
│   ├── gen_python.sh       # Python 代码生成脚本
│   └── gen_all.sh          # 生成所有语言代码
└── README.md               # 本文件
```

## 使用方法

### 方式一：Git Submodule（推荐）

在业务仓库中添加 submodule：

```bash
git submodule add <iarnet-proto-repo-url> third_party/iarnet-proto
git submodule update --init --recursive
```

### 方式二：直接克隆

```bash
git clone <iarnet-proto-repo-url> third_party/iarnet-proto
```

### 生成代码

各业务仓库应创建自己的生成脚本，调用 `iarnet-proto/scripts/` 中的脚本。

示例（在业务仓库中）：

```bash
#!/bin/bash
PROTO_DIR="$(pwd)/third_party/iarnet-proto"
OUTPUT_DIR="$(pwd)/internal/proto"

# 生成 Go 代码
bash "$PROTO_DIR/scripts/gen_go.sh" "$PROTO_DIR/proto" "$OUTPUT_DIR"
```

## 版本管理

- 使用语义化版本号（如 v1.0.0）
- 每个版本打 git tag
- 业务仓库通过 submodule 的 commit 或 tag 锁定版本

## 协议修改规范

1. **向后兼容原则**：只允许添加新字段，禁止删除或修改现有字段
2. **字段编号**：新增字段使用新的编号，不要重用已删除的字段号
3. **版本升级**：不兼容变更需要创建新的 major 版本

