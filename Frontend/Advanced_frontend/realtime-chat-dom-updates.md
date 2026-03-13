# ⚛️ Scenario — Real-time Chat: Efficient DOM Updates

## The Problem
Multiple messages arriving simultaneously → each message triggers separate re-render → main thread gets blocked → UI freezes.

---

## Solution Approach

```jsx
import { useState, useTransition, useCallback, memo } from "react";

export default function ChatApp() {
  const [messages, setMessages] = useState([]);
  const [isPending, startTransition] = useTransition();

  const addMessage = useCallback((newMessage) => {
    // useTransition marks this as non-urgent
    // React can pause this if something more important comes in
    startTransition(() => {
      setMessages(prev => [...prev, newMessage]);
    });
  }, []);

  return (
    <div>
      {isPending && <p>Updating...</p>}
      <MessageList messages={messages} />
    </div>
  );
}

// React.memo — prevents re-render if messages didn't change
const MessageList = memo(({ messages }) => {
  return (
    <div>
      {messages.map(msg => (
        <MessageItem key={msg.id} message={msg} />
      ))}
    </div>
  );
});

// Each message isolated — one update doesn't re-render others
const MessageItem = memo(({ message }) => {
  return <div>{message.text}</div>;
});
```

---

## Key Concepts Used

| Concept | Why Used |
|---------|---------|
| `useTransition` | Marks message updates as non-urgent — keeps UI responsive |
| `React.memo` | Prevents re-render of list if messages didn't change |
| `useCallback` | Stable function reference — no re-creation on every render |
| `key` prop | React efficiently identifies which message changed |

---

## For Large Message Lists — Add Virtualization
If chat has 10,000+ messages, only render what's visible in viewport:

```jsx
import { FixedSizeList } from 'react-window';

<FixedSizeList
  height={500}
  itemCount={messages.length}
  itemSize={50}
>
  {({ index, style }) => (
    <div style={style}>
      {messages[index].text}
    </div>
  )}
</FixedSizeList>
```

---

## 🔑 Key Points To Remember
- `useTransition` = marks updates as non-urgent, React can pause them
- `React.memo` = component level memoization — skips re-render if props same
- Virtualization (`react-window`) = only render visible items in DOM
- WebSocket batching — batch incoming messages before calling setState
- Always use `key={msg.id}` — never use index as key in dynamic lists
