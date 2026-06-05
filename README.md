# ShrimpSend public-ops（公开配置样例）

本仓库提供 **ultrasend / ShrimpSend** 运维配置目录的**公开样例**，所有值为占位符（`CHANGE_ME`、`price_xxx`、空字符串等），**不可直接用于生产**。

维护者请 fork 或复制为私有仓库，填入真实密钥后使用；自托管用户可 clone 到业务仓平级目录作为起点。

## 与私有 ops 的区别

| 仓库 | 用途 |
|------|------|
| [shrimpsend/public-ops](https://github.com/shrimpsend/public-ops)（本仓） | 公开样例，无真实密钥 |
| `git@github.com:shrimpsend/ops.git`（私有） | 官方生产配置，仅维护者可访问 |

## 推荐目录布局

业务仓与 ops 平级放置，脚本会自动发现 `../ops`：

```
/path/to/
├── ultrasend/          # git clone git@github.com:shrimpsend/shrimpsend.git
└── ops/                # git clone git@github.com:shrimpsend/public-ops.git
```

## ops 目录解析顺序

业务仓脚本（`sync-to-local.sh`、`sync-to-build-machine.sh`、`deploy.sh` 等）按以下顺序定位 ops：

1. **环境变量** `ULTRASEND_OPS_DIR`（若已设置）
2. **平级目录** `../ops`（相对业务仓根目录）
3. 若均未找到或未通过校验 → 报错并提示 clone 方式

### 校验特征

ops 根目录须同时满足：

- 存在 marker 文件 `.ultrasend-ops`，首行内容为 `ultrasend-ops`
- 至少存在一个配置子目录：`cn/`、`overseas/`、`local/`、`flutter/`、`web/`、`harmonyos/`

## 快速开始

```bash
# 1. clone 业务仓与 ops 样例（平级）
git clone git@github.com:shrimpsend/shrimpsend.git ultrasend
git clone git@github.com:shrimpsend/public-ops.git ops
cd ultrasend

# 2. 本地开发：同步样例配置（需 MySQL）
./scripts/deploy-local.sh

# 3. 生产部署前：替换 ops 内占位值为真实配置，再同步
./scripts/sync-to-build-machine.sh
./scripts/deploy.sh
```

自定义 ops 路径：

```bash
export ULTRASEND_OPS_DIR=/path/to/your-ops
./scripts/sync-to-build-machine.sh
```

## 目录结构

```
ops/
├── .ultrasend-ops              # marker（勿删）
├── cn/                         # 国内生产
│   ├── application-prod.yml
│   └── config.prod.bare.json
├── overseas/                   # 海外生产
│   ├── application-prod-overseas.yml
│   └── config.prod-overseas.bare.json
├── local/                      # 本地调试
│   ├── config.json
│   ├── application-dev-overseas.yml
│   ├── backend.env
│   └── docker.env
├── flutter/
│   ├── env.secrets.dart
│   ├── openpanel_env.secrets.dart
│   └── build.env
├── web/
│   └── .env.local
├── harmonyos/
│   └── build-profile.json5
└── scripts/                    # 兼容入口（请在业务仓 scripts/ 执行同步）
```

## 脚本位置（重要）

**同步命令请在业务仓根目录执行**，勿在 ops 仓内直接运行（ops/scripts/ 仅为旧路径兼容转发）：

| 用途 | 命令（在业务仓根目录） |
|------|------------------------|
| 同步本地配置 + 建库 | `./scripts/deploy-local.sh` |
| 同步生产配置 | `./scripts/sync-to-build-machine.sh` |
| 生产部署 | `./scripts/deploy.sh` |

详见业务仓 [ops/README.md](https://github.com/shrimpsend/shrimpsend/blob/main/ops/README.md) 与 [docs/SELF_HOST.md](https://github.com/shrimpsend/shrimpsend/blob/main/docs/SELF_HOST.md)。

## 自托管 checklist

1. Clone 本仓到 `../ops`，或复制目录结构到私有 git 仓库
2. 将所有 `CHANGE_ME` / `price_xxx` 替换为真实值
3. 轮换 JWT、Centrifugo、数据库、支付、对象存储等全部密钥
4. 在业务仓执行 `./scripts/sync-to-build-machine.sh` 后再 `./scripts/deploy.sh`
