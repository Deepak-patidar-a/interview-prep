# ⚛️ Optimistic vs Pessimistic Updates

## The Core Difference

```
Pessimistic Update:
  User clicks Like
  → Wait for API response (500ms...)
  → If success → update UI
  → User waits seeing nothing  ← bad UX 😤

Optimistic Update:
  User clicks Like
  → Update UI immediately (assume success)
  → API call happens in background
  → If API fails → rollback UI  ← great UX 😊
```

---

## Pessimistic Update — Code

```javascript
function LikeButton({ postId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  const [loading, setLoading] = useState(false);

  const handleLike = async () => {
    setLoading(true);
    try {
      await api.likePost(postId);  // wait for API first
      setLikes(prev => prev + 1);  // then update UI
    } catch (error) {
      alert("Failed to like");
    } finally {
      setLoading(false);
    }
  };

  return (
    <button onClick={handleLike} disabled={loading}>
      {loading ? "..." : `❤️ ${likes}`}
    </button>
  );
}
```

---

## Optimistic Update — Code

```javascript
function LikeButton({ postId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  const [liked, setLiked] = useState(false);

  const handleLike = async () => {
    // Step 1 — Update UI immediately (optimistic)
    setLikes(prev => prev + 1);
    setLiked(true);

    try {
      await api.likePost(postId);  // API in background
    } catch (error) {
      // Step 2 — API failed → rollback UI
      setLikes(prev => prev - 1);
      setLiked(false);
      alert("Failed to like — please try again");
    }
  };

  return (
    <button onClick={handleLike}>
      {liked ? "❤️" : "🤍"} {likes}
    </button>
  );
}
```

---

## When To Use Which

| Scenario | Use |
|----------|-----|
| Like / Follow / Bookmark | ✅ Optimistic |
| Add to cart | ✅ Optimistic |
| Form submission | ❌ Pessimistic |
| Payment / checkout | ❌ Pessimistic |
| Delete account | ❌ Pessimistic |
| Sending a message | ✅ Optimistic (show as "sending") |

---

## 🔑 Key Points To Remember
- Optimistic = update UI first, rollback on failure
- Pessimistic = wait for API, then update UI
- Use optimistic for low-risk, reversible actions (like, bookmark, add to cart)
- Use pessimistic for high-risk, irreversible actions (payment, delete, form submit)
- Always implement rollback logic in optimistic updates — don't skip it
- React Query's `onMutate` + `onError` makes optimistic updates much cleaner
