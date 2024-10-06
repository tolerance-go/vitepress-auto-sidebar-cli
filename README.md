# vitepress-auto-sidebar-cli

vitepress-auto-sidebar-cli 是一个用于 VitePress 的命令行工具，用于自动生成侧边栏配置。

## 为什么开发这个工具？

在开发 VitePress 文档时，我发现侧边栏的配置与文件目录结构和 URL 之间存在着天然的映射关系。这种高度相关的结构让我意识到，应该有一个自动化工具来帮助从文档结构生成相应的配置。

现有的解决方案要么配置复杂，要么不支持实时监听。因此，我开发了 vitepress-auto-sidebar-cli —— 一个简单易用、支持监听模式、高效的自动生成侧边栏工具，旨在提高文档开发的效率。

## 特性

- 工具零配置，开箱即用
- 自动读取导航内容，扫描相关文档目录结构
- 生成外部 JSON 文件（`.vitepress/sidebar.json`），无侵入性集成
- 提供监听模式，结合高效的缓存机制，实时更新侧边栏配置

## 安装

使用 npm 安装：

```bash
npm install vitepress-auto-sidebar-cli --save-dev
```

或者使用 pnpm：

```bash
pnpm add vitepress-auto-sidebar-cli -D
```

## 使用

1. 在您的项目的 `package.json` 文件中添加以下脚本：

```json
{
  "scripts": {
    "generate-sidebar": "vitepress-auto-sidebar generate",
    "watch-sidebar": "vitepress-auto-sidebar watch"
  }
}
```

2. 运行命令生成侧边栏：

```bash
npm run generate-sidebar
```

或者启动监听模式，自动更新侧边栏：

```bash
npm run watch-sidebar
```

3. 在您的 VitePress 配置文件 (`.vitepress/config.ts` 或 `.vitepress/config.mts`) 中引入生成的侧边栏配置：

```javascript
import { defineConfig } from 'vitepress';
import sidebar from './.vitepress/sidebar.json';

export default defineConfig({
  // ...其他配置
  sidebar,
});
```

## 实现原理概要

vitepress-auto-sidebar-cli 通过以下步骤实现自动生成侧边栏：

1. 读取 VitePress 配置文件中的导航信息
2. 根据导航信息扫描指定的文档目录
3. 将所有扫描到的目录和文件信息加载到内存中，包括：
   - 文件路径
   - 文件内容
   - Markdown 文件的 frontmatter 信息
4. 根据内存中的信息构建侧边栏数据结构，遵循以下规则：
   - 对于文件：
     - 如果文件指定了侧边栏标题（通过 frontmatter 的 `sidebarTitle`），使用指定的标题
     - 如果没有指定 `sidebarTitle`，则使用文件内容中的第一个一级标题
     - 如果文件既没有指定 `sidebarTitle` 也没有一级标题，则使用文件名作为侧边栏标题
   - 对于目录：
     - 读取目录下 `index.md` 文件的 frontmatter
     - 使用 `index.md` 的 `sidebarTitle` 作为目录的侧边栏组标题
     - 如果 `index.md` 没有指定 `sidebarTitle`，则遵循与文件相同的规则（一级标题 > 文件名）
   - 对于 `index.md` 文件，将其链接添加到父级分组的侧边栏上，而不是作为单独的项目
   - 读取 frontmatter 中的 `sidebarOrder`，用于文件和目录的排序
5. 生成 `.vitepress/sidebar.json` 文件
6. 在监听模式下，使用缓存机制检测文件变化并高效地更新侧边栏配置：
   - 对于文件的删除和新增：
     - 找到变化相关的节点
     - 将其父节点及其往下所有的节点全部重新更新数据
   - 对于文件内容的修改：
     - 读取文件内容中的 frontmatter 相关信息
     - frontmatter 信息的更新依赖两个关键字段：
       - `sidebarTitle`：用于设置侧边栏标题
       - `sidebarOrder`：用于设置排序
     - 对于 `sidebarTitle`，其最终值是通过以下逻辑确定的：
       - 首先检查 frontmatter 中是否指定了 `sidebarTitle`
       - 如果没有指定，则查找文件内容中的第一个一级标题
       - 如果仍未找到，则使用文件名作为标题
     - 如果 `sidebarTitle` 或 `sidebarOrder` 有变化，则重新生成该文件相关的侧边栏数据
     - 如果这两个字段都没有变化，则跳过更新
   - 根据更新的数据重新生成 `.vitepress/sidebar.json` 文件

## 参与贡献

我们欢迎任何形式的贡献！如果您想为 vitepress-auto-sidebar-cli 做出贡献，请遵循以下步骤：

1. Fork 本仓库
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启一个 Pull Request

请确保您的代码符合我们的编码规范，并且包含适当的测试。

### 项目结构

项目的源代码位于 `src` 目录下，主要文件结构如下：

```
src/
├── main.ts                 # CLI 入口文件
├── config-reader.ts        # 读取 VitePress 配置文件
├── scanner.ts              # 扫描文档目录结构
├── parser.ts               # 解析 Markdown 文件、frontmatter 和维护内存数据
├── sidebar-builder.ts      # 构建侧边栏数据结构
├── json-generator.ts       # 生成 sidebar.json 文件
├── watcher.ts              # 文件监听和缓存机制
├── memory-store.ts         # 内存数据存储和管理
└── utils/
    ├── file-utils.ts       # 文件操作相关工具函数
    └── path-utils.ts       # 路径处理相关工具函数
```

- `main.ts`: CLI 的主入口文件，处理命令行参数并协调其他模块的工作。
- `config-reader.ts`: 负责读取 VitePress 的配置文件，获取导航信息。
- `scanner.ts`: 根据导航信息扫描指定的文档目录，收集文件和目录信息。
- `parser.ts`: 解析 Markdown 文件的内容和 frontmatter 信息，并将解析结果加载到内存中。
- `memory-store.ts`: 管理和维护内存中的文件信息、frontmatter 数据和其他相关信息。
- `sidebar-builder.ts`: 根据内存中的信息构建侧边栏数据结构。
- `json-generator.ts`: 将构建好的侧边栏数据结构生成为 JSON 文件。
- `watcher.ts`: 实现文件监听功能和缓存机制，用于监听模式下的实时更新。
- `utils/`: 包含一些通用的工具函数，如文件操作和路径处理。

这个结构设计遵循了单一职责原则，每个模块负责特定的功能，使得代码更易于维护和扩展。特别是，`parser.ts` 和新增的 `memory-store.ts` 共同负责解析文件内容、frontmatter 信息，并将这些信息维护在内存中，为其他模块提供快速访问和更新的能力。

### 项目工程化

本项目采用以下技术栈和工程化方案：

- 开发语言：TypeScript
- 构建工具：Rollup
- 输出格式：ES 模块 (ESM)
- 测试框架：Vitest
- 版本管理：date-based versioning
- Commit 规范：Conventional Commits
- 更新日志：自动根据 commit messages 生成

#### 主要 npm 脚本

```json
{
  "scripts": {
    "dev": "开发模式命令",
    "build": "构建打包命令",
    "publish": "发布命令",
    "test": "vitest run",
    "test:watch": "vitest",
    "changelog": "生成更新日志的命令"
  }
}
```

使用 TypeScript 开发可以提供更好的类型检查和开发体验。Rollup 用于将 TypeScript 代码打包成 ES 模块格式，这样可以确保良好的兼容性和性能。选择 ES 模块作为输出格式，可以更好地支持现代 JavaScript 生态系统，并允许用户使用 tree-shaking 等优化技术。

Vitest 作为测试框架，提供了快速、并行的测试执行能力，并与 Vite 生态系统良好集成。

我们采用基于日期的版本管理策略，这有助于清晰地反映发布时间。

项目遵循 Conventional Commits 规范来编写 commit messages，这有助于保持 commit 历史的一致性和可读性。基于这些规范化的 commit messages，我们可以自动生成更新日志，使项目的变更历史更加透明和易于理解。

## 许可证

本项目采用 MIT 许可证。详情请见 [LICENSE](LICENSE) 文件。
