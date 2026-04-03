# vae-skills

一个用于存储 Agent Skills 的仓库，适用于 Claude Code 和其他 AI Agent 终端。

## Skills

| Skill | 描述 | 安装 |
| --- | --- | --- |
| **code-style** | Vue3/Vite + Element Plus 代码风格规范。强制使用 defineAsyncComponent 异步导入组件、defineModel 双向绑定、行内单行注释 //、if 必须带花括号等 | `npx skills add vaebe/vae-skills` |

## 快速开始

安装 skill：

```bash
npx skills add vaebe/vae-skills
```

然后在 Agent 终端中使用：

```base
/code-style    # 应用代码风格规范
```

## 可用 Skills 详情

### code-style

**描述**: Vue3/Vite + Element Plus 代码风格规范。

**适用场景**:

- Vue3 后台管理项目代码生成
- 组件开发
- API 接口定义
- 代码重构

**核心规范**:

- 组件导入必须使用 `defineAsyncComponent`
- 双向绑定使用 `defineModel()`
- 接口字段注释使用行内单行注释 `//`
- `if` 语句必须使用花括号
- 模板引用使用 `useTemplateRef`

**触发关键词**: Vue3, script setup, Composition API, Element Plus, 异步组件, defineModel, 代码规范

## License

MIT
