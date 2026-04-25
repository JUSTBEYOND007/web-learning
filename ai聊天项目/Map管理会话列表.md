这里的设计就是：

- 用一个 Map<string, MessageProps[]> 来存所有消息：

- key：会话 id（chatId）

- value：这条会话下的消息数组

- 当前选中的会话 id 存在 useConversationStore.selectedId 里：

- addMessage 时，通过 selectedId 找到对应的数组，把新消息 push 进去；

- ChatBubble 渲染时，通过 messages.get(selectedId) 取出当前会话的所有消息。