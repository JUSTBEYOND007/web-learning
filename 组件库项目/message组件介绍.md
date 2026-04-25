# Message 组件技术实现说明


> 用于面试时说明 Message 组件的实现思路和技术亮点


---

  

## 📋 组件概述

  

Message 是一个**通知提示组件**，支持函数式调用、动态创建/销毁实例，并自动计算垂直偏移量解决消息堆叠问题。

  

**核心特性：**

- ✅ 函数式调用 API（`Message.success()`、`Message.error()` 等）

- ✅ 动态创建/销毁消息实例

- ✅ 自动计算垂直偏移量，支持多条消息堆叠

- ✅ 自动关闭和手动关闭

- ✅ 平滑的动画效果

- ✅ 使用 React Portal 渲染到 body

  

---

  

## 🏗️ 架构设计

  

### 1. 组件结构

  

```

Message/

├── Message.tsx              # 主组件文件（包含 API 和组件）

├── Message.module.scss      # 样式文件

├── Message.test.tsx         # 测试文件

├── Message.stories.tsx      # Storybook 文档

└── index.ts                 # 导出文件

```

  

### 2. 核心模块

  

**三个核心部分：**

  

1. **API 层**：`Message` 对象，提供函数式调用接口

2. **状态管理层**：全局消息列表和监听机制

3. **渲染层**：`MessageContainer` 组件，使用 Portal 渲染

  

---

  

## 💡 技术实现细节

  

### 1. 函数式调用 API

  

**实现方式：**

  

```typescript

export const Message: MessageInstance = {

  success: (content: React.ReactNode, duration?: number) => {

    return addMessage({ content, type: 'success', duration });

  },

  error: (content: React.ReactNode, duration?: number) => {

    return addMessage({ content, type: 'error', duration });

  },

  // ...

};

```

  

**设计思路：**

- 提供简洁的 API，无需手动创建组件实例

- 返回消息的 `key`，支持后续操作（关闭、更新等）

- 支持链式调用和批量创建

  

**使用示例：**

```typescript

// 简单调用

Message.success('操作成功');

  

// 获取 key 用于后续操作

const key = Message.error('操作失败');

Message.close(key);

```

  

---

  

### 2. 动态创建/销毁实例

  

**实现原理：**

  

```typescript

// 全局消息列表

let messageList: MessageItem[] = [];

let listeners: Set<(messages: MessageItem[]) => void> = new Set();

  

// 添加消息

const addMessage = (options: MessageOptions): string => {

  const key = options.key?.toString() || `message-${Date.now()}-${Math.random()}`;

  const top = calculateTop();

  const message: MessageItem = {

    ...options,

    key,

    top,

    duration: options.duration ?? 3000,

  };

  

  messageList.push(message);

  notifyListeners(); // 通知所有订阅者

  return key;

};

  

// 移除消息

const removeMessage = (key: string | number) => {

  const keyStr = key.toString();

  const index = messageList.findIndex((msg) => msg.key === keyStr);

  if (index !== -1) {

    messageList.splice(index, 1);

    recalculatePositions(); // 重新计算位置

    notifyListeners();

  }

};

```

  

**关键点：**

- 使用全局状态管理消息列表（避免 Context 的复杂性）

- 采用发布-订阅模式，通知组件更新

- 自动生成唯一 key，支持自定义 key

  

---

  

### 3. 自动计算垂直偏移量

  

**核心算法：**

  

```typescript

// 消息配置常量

const MESSAGE_HEIGHT = 48; // 单条消息高度（包含间距）

const MESSAGE_GAP = 16;     // 消息之间的间距

const START_TOP = 24;       // 起始位置

  

// 计算新消息的 top 位置

const calculateTop = (): number => {

  if (messageList.length === 0) {

    return START_TOP;

  }

  // 获取最后一条消息的位置

  const lastMessage = messageList[messageList.length - 1];

  return lastMessage.top + MESSAGE_HEIGHT + MESSAGE_GAP;

};

  

// 重新计算所有消息的位置（当消息被移除时）

const recalculatePositions = () => {

  messageList.forEach((message, index) => {

    message.top = START_TOP + index * (MESSAGE_HEIGHT + MESSAGE_GAP);

  });

  notifyListeners();

};

```

  

**工作流程：**

  

1. **添加消息时**：

   - 如果列表为空，位置 = `START_TOP`

   - 否则，位置 = 最后一条消息的 top + 消息高度 + 间距

  

2. **移除消息时**：

   - 从列表中删除消息

   - 重新计算所有剩余消息的位置

   - 通知组件更新，触发平滑动画

  

**优势：**

- ✅ 自动处理堆叠，无需手动计算

- ✅ 移除消息后自动调整位置

- ✅ 支持任意数量的消息

  

---

  

### 4. React Portal 渲染

  

**实现方式：**

  

```typescript

// 获取或创建容器元素

const getContainerElement = (): HTMLDivElement => {

  if (!containerElement) {

    containerElement = document.createElement('div');

    containerElement.className = 'aurora-message-container';

    document.body.appendChild(containerElement);

  }

  return containerElement;

};

  

// MessageContainer 组件

export const MessageContainer: React.FC = () => {

  const [messages, setMessages] = useState<MessageItem[]>([]);

  

  useEffect(() => {

    const listener = (msgs: MessageItem[]) => {

      setMessages(msgs);

    };

    listeners.add(listener);

    listener(messageList); // 立即获取当前消息

  

    return () => {

      listeners.delete(listener);

    };

  }, []);

  

  const containerEl = typeof document !== 'undefined' ? getContainerElement() : null;

  

  if (!containerEl || messages.length === 0) {

    return null;

  }

  

  return createPortal(

    <div className={styles.messageWrapper}>

      {messages.map((msg) => (

        <MessageItemComponent key={msg.key} message={msg} onClose={removeMessage} />

      ))}

    </div>,

    containerEl

  );

};

```

  

**为什么使用 Portal：**

- ✅ 避免 z-index 层级问题

- ✅ 不受父组件样式影响

- ✅ 渲染到 body，确保在最顶层显示

  

---

  

### 5. 动画效果

  

**CSS 动画实现：**

  

```scss

.messageItem {

  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  opacity: 0;

  transform: translateX(-50%) translateY(-20px);

  &--visible {

    opacity: 1;

    transform: translateX(-50%) translateY(0);

  }

  &--closing {

    opacity: 0;

    transform: translateX(-50%) translateY(-20px);

  }

}

```

  

**JavaScript 控制：**

  

```typescript

const [visible, setVisible] = useState(false);

const [closing, setClosing] = useState(false);

  

useEffect(() => {

  // 入场动画

  const timer = setTimeout(() => setVisible(true), 10);

  

  // 自动关闭

  if (message.duration > 0) {

    const closeTimer = setTimeout(() => {

      handleClose();

    }, message.duration);

  

    return () => {

      clearTimeout(timer);

      clearTimeout(closeTimer);

    };

  }

  

  return () => clearTimeout(timer);

}, [message.duration]);

  

const handleClose = () => {

  setClosing(true);

  setTimeout(() => {

    onClose(message.key);

    message.onClose?.();

  }, 300); // 等待动画完成

};

```

  

**动画流程：**

1. 初始状态：`opacity: 0`，向上偏移

2. 入场：添加 `visible` 类，`opacity: 1`，回到原位置

3. 退场：添加 `closing` 类，`opacity: 0`，向上偏移

4. 动画完成后从 DOM 移除

  

---

  

## 🎯 面试回答要点

  

### 问题：如何实现 Message 组件的函数式调用方式？

  

**回答：**

  

> "Message 组件采用**全局状态管理 + 发布订阅模式**实现函数式调用：

>

> **1. API 层设计：**

> - 提供 `Message.success()`、`Message.error()` 等静态方法

> - 每个方法内部调用 `addMessage()` 添加消息到全局列表

> - 返回消息的 `key`，支持后续操作

>

> **2. 状态管理：**

> - 使用模块级变量 `messageList` 存储所有消息

> - 使用 `Set` 存储订阅者（监听器）

> - 通过 `notifyListeners()` 通知所有订阅者更新

>

> **3. 组件渲染：**

> - `MessageContainer` 组件订阅消息变化

> - 使用 `React Portal` 渲染到 `document.body`

> - 根据消息列表动态渲染 `MessageItem` 组件

>

> **优势：**

> - 无需手动创建组件实例

> - 可以在任何地方调用，不受组件层级限制

> - 支持批量创建和动态管理"

  

---

  

### 问题：如何实现自动计算垂直偏移量解决消息堆叠问题？

  

**回答：**

  

> "我实现了一个**位置计算算法**，自动处理消息的垂直堆叠：

>

> **1. 添加消息时的计算：**

> ```typescript

> const calculateTop = (): number => {

>   if (messageList.length === 0) {

>     return START_TOP; // 第一条消息从顶部开始

>   }

>   const lastMessage = messageList[messageList.length - 1];

>   return lastMessage.top + MESSAGE_HEIGHT + MESSAGE_GAP;

> };

> ```

>

> **2. 移除消息时的重新计算：**

> ```typescript

> const recalculatePositions = () => {

>   messageList.forEach((message, index) => {

>     message.top = START_TOP + index * (MESSAGE_HEIGHT + MESSAGE_GAP);

>   });

>   notifyListeners(); // 触发重新渲染

> };

> ```

>

> **3. 工作流程：**

> - 添加消息：计算新位置 = 最后一条消息位置 + 高度 + 间距

> - 移除消息：重新遍历列表，按索引计算新位置

> - CSS 过渡动画：位置变化时自动触发平滑动画

>

> **技术细节：**

> - 使用常量定义消息高度和间距，便于调整

> - 通过 `style.top` 动态设置位置

> - CSS `transition` 实现平滑的位置变化动画

>

> **优势：**

> - 完全自动化，无需手动计算

> - 支持任意数量的消息堆叠

> - 移除消息后自动调整，保持视觉连贯性"

  

---

  

### 问题：如何实现动态创建/销毁实例？

  

**回答：**

  

> "Message 组件采用**单例模式 + 全局状态管理**实现动态实例管理：

>

> **1. 单例容器：**

> - 使用模块级变量存储消息列表和监听器

> - 确保全局只有一个消息列表实例

> - 通过函数暴露操作接口

>

> **2. 创建实例：**

> ```typescript

> const addMessage = (options: MessageOptions): string => {

>   const key = generateUniqueKey();

>   const top = calculateTop();

>   const message = { ...options, key, top };

>   messageList.push(message);

>   notifyListeners(); // 通知组件更新

>   return key;

> };

> ```

>

> **3. 销毁实例：**

> ```typescript

> const removeMessage = (key: string | number) => {

>   const index = messageList.findIndex(msg => msg.key === key);

>   if (index !== -1) {

>     messageList.splice(index, 1);

>     recalculatePositions();

>     notifyListeners();

>   }

> };

> ```

>

> **4. 自动销毁：**

> - 通过 `setTimeout` 实现自动关闭

> - 支持自定义持续时间（`duration`）

> - 关闭时触发动画，动画完成后从 DOM 移除

>

> **5. 批量销毁：**

> ```typescript

> Message.destroy(); // 清除所有消息

> ```

>

> **优势：**

> - 内存管理：消息关闭后自动清理

> - 性能优化：只渲染存在的消息

> - 灵活控制：支持手动和自动销毁"

  

---

  

## 📊 技术亮点总结

  

1. **函数式 API 设计**：简洁易用，符合 React 生态习惯

2. **自动位置计算**：智能堆叠，无需手动管理

3. **Portal 渲染**：解决层级问题，确保正确显示

4. **发布订阅模式**：解耦状态管理和 UI 渲染

5. **平滑动画**：CSS 过渡 + JavaScript 控制，体验流畅

6. **类型安全**：完整的 TypeScript 类型定义

7. **测试覆盖**：单元测试覆盖核心功能

8. **文档完善**：Storybook 展示各种使用场景

  

---

  

## 🚀 使用示例

  

```typescript

// 1. 在应用根组件中渲染 MessageContainer

import { MessageContainer } from '@aurora-ui/components';

  

function App() {

  return (

    <>

      <MessageContainer />

      {/* 其他组件 */}

    </>

  );

}

  

// 2. 在任何地方调用

import { Message } from '@aurora-ui/components';

  

// 基础用法

Message.success('操作成功');

Message.error('操作失败');

  

// 自定义持续时间

Message.warning('警告信息', 5000);

  

// 手动关闭

const key = Message.info('提示信息', 0);

setTimeout(() => Message.close(key), 3000);

  

// 回调函数

Message.open({

  content: '自定义消息',

  type: 'success',

  onClose: () => console.log('消息已关闭'),

});

```

  

---

  

## 📝 面试加分点

  

1. **架构设计能力**：展示了全局状态管理、发布订阅模式的应用

2. **算法思维**：位置计算算法体现了对数据结构和算法的理解

3. **React 深度应用**：Portal、Hooks、状态管理的综合运用

4. **用户体验考虑**：动画效果、自动关闭、位置调整等细节

5. **工程化思维**：类型定义、测试覆盖、文档完善

  

这个组件展示了从**需求分析 → 架构设计 → 实现细节 → 用户体验**的完整开发流程，是一个很好的面试项目亮点！