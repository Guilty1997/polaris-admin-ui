# Polaris Admin UI

**Polaris** 管理后台前端，与后端 [polaris-admin](https://github.com/Guilty1997/polaris-admin) 配套使用。技术栈为 [Vue 3](https://cn.vuejs.org) + [TypeScript](https://www.typescriptlang.org) + [Element Plus](https://element-plus.org/zh-CN) + [Vite](https://cn.vitejs.dev)。

| 项目 | 地址 |
| --- | --- |
| 前端（本仓库） | [github.com/Guilty1997/polaris-admin-ui](https://github.com/Guilty1997/polaris-admin-ui) |
| 后端 | [github.com/Guilty1997/polaris-admin](https://github.com/Guilty1997/polaris-admin) |

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)

**版本**：与 `package.json` 中 `version` 一致（当前多为 `1.0.0-SNAPSHOT`）。  
**分支**：日常开发可使用 `dev`；稳定发布以仓库主分支为准。

---

## 环境要求

- Node.js（建议 LTS，如 20.x）
- npm / pnpm / yarn（示例命令以 npm 为主）

---

## 安装与运行

```bash
# 安装依赖（可使用国内镜像加速）
npm install --registry=https://registry.npmmirror.com

# 本地开发
npm run dev

# 生产构建
npm run build:prod

# 本地预览构建结果
npm run preview
```

开发环境下默认监听端口、接口前缀等由 **`.env.development`** 决定（例如 `VITE_APP_PORT`、`VITE_APP_BASE_API`）。请保证 Vite 代理指向的后端地址与 **polaris-admin** 实际启动地址一致；若修改了接口加密、RSA 密钥或 `clientId`，需与后端配置同步。

默认开发访问地址一般为 **`http://localhost:80`**（若80 端口被占用，可在环境文件中修改 `VITE_APP_PORT`）。

---

## 与后端的关系

本前端继承 **RuoYi-Vue-Plus / plus-ui** 体系的交互与权限模型，与 **Polaris** 后端 API 对接。业务菜单、权限标识需与后端菜单配置保持一致。

后端结构说明、本地启动数据库与 Redis 等步骤见 [polaris-admin README](https://github.com/Guilty1997/polaris-admin/blob/dev/README.md)。

---

## 其他说明

- **上游参考**：[RuoYi-Vue-Plus](https://github.com/dromara/RuoYi-Vue-Plus)、[plus-doc](https://plus-doc.dromara.org) 中仍有大量通用配置说明，排错时可对照。
- **协议**：MIT，使用或分发时请保留仓库中的许可证与版权声明（见 [LICENSE](./LICENSE)）。

---

## 脚本速查

| 命令 | 说明 |
| --- | --- |
| `npm run dev` | 开发模式（Vite） |
| `npm run build:prod` | 生产构建 |
| `npm run build:dev` | 开发环境构建 |
| `npm run lint:eslint` | ESLint 检查 |
| `npm run prettier` | Prettier 格式化 |
