# Aurora-UI 组件库 - 面试 Q&A 完整版

> 基于项目实际实现，针对 PDF 中"可能提问"部分的完整回答

---

## 📋 目录

1. [架构设计类问题](#架构设计类问题)
2. [组件实现类问题](#组件实现类问题)
3. [工程化类问题](#工程化类问题)
4. [测试类问题](#测试类问题)
5. [文档系统类问题](#文档系统类问题)
6. [技术选型类问题](#技术选型类问题)
7. [性能优化类问题](#性能优化类问题)
8. [团队协作类问题](#团队协作类问题)

---

## 架构设计类问题

### Q1: 为什么选择 Monorepo 架构来管理组件库？相比多仓库方式有哪些优劣？

**回答要点：**

> "我们选择 Monorepo 架构主要基于以下几个考虑：
> 
> **优势：**
> 1. **统一依赖管理**：使用 pnpm workspace 可以统一管理所有包的依赖版本，避免版本冲突
> 2. **代码复用**：packages 之间可以直接通过 workspace 协议引用，如 `@aurora-ui/hooks` 和 `@aurora-ui/utils` 被 components 包直接引用
> 3. **原子性提交**：组件库和文档站可以一起提交，保证版本一致性
> 4. **开发效率**：修改组件后，文档站可以立即使用最新代码，无需发布到 npm
> 5. **构建优化**：pnpm 的依赖提升机制，减少重复安装
> 
> **劣势：**
> 1. **仓库体积**：所有代码在一个仓库，体积较大
> 2. **权限管理**：需要更细粒度的权限控制
> 
> **我们的实现：**
> - 使用 `pnpm-workspace.yaml` 定义 workspace 范围
> - packages 之间通过 `workspace:*` 协议引用
> - 统一的构建脚本和代码规范"

**技术细节（如果追问）：**

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

```json
// packages/components/package.json
{
  "dependencies": {
    "@aurora-ui/hooks": "workspace:*",
    "@aurora-ui/utils": "workspace:*"
  }
}
```

---

### Q2: 你是如何实现组件的按需引入和全量引入的？用了哪些工具或技术？
<font color='#d9534f' >我们通过 Vite + Rollup 的配置实现了按需引入和全量引入。全量引入时，直接导入组件库入口文件即可使用。按需引入时，利用 Rollup 的 ESM 静态分析和 Tree Shaking，只打包项目实际用到的组件，从而减小 bundle 体积。</font>
**回答要点：**

> "我们通过 Vite + Rollup 的配置实现了按需引入和全量引入：
> 
> **1. 构建配置：**
> - 使用 Vite 的 library 模式，配置了 ESM 和 CJS 两种格式
> - 通过 `rollupOptions.external` 将 React 等依赖外部化，避免重复打包
> 
> **2. package.json exports 字段：**
> ```json
> "exports": {
>   ".": {
>     "import": "./dist/index.esm.js",  // ESM 格式，支持 tree-shaking
>     "require": "./dist/index.js",     // CJS 格式
>     "types": "./dist/index.d.ts"
>   },
>   "./style": "./dist/style.css"
> }
> ```
> 
> **3. 按需引入：**
> ```tsx
> // 支持 tree-shaking，只打包使用的组件
> import { Button } from '@aurora-ui/components';
> ```
> 
> **4. 全量引入：**
> ```tsx
> // 引入所有组件
> import * as AuroraUI from '@aurora-ui/components';
> ```
> 
> **5. 样式引入：**
> ```tsx
> import '@aurora-ui/components/style'; // 全量样式
> // 或者按需引入（需要配置 CSS Modules 的单独导出）"

**技术细节：**

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.ts'),
      formats: ['es', 'cjs'], // 同时输出 ESM 和 CJS
    },
    rollupOptions: {
      external: ['react', 'react-dom'], // 外部化依赖
    },
  },
});
```

---

### Q3: 组件库是如何分类和组织的？你怎么保证结构清晰且便于扩展？

**回答要点：**

> "我们按照功能将组件分为三类：
> 
> **1. 基础通用组件**（packages/components/src/）：
> - Button：按钮组件
> - Input：输入框组件（包含 TextArea、Password 子组件）
> - Upload：文件上传组件
> 
> **2. 工具包**（packages/）：
> - `hooks/`：共享的 React Hooks，如 `useUpload`
> - `utils/`：工具函数，如 `classNames`
> 
> **3. 文档站点**（apps/docs/）：
> - 自定义文档系统，展示组件使用示例
> 
> **组织结构：**
> ```
> packages/components/src/
>   ├── Button/
>   │   ├── Button.tsx          # 组件实现
>   │   ├── Button.module.scss  # 样式文件
>   │   ├── Button.test.tsx     # 测试文件
>   │   └── index.ts            # 导出文件
>   ├── Input/
>   │   ├── Input.tsx
>   │   ├── Input.module.scss
>   │   └── index.ts
>   └── index.ts                # 统一导出
> ```
> 
> **扩展性保证：**
> 1. **统一的组件结构**：每个组件都有独立的文件夹，包含实现、样式、测试、导出
> 2. **类型定义**：每个组件都导出 TypeScript 类型，便于类型检查
> 3. **统一导出**：通过 `src/index.ts` 统一导出，外部只需一个入口
> 4. **样式隔离**：使用 SCSS Modules，避免样式冲突"

---

## 组件实现类问题

### Q4: 你在实现比如 Upload 或 Input 这类复杂组件时，遇到过哪些难点？是怎么解决的？

**回答要点：**
![Pasted image 20260122200524](../assets/images/Pasted%20image%2020260122200524.png)

> "**Upload 组件的难点：**
> 
> **1. 文件状态管理：**
> - 难点：需要管理文件列表、上传进度、状态变化
> - 解决：封装了 `useUpload` Hook，统一管理文件状态
> ```tsx
> const { fileList, upload, remove } = useUpload({
>   onSuccess: (file) => onUpload?.(file),
>   onRemove: (file) => onRemove?.(file),
> });
> ```
> 
> **2. 拖拽上传：**
> - 难点：需要处理 dragenter、dragover、drop 事件，防止默认行为
> - 解决：使用 React 事件处理，设置 `preventDefault()` 和状态管理
> ```tsx
> const handleDragOver = (e: React.DragEvent) => {
>   e.preventDefault();
>   setIsDragging(true);
> };
> ```
> 
> **3. 进度模拟：**
> - 难点：需要模拟上传进度（因为 POC 阶段没有真实后端）
> - 解决：使用 `setInterval` 模拟进度更新，每 200ms 增加 10%
> 
> **Input 组件的难点：**
> 
> **1. 复合组件模式：**
> - 难点：需要实现 `Input.TextArea` 和 `Input.Password` 子组件
> - 解决：使用 TypeScript 的类型断言和属性赋值
> ```tsx
> export const Input = InputComponent as typeof InputComponent & {
>   TextArea: typeof TextArea;
>   Password: typeof Password;
> };
> Input.TextArea = TextArea;
> Input.Password = Password;
> ```
> 
> **2. 受控/非受控组件：**
> - 难点：需要同时支持受控和非受控模式
> - 解决：通过判断 `value` prop 是否存在来判断模式
> ```tsx
> const isControlled = value !== undefined;
> const currentValue = isControlled ? String(value || '') : internalValue;
> ```
> 
> **3. forwardRef 传递：**
> - 难点：需要将 ref 正确传递到底层 DOM 元素
> - 解决：使用 `React.forwardRef` 包装组件
> ```tsx
> export const Input = React.forwardRef<HTMLInputElement, InputProps>(
>   ({ ...props }, ref) => {
>     return <input ref={ref} {...props} />;
>   }
> );
> ```"

---

### Q5: 组件的 props 类型定义是如何做的？有没有用到泛型或联合类型？

**回答要点：**

> "我们使用 TypeScript 严格定义组件 props，充分利用了类型系统的特性：
> 
> **1. 基础类型定义：**
> ```tsx
> export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
>   variant?: 'primary' | 'default' | 'text';  // 联合类型
>   size?: 'small' | 'medium' | 'large';
> }
> ```
> 
> **2. 继承原生属性：**
> - 使用 `extends` 继承原生 HTML 元素属性，减少重复定义
> - 使用 `Omit` 排除冲突的属性
> ```tsx
> export interface InputProps extends Omit<React.InputHTMLAttributes<HTMLInputElement>, 'size'> {
>   size?: 'small' | 'medium' | 'large';  // 覆盖原生的 size 属性
> }
> ```
> 
> **3. 泛型使用（forwardRef）：**
> `forwardRef` 是一个 **高阶函数**，用来将 ref 转发给子组件（通常是 DOM 元素或 class 组件）
> React.forwardRef<HTMLButtonElement, ButtonProps>
	HTMLButtonElement → ref 的类型
    ButtonProps → 组件 props 的类型
> ```tsx
> export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
>   ({ ...props }, ref) => { ... }
> );
> ```
> 
> **4. 联合类型应用：**
> - 用于限制 props 的可选值
> - 提供更好的 IDE 自动补全和类型检查
> 
> **5. 类型导出：**
> ```tsx
> export type { ButtonProps } from './Button';
> ```
> 方便外部使用时进行类型约束"

---

### Q6: 如何保证组件在多种场景下的可复用性和扩展性？有没有做 slots、render props 或 context 的封装？

**回答要点：**

> "我们通过多种模式保证组件的可复用性和扩展性：
> 组件模式本质是 **组合子组件来形成更复杂的功能**
> - `Input` 是一个主组件
- `TextArea` 和 `Password` 是 `Input` 的“子组件”
> **1. 组合模式（Composition）：**
> - Input 组件使用复合组件模式，提供 `Input.TextArea` 和 `Input.Password`
> - 用户可以根据需求选择合适的子组件
> 
> **2. 灵活的 Props 设计：**
> - 继承原生 HTML 属性，支持所有标准属性
> - 通过 `...props` 传递额外属性
> ```tsx
> <button {...props} className={classNames(...)}>
> ```
> 
> **3. 回调函数支持：**
> - Upload 组件提供 `onUpload`、`onRemove` 回调
> - 允许外部自定义上传逻辑和错误处理
> 
> **4. className 支持：**
> - 所有组件都支持 `className` prop，允许外部自定义样式
> - 使用 `classNames` 工具函数合并类名
> 
> **5. forwardRef 支持：**
> - 所有组件都使用 `forwardRef`，允许外部获取 DOM 引用
> - 便于集成到表单库或其他需要直接操作 DOM 的场景
> 
> **未来扩展方向：**
> - 可以考虑添加 render props 模式，如 `Upload` 组件支持自定义上传区域渲染
> - 可以使用 Context 提供全局配置，如主题、国际化等"

---

## 工程化类问题

### Q7: ESLint、Prettier、Husky、lint-staged 是如何集成在 CI 流程中的？这些工具分别解决了哪些问题？

**回答要点：**

> "我们通过 Git Hooks 和 lint-staged 实现了提交前的自动校验：
> 
> **1. ESLint：代码质量检查**
> - 配置：`.eslintrc.cjs`
> - 作用：检查代码规范、潜在错误、React Hooks 规则等
> - 插件：`@typescript-eslint`、`eslint-plugin-react`、`eslint-plugin-react-hooks`
> 
> **2. Prettier：代码格式化**
> - 配置：`.prettierrc.json`
> - 作用：统一代码风格（缩进、引号、分号等）
> - 与 ESLint 集成：使用 `eslint-config-prettier` 避免冲突
> 
> **3. Husky：Git Hooks 管理**
> - 配置：`.husky/pre-commit`
> - 作用：在提交前自动运行校验脚本
> - 安装：通过 `pnpm prepare` 脚本自动安装
> 
> **4. lint-staged：只检查暂存文件**
> - 配置：`package.json` 中的 `lint-staged` 字段
> - 作用：只对暂存区的文件进行检查，提高效率 （**暂存区** = git add 后加入的文件）
> ```json
> "lint-staged": {
>   "*.{ts,tsx,js,jsx}": ["eslint --fix"],
>   "*.{css,scss,md,json}": ["prettier --write"]
> }
> ```
> 
> **工作流程：**
> 1. 开发者执行 `git commit`
> 2. Husky 触发 `pre-commit` hook
> 3. lint-staged 只检查暂存区的文件
> 4. ESLint 自动修复可修复的问题
> 5. Prettier 格式化代码
> 6. 如果检查通过，提交成功；否则阻止提交
> 
> **优势：**
> - 保证代码质量，避免不规范代码进入仓库
> - 统一团队代码风格
> - 减少 Code Review 的工作量"

**配置文件示例：**

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged
```

---

### Q8: 使用 pnpm workspace 时，是否遇到过依赖冲突或包引用异常？是怎么处理的？

**回答要点：**

> "**遇到的常见问题：**
> 
> **1. 依赖版本冲突：**
> - 问题：不同包需要不同版本的同一依赖
> - 解决：pnpm 的依赖解析机制会自动处理，通过 `node_modules/.pnpm` 的符号链接实现
> 
> **2. workspace 协议引用：**
> - 问题：本地包引用需要使用 `workspace:*` 协议
> - 解决：确保 package.json 中使用正确的协议
> ```json
> {
>   "dependencies": {
>     "@aurora-ui/hooks": "workspace:*"
>   }
> }
> ```
> 
> **3. 依赖提升问题：**
> - 问题：某些依赖需要提升到根目录
> - 解决：使用 `.npmrc` 配置或 pnpm 的 `shamefully-hoist` 选项（一般不推荐）
> 
> **4. 构建顺序问题：**
> - 问题：components 包依赖 hooks 和 utils，需要先构建依赖包
> - 解决：使用 `pnpm -r build` 按依赖顺序构建，或使用 `--filter` 指定构建顺序
> 
> **最佳实践：**
> - 统一依赖版本：在根目录的 package.json 中定义共享依赖
> - 使用 peerDependencies：对于 React 等框架依赖，使用 peerDependencies
> - 定期清理：使用 `pnpm store prune` 清理未使用的包"

---

## 测试类问题

### Q9: Jest 单元测试中是如何设计用例的？主要测试哪些场景？有没有遇到过模拟异步交互的问题？

**回答要点：**

> "我们使用 Jest + React Testing Library 进行单元测试：
> 
> **1. 测试配置：**
> - `jest.config.cjs`：配置测试环境、模块解析、覆盖率等
> - `setupTests.ts`：引入 `@testing-library/jest-dom`，扩展匹配器
> 
> **2. 测试场景设计：**
> 
> **基础逻辑测试：**
> - 组件能否正常渲染
> - Props 是否正确传递
> - 默认值是否正确
> ```tsx
> it('renders with default props', () => {
>   render(<Button>Click me</Button>);
>   expect(screen.getByRole('button')).toBeInTheDocument();
> });
> ```
> 
> **交互测试：**
> - 点击事件是否正确触发
> - 用户输入是否正确处理
> ```tsx
> it('calls onClick when clicked', async () => {
>   const handleClick = jest.fn();
>   render(<Button onClick={handleClick}>Click</Button>);
>   await userEvent.click(screen.getByRole('button'));
>   expect(handleClick).toHaveBeenCalledTimes(1);
> });
> ```
> 
> **边界情况测试：**
> - 空值处理
> - 禁用状态
> - 极端输入
> 
> **3. 异步交互模拟：**
> 
> **问题：** Upload 组件的进度更新是异步的
> 
> **解决：**
> - 使用 `waitFor` 等待异步更新
> - 使用 `jest.useFakeTimers()` 控制定时器
> ```tsx
> it('shows upload progress', async () => {
>   jest.useFakeTimers();
>   render(<Upload />);
>   // 模拟文件上传
>   // 使用 act() 和 waitFor 等待状态更新
>   jest.useRealTimers();
> });
> ```
> 
> **4. 测试覆盖率：**
> - 目标：核心逻辑和交互行为达到 80%+ 覆盖率
> - 命令：`pnpm test:coverage`
> - 重点关注：用户交互、边界情况、错误处理"

**测试文件示例：**

```tsx
// Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Test</Button>);
    expect(screen.getByRole('button', { name: /test/i })).toBeInTheDocument();
  });

  it('handles click events', async () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    await userEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

---

### Q10: 组件的样式是怎么写的？用了哪种 CSS 方案（如 CSS Modules、Tailwind、cssinjs）？为什么选它？
<font color='#d9534f' >“我们组件库的样式使用了 **SCSS Modules**。这样每个组件的 className 都是局部作用域，不会污染全局，同时 SCSS 提供了嵌套、变量和 mixin，让复杂样式更易维护和复用。</font>

**回答要点：**

> "我们选择了 **SCSS Modules** 作为样式方案：
> 
> **1. 为什么选择 SCSS Modules：**
> 
> **样式隔离：**
> - 每个组件的样式都是独立的，不会相互影响
> - 类名会被编译成唯一的哈希值，如 `.button_a1b2c3`
> 
> **开发体验：**
> - 支持 SCSS 的嵌套、变量、混入等特性
> - TypeScript 支持：可以导入样式对象并获得类型提示
> ```tsx
> import styles from './Button.module.scss';
> <button className={styles.button} />
> ```
> 
> **性能优势：**
> - 编译时处理，运行时无额外开销
> - 支持 CSS 代码分割，按需加载
> 
> **2. 实现方式：**
> ```tsx
> // Button.tsx
> import styles from './Button.module.scss';
> 
> <button className={classNames(styles.button, styles[`button--${variant}`])}>
> ```
> 
> **3. 样式组织：**
> - 每个组件有独立的 `.module.scss` 文件
> - 使用 BEM 命名规范：`.button--primary`、`.button--small`
> - 统一的变量和混入（可以提取到共享文件）
> 
> **4. 与其他方案对比：**
> - **vs CSS-in-JS**：编译时处理，性能更好，但动态主题支持较弱
> - **vs Tailwind**：更适合组件库，样式更可控
> - **vs 普通 CSS**：提供了作用域隔离，避免样式冲突
> 
> **5. 构建配置：**
> - Vite 原生支持 SCSS Modules
> - 无需额外配置，开箱即用"

---

## 文档系统类问题

### Q11: 如何在 Storybook 中实现组件的交互演示和属性说明？有没有做二次封装或自定义插件？
<font color='#d9534f' >“在 Storybook 中，我们用 **Args 和 Controls** 实现组件交互演示，团队可以在控制面板修改 props 并实时看到效果。同时结合 Storybook Docs + TypeScript 类型 自动生成属性说明，让每个组件的类型、默认值和说明都一目了然。</font>


**回答要点：**

> "我们项目采用了**两种文档方案**：
> 
> **方案一：自定义文档系统（当前实现）**
> 
> **1. ComponentPreview 组件封装：**
> ```tsx
> <ComponentPreview
>   title="Basic Input"
>   description="A basic input field"
>   sourceCode={codeString}
> >
>   <Input placeholder="Enter text" />
> </ComponentPreview>
> ```
> 
> **2. CodeBlock 组件：**
> - 使用 `highlight.js` 实现代码高亮
> - 支持多种语言（TSX、JSX、CSS 等）
> - 自动检测语言类型
> 
> **3. 实时预览：**
> - 组件直接渲染，支持完整交互
> - 源码实时展示，便于学习
> 
> **方案二：Storybook（已配置，可选）**
> 通过 argTypes 配置每个属性的说明、控件和文档
> 
> **1. Story 编写：**
> ```tsx
> export const Primary: Story = {
>   args: {
>     variant: 'primary',
>     children: 'Primary Button',
>   },
> };
> ```
> 
> **2. 属性说明（argTypes）：**
> ```tsx
> argTypes: {
>   variant: {
>     control: 'select',
>     options: ['primary', 'default', 'text'],
>     description: '按钮的视觉样式变体',
>     table: {
>       type: { summary: 'primary | default | text' },
>       defaultValue: { summary: 'default' },
>     },
>   },
> }
> ```
> 
> **3. 交互演示（play 函数）：**
> ```tsx
> export const InteractiveExample: Story = {
>   play: async ({ canvasElement }) => {
>     const canvas = within(canvasElement);
>     const button = canvas.getByRole('button');
>     await userEvent.click(button);
>     await expect(button).toHaveTextContent('Clicked!');
>   },
> };
> ```
> 
> **4. 自定义插件（可选）：**
> - 可以开发自定义 Addon 扩展功能
> - 如设计 token 面板、组件使用统计等
> 
> **选择理由：**
> - 自定义方案：轻量、完全控制、与现有构建流程集成
> - Storybook：功能丰富、社区支持、适合大型项目"

---

### Q12: Preview 插件的设计原理是什么？它是怎么做到实时预览和代码高亮的？
<font color='#d9534f' >Storybook Preview 的原理是通过 **iframe ** 隔离组件渲染，结合 **Args 和 Controls** 实时驱动组件状态。当我们修改 props 时，组件会自动重新渲染。底层依赖  Vite 的 HMR** 实现热更新，无需刷新页面。代码高亮方面，Docs 会抓取组件源码，使用 react-syntax-highlighter 做语法解析和高亮渲染，使预览和源码保持一致</font>

**回答要点：**

> "**ComponentPreview 组件设计原理：**
> 
> **1. 组件结构：**
> ```tsx
> <div className="component-preview">
>   {/* 标题和描述 */}
>   {title && <h3>{title}</h3>}
>   {description && <p>{description}</p>}
>   
>   {/* 组件渲染区域 */}
>   <div className="component-preview-demo">
>     {children}  {/* 直接渲染传入的组件 */}
>   </div>
>   
>   {/* 代码展示区域 */}
>   <CodeBlock code={sourceCode} />
> </div>
> ```
> 
> **2. 实时预览原理：**
> - 使用 React 的 `children` prop 直接渲染组件
> - 组件是**真实渲染**的，不是截图或 iframe
> - 支持完整的用户交互（点击、输入、拖拽等）
> - 通过状态管理实现响应式更新
> 
> **3. 代码高亮原理：**
> 
> **CodeBlock 组件实现：**
> ```tsx
> import hljs from 'highlight.js';
> 
> useEffect(() => {
>   if (codeRef.current) {
>     hljs.highlightElement(codeRef.current);  // 高亮代码块
>   }
> }, [code]);
> ```
> 
> **工作流程：**
> 1. 接收 `sourceCode` 字符串
> 2. 渲染到 `<code>` 元素中
> 3. 使用 `highlight.js` 的 `highlightElement` 方法
> 4. highlight.js 自动检测语言类型（通过 `language-tsx` 类名）
> 5. 应用语法高亮样式
> 
> **4. 样式定制：**
> - 使用 `highlight.js/styles/github-dark.css` 主题
> - 可以通过 CSS 覆盖自定义样式
> - 支持代码块滚动、复制等功能（可扩展）
> 
> **5. 优势：**
> - **实时性**：组件和代码同步更新
> - **交互性**：支持完整的用户交互
> - **可读性**：语法高亮提升代码可读性
> - **可维护性**：组件化设计，易于扩展"

---

### Q13: highlight.js 是怎么集成到文档展示中的？你有没有对高亮主题或样式做过定制？

<font color='#d9534f' >highlight.js 是一个前端语法高亮库，我们在文档中通过引入库，然后在渲染 `<pre><code>` 的代码块时调用 `highlightElement` 来自动高亮。主题方面，我们可以直接用自带的主题，也可以基于 CSS 修改关键字颜色、背景或字体，保证文档风格和产品 UI 保持一致。在我们的项目中，我曾定制过深色主题，使代码在暗模式下显示效果更好</font>

**回答要点：**

> "**集成方式：**
> 
> **1. 安装依赖：**
> ```bash
> pnpm add highlight.js
> pnpm add -D @types/highlight.js
> ```
> 
> **2. 引入样式：**
> ```tsx
> import hljs from 'highlight.js';
> import 'highlight.js/styles/github-dark.css';  // 选择主题
> ```
> 
> **3. 使用方式：**
> ```tsx
> const CodeBlock: React.FC<CodeBlockProps> = ({ code, language = 'tsx' }) => {
>   const codeRef = useRef<HTMLElement>(null);
> 
>   useEffect(() => {
>     if (codeRef.current) {
>       hljs.highlightElement(codeRef.current);  // 高亮处理
>     }
>   }, [code]);
> 
>   return (
>     <div className="code-block">
>       <pre>
>         <code ref={codeRef} className={`language-${language}`}>
>           {code}
>         </code>
>       </pre>
>     </div>
>   );
> };
> ```
> 
> **4. 样式定制：**
> 
> **主题选择：**
> - 默认使用 `github-dark` 主题（深色背景）
> - 可以替换为其他主题：`github`、`atom-one-dark`、`vs2015` 等
> 
> **自定义样式：**
> ```css
> /* CodeBlock.css */
> .code-block {
>   border-radius: 4px;
>   overflow: hidden;
>   border: 1px solid #e8e8e8;
> }
> 
> .code-block pre {
>   margin: 0;
>   padding: 16px;
>   background-color: #0d1117;
>   overflow-x: auto;
> }
> 
> .code-block code {
>   font-family: 'Fira Code', 'Consolas', monospace;
>   font-size: 13px;
>   line-height: 1.6;
> }
> ```
> 
> **5. 语言检测：**
> - 通过 `className={`language-${language}`}` 指定语言
> - highlight.js 自动识别并应用对应的高亮规则
> - 支持 TSX、JSX、CSS、SCSS、JSON 等
> 
> **6. 性能优化：**
> - 使用 `useEffect` 只在代码变化时重新高亮
> - 使用 `useRef` 避免重复创建 DOM 引用"

---

## 技术选型类问题

### Q14: 组件库是否做了版本管理和发布？是怎么发布到 npm 的？如何处理依赖更新问题？

**回答要点：**

> "**当前状态：**
> - 项目目前是 `private: true`，主要用于学习和演示
> - 已配置好发布所需的 package.json 字段
> 
> **发布准备：**
> 
> **1. package.json 配置：**
> ```json
> {
>   "name": "@aurora-ui/components",
>   "version": "1.0.0",
>   "main": "./dist/index.js",
>   "module": "./dist/index.esm.js",
>   "types": "./dist/index.d.ts",
>   "files": ["dist"],  // 只发布 dist 目录
>   "exports": {
>     ".": {
>       "import": "./dist/index.esm.js",
>       "require": "./dist/index.js",
>       "types": "./dist/index.d.ts"
>     }
>   }
> }
> ```
> 
> **2. 版本管理策略：**
> - 使用语义化版本（Semantic Versioning）
> - `1.0.0` → `1.0.1`（补丁版本：bug 修复）
> - `1.0.0` → `1.1.0`（次要版本：新功能，向后兼容）
> - `1.0.0` → `2.0.0`（主要版本：破坏性变更）
> 
> **3. 发布流程：**
> ```bash
> # 1. 构建组件库
> pnpm build:components
> 
> # 2. 更新版本号
> npm version patch|minor|major
> 
> # 3. 发布到 npm
> npm publish --access public
> ```
> 
> **4. 依赖更新处理：**
> 
> **peerDependencies：**
> - React、React-DOM 使用 peerDependencies
> - 避免版本冲突，由使用者提供
> 
> **workspace 依赖：**
> - `@aurora-ui/hooks` 和 `@aurora-ui/utils` 需要一起发布
> - 或者打包时将这些依赖内联（bundle）
> 
> **5. 自动化发布（CI/CD）：**
> - 可以使用 GitHub Actions 自动化发布流程
> - 在 push tag 时自动构建和发布
> 
> **6. 变更日志（CHANGELOG）：**
> - 维护 CHANGELOG.md 记录版本变更
> - 使用 `conventional-changelog` 自动生成"

---

### Q15: 你有没有设计"主题化"或"国际化"功能？如果需要支持换肤或多语言，该怎么扩展组件架构？

**回答要点：**
### `var()` 函数

`var(变量名, fallback)`

含义是：

- 如果变量存在 → 用变量值
    
- 如果变量不存在 → 用 fallback（兜底值）
  DOM 操作：设置 data-theme 属性

  document.documentElement.setAttribute('data-theme', theme);

> "**当前实现：**
> - 项目目前使用 SCSS Modules，样式是静态的
> - 未实现主题化和国际化功能
> 
> **主题化扩展方案：**
> 
> **方案一：CSS 变量（推荐）**
> ```scss
> // 定义 CSS 变量
> :root {
>   --color-primary: #1890ff;
>   --color-text: rgba(0, 0, 0, 0.85);
>   --spacing-sm: 4px;
> }
> 
> // 组件中使用
> .button {
>   background-color: var(--color-primary);
>   color: var(--color-text);
>   padding: var(--spacing-sm);
> }
> ```
> 
> **方案二：Context + CSS-in-JS**
> ```tsx
> // ThemeContext.tsx
> const ThemeContext = createContext<Theme>(defaultTheme);
> 
> // 组件中使用
> const theme = useContext(ThemeContext);
> <button style={{ backgroundColor: theme.primaryColor }}>
> ```
> 
> **方案三：SCSS 变量 + 多主题文件**
> ```scss
> // themes/default.scss
> $primary-color: #1890ff;
> 
> // themes/dark.scss
> $primary-color: #177ddc;
> ```
> 
> **国际化扩展方案：**
> 
> **1. 使用 Context：**
> ```tsx
> const I18nContext = createContext<Locale>('zh-CN');
> 
> // 组件中使用
> const locale = useContext(I18nContext);
> <button>{locale.button.confirm}</button>
> ```
> 
> **2. 配置文件：**
> ```tsx
> // locales/zh-CN.ts
> export default {
>   button: {
>     confirm: '确认',
>     cancel: '取消',
>   },
> };
> ```
> 
> **3. 组件扩展：**
> - 为需要国际化的组件添加 `locale` prop
> - 提供默认的国际化文本
> - 允许外部覆盖
> 
> **架构设计：**
> ```
> packages/
>   ├── components/     # 组件库
>   ├── theme/          # 主题系统（新增）
>   │   ├── tokens/     # 设计令牌
>   │   └── themes/     # 主题配置
>   └── i18n/           # 国际化（新增）
>       └── locales/    # 语言包
> ```
> 
> **实现优先级：**
> 1. 先实现 CSS 变量方案（最简单）
> 2. 再添加 Context 支持（更灵活）
> 3. 最后考虑完整的主题系统（最复杂）"

---

## 性能优化类问题

### Q16: 你是否做过组件性能优化，比如渲染优化、虚拟滚动、懒加载等？

**回答要点：**

> "**当前实现的优化：**
> 
> **1. forwardRef 优化：**
> - 所有组件都使用 `forwardRef`，避免不必要的包装组件
> - 减少组件层级，提升渲染性能
> 
> **2. SCSS Modules：**
> - 编译时处理，运行时无额外开销
> - 样式隔离，避免全局样式污染
> 
> **3. 按需引入：**
> - 通过 tree-shaking 只打包使用的组件
> - 减少最终打包体积
> 
> **未来优化方向：**
> 
> **1. React.memo：**
> ```tsx
> export const Button = React.memo(
>   React.forwardRef<HTMLButtonElement, ButtonProps>((props, ref) => {
>     // ...
>   })
> );
> ```
> - 避免不必要的重渲染
> - 适用于 props 变化频率低的组件
> 
> **2. useMemo / useCallback：**
> ```tsx
> const handleClick = useCallback((e: React.MouseEvent) => {
>   onClick?.(e);
> }, [onClick]);
> ```
> - 缓存回调函数，避免子组件重渲染
> 
> **3. 虚拟滚动（Table 组件）：**
> - 只渲染可见区域的列表项
> - 适用于大数据量场景
> - 可以使用 `react-window` 或 `react-virtualized`
> 
> **4. 懒加载：**
> ```tsx
> const Upload = React.lazy(() => import('./Upload'));
> ```
> - 按需加载组件，减少初始包体积
> 
> **5. 代码分割：**
> - 每个组件独立打包
> - 支持动态导入
> 
> **6. 样式优化：**
> - 提取公共样式，减少重复
> - 使用 CSS 变量，减少样式计算
> 
> **性能监控：**
> - 使用 React DevTools Profiler 分析性能
> - 关注组件渲染次数和耗时"

---

### Q17: 你会如何衡量一个组件是否"设计得好"？有哪些判断标准或经验？

**回答要点：**

> "**设计好的组件的标准：**
> 
> **1. API 设计：**
> - **直观性**：Props 命名清晰，符合直觉
> - **一致性**：相似的组件有相似的 API
> - **灵活性**：支持多种使用场景
> - **可扩展性**：易于添加新功能
> 
> **2. 类型安全：**
> - 完整的 TypeScript 类型定义
> - 良好的类型推断
> - 避免 any 类型
> 
> **3. 可访问性（A11y）：**
> - 支持键盘导航
> - ARIA 属性正确
> - 语义化 HTML
> - 屏幕阅读器友好
> 
> **4. 性能：**
> - 渲染性能良好
> - 内存占用合理
> - 支持按需加载
> 
> **5. 可测试性：**
> - 易于编写单元测试
> - 测试覆盖率达标
> - 边界情况处理完善
> 
> **6. 文档完善：**
> - 清晰的 API 文档
> - 丰富的使用示例
> - 常见问题解答
> 
> **7. 用户体验：**
> - 交互流畅
> - 错误处理友好
> - 加载状态明确
> - 反馈及时
> 
> **8. 代码质量：**
> - 代码结构清晰
> - 注释适当
> - 遵循最佳实践
> - 易于维护
> 
> **实际案例（Button 组件）：**
> - ✅ API 直观：`variant`、`size` 命名清晰
> - ✅ 类型安全：完整的 TypeScript 类型
> - ✅ 灵活性：支持所有原生 button 属性
> - ✅ 可访问性：使用语义化 `<button>` 标签
> - ✅ 性能：使用 forwardRef，减少包装
> - ✅ 可测试：易于编写测试用例
> 
> **持续改进：**
> - 收集用户反馈
> - 分析使用数据
> - 定期重构优化
> - 跟进 React 最佳实践"

---

## 团队协作类问题

### Q18: 项目开发过程中，有没有团队协作？你是如何制定组件规范和开发流程的？

**回答要点：**

> "**项目背景：**
> - 这是一个学习项目，主要用于展示组件库开发能力
> - 但设计时考虑了团队协作的场景
> 
> **组件规范：**
> 
> **1. 代码规范：**
> - ESLint + Prettier 统一代码风格
> - TypeScript 严格模式
> - 统一的文件命名（PascalCase 组件，camelCase 工具函数）
> 
> **2. 组件结构规范：**
> ```
> ComponentName/
>   ├── ComponentName.tsx      # 组件实现
>   ├── ComponentName.module.scss  # 样式文件
>   ├── ComponentName.test.tsx    # 测试文件
>   └── index.ts                 # 导出文件
> ```
> 
> **3. 命名规范：**
> - 组件名：PascalCase（Button、Input）
> - Props 接口：`ComponentNameProps`
> - 样式类：BEM 规范（`.button--primary`）
> 
> **4. 提交规范：**
> - 使用 Conventional Commits
> - `feat:` 新功能
> - `fix:` Bug 修复
> - `docs:` 文档更新
> - `style:` 代码格式
> - `refactor:` 重构
> 
> **开发流程：**
> 
> **1. 组件开发流程：**
> 1. 需求分析 → 2. API 设计 → 3. 实现组件 → 4. 编写测试 → 5. 编写文档 → 6. Code Review → 7. 合并
> 
> **2. 代码审查：**
> - 通过 Git Hooks 自动检查
> - 人工 Code Review（如果团队协作）
> - 关注：功能正确性、代码质量、性能、可访问性
> 
> **3. 测试要求：**
> - 新组件必须包含测试用例
> - 覆盖率要求：核心逻辑 80%+
> - 必须通过所有测试才能合并
> 
> **4. 文档要求：**
> - 每个组件必须有使用示例
> - API 文档完整
> - 包含常见问题
> 
> **5. 版本管理：**
> - 使用语义化版本
> - 维护 CHANGELOG
> - 重大变更需要迁移指南
> 
> **团队协作工具：**
> - Git：版本控制
> - GitHub/GitLab：代码托管和协作
> - Issues：需求跟踪
> - Pull Requests：代码审查
> - Discussions：技术讨论"

---

### Q19: 你怎么看目前市面上的开源组件库（如 Ant Design、MUI）？你们组件库的定位和目标是什么？

**回答要点：**

> "**对主流组件库的看法：**
> 
> **Ant Design：**
> - **优势**：组件丰富、文档完善、企业级应用广泛
> - **特点**：设计语言统一、TypeScript 支持好
> - **适用场景**：中后台系统、企业应用
> 
> **Material-UI (MUI)：**
> - **优势**：遵循 Material Design，组件质量高
> - **特点**：主题系统强大、可定制性强
> - **适用场景**：移动端、Material Design 风格应用
> 
> **Chakra UI：**
> - **优势**：API 简洁、开发体验好
> - **特点**：组件化设计、样式系统灵活
> - **适用场景**：快速开发、现代 Web 应用
> 
> **Aurora-UI 的定位：**
> 
> **1. 学习导向：**
> - 主要用于学习和展示组件库开发能力
> - 理解组件库的设计和实现原理
> 
> **2. 轻量级：**
> - 只实现核心组件，避免过度设计
> - 保持代码简洁，易于理解和维护
> 
> **3. 现代化：**
> - 使用最新的 React 18 + TypeScript
> - 遵循 React 最佳实践
> - 支持现代构建工具（Vite）
> 
> **4. 可扩展：**
> - 架构设计支持扩展
> - 易于添加新组件
> - 支持自定义主题和样式
> 
> **5. 工程化：**
> - 完整的开发工具链
> - 测试和文档体系
> - 代码质量保证
> 
> **与主流组件库的区别：**
> - **规模**：更小，聚焦核心组件
> - **目标**：学习 > 生产使用
> - **设计**：简洁实用，不追求过度设计
> - **技术**：展示现代前端工程化能力
> 
> **未来方向：**
> - 如果用于生产，可以考虑：
>   - 参考 Ant Design 的组件设计
>   - 学习 MUI 的主题系统
>   - 借鉴 Chakra UI 的 API 设计
>   - 保持自己的特色和定位"

---

## 📝 总结

这份 Q&A 文档涵盖了 PDF 中提到的所有"可能提问"，并基于项目的实际实现提供了详细的回答。每个回答都包含：

1. **核心要点**：简洁明了的回答
2. **技术细节**：代码示例和配置说明
3. **实际案例**：基于项目代码的具体示例

**使用建议：**
- 先通读一遍，理解整体架构
- 针对每个问题，结合项目代码深入理解
- 准备时可以重点记忆核心要点
- 面试时根据实际情况灵活调整回答

**项目亮点总结：**
1. ✅ Monorepo 架构（pnpm workspace）
2. ✅ 完整的工程化工具链（ESLint、Prettier、Husky、lint-staged）
3. ✅ 测试体系（Jest + React Testing Library）
4. ✅ 文档系统（自定义 Preview + highlight.js）
5. ✅ TypeScript 类型安全
6. ✅ SCSS Modules 样式隔离
7. ✅ 组件设计规范（forwardRef、复合组件等）

祝你面试顺利！🎉

