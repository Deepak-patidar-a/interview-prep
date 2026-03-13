# ⚛️ Scenario — E-commerce Cart with React Query Optimistic Updates

## The Problem
Multiple users adding items to cart simultaneously → stale UI → cart shows wrong count.

---

## Solution — React Query with Optimistic Updates

```javascript
import { useMutation, useQueryClient } from '@tanstack/react-query';

function ProductCard({ product }) {
  const queryClient = useQueryClient();

  const addToCartMutation = useMutation({
    mutationFn: (productId) => api.addToCart(productId),

    // Step 1 — runs BEFORE API call (optimistic update)
    onMutate: async (productId) => {
      // Cancel any in-flight refetches — avoid overwriting optimistic update
      await queryClient.cancelQueries({ queryKey: ['cart'] });

      // Snapshot current cart (for rollback if API fails)
      const previousCart = queryClient.getQueryData(['cart']);

      // Optimistically update cart in cache immediately
      queryClient.setQueryData(['cart'], (old) => ({
        ...old,
        items: [...old.items, { id: productId, quantity: 1 }]
      }));

      return { previousCart };  // return snapshot for onError
    },

    // Step 2 — runs if API fails (rollback)
    onError: (error, productId, context) => {
      queryClient.setQueryData(['cart'], context.previousCart);  // restore snapshot
    },

    // Step 3 — always runs after success or error (sync with server)
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });  // refetch real data
    }
  });

  return (
    <button
      onClick={() => addToCartMutation.mutate(product.id)}
      disabled={addToCartMutation.isPending}
    >
      {addToCartMutation.isPending ? "Adding..." : "Add to Cart"}
    </button>
  );
}
```

---

## The Complete Flow

```
User clicks "Add to Cart"
  ↓
onMutate fires
  → cancelQueries (stop any ongoing fetches)
  → snapshot current cart (for rollback)
  → update cache immediately (UI updates instantly ✅)
  ↓
API call happens in background
  ↓
  ├── Success → onSettled → invalidateQueries → refetch real cart from server
  └── Failure → onError  → restore snapshot (rollback UI) → onSettled
```

---

## Why cancelQueries is Important

```javascript
// Without cancelQueries:
// 1. Optimistic update sets cart = [A, B, C]
// 2. Old in-flight query finishes and returns [A, B]
// 3. React Query updates cache with [A, B] — overwrites optimistic update 💥

// With cancelQueries:
// 1. Cancel any in-flight cart queries
// 2. Optimistic update sets cart = [A, B, C] safely ✅
// 3. onSettled triggers fresh refetch after mutation
```

---

## React Query vs Manual Optimistic Update

| | Manual useState | React Query |
|--|----------------|-------------|
| Cache management | Manual | Automatic |
| Rollback | Manual | Via context |
| Refetch after mutation | Manual | invalidateQueries |
| Loading states | Manual | isPending built-in |
| Code complexity | High | Low |

---

## 🔑 Key Points To Remember
- `onMutate` = runs before API call — perfect for optimistic updates
- Always snapshot previous data in `onMutate` — needed for rollback
- `cancelQueries` = prevents race conditions with in-flight requests
- `onError` = rollback using the snapshot from `onMutate` context
- `onSettled` = always runs — use to sync with real server data
- `invalidateQueries` = marks cache as stale → triggers refetch
